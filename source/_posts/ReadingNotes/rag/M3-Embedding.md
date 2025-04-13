---
title: M3-Embedding Multi-Linguality, Multi-Functionality, Multi-Granularity Text Embeddings Through Self-Knowledge Distillation
date: 2025-03-13 15:47:31
tags:
categories: [ReadingNotes,rag]
---

## 研究背景

这篇文章要解决的问题是如何设计一种多语言、多功能、多粒度的文本嵌入模型，以支持超过100种工作语言的语义检索，并能同时完成<mark>密集检索、多向量检索和稀疏检索</mark>等多种检索功能。

## 研究方法
### 混合检索
M3-Embedding统一了文本嵌入模型的常见检索功能，包括密集检索、词汇（稀疏）检索和多向量检索。这些功能不仅可以单独工作，还可以协同工作以获得更强的检索质量。

#### 密集检索（Dense retrieval.）
- 输入query $q$ 将转换为基于文本编码器的隐藏状态 $\mathbf{H_q}$。使用特殊标记 “[CLS]” 的规范化隐藏状态来表示query：$e_q = \text{norm}(\mathbf{H_q}[0])$
- passage的嵌入 $p$ 为 $e_p = \text{norm}(\mathbf{H_p}[0])$。

- 因此，query和passage之间的相关性分数由两个嵌入和 $e_q e_p$：$s_{\text{dense}} \leftarrow \langle e_p, e_q \rangle$ 之间的内积来衡量。



#### 词汇检索（Lexical Retrieval. ）
- 对于query中的每个token $t$，其权重的计算公式为

$$
w_{q_t} \leftarrow\text{Relu}(\mathbf{W}_{\text{lex}}^T \mathbf{H_q}[i])
$$


- 其中 $\mathbf{W}_{\text{lex}} \in \mathbb{R}^{d \times 1}$ 是将隐藏状态映射到浮点数的矩阵。
- 如果某个token $t$ 在query中多次出现，只保留其最大权重。并使用相同的方法来计算passage中每个token的权重。
- 因此query和passage之间的相关性分数是通过query和passage中共存token（表示为 $q \cap p$）的联合重要性来计算的：
$$
s_{\text{lex}} \leftarrow \sum_{t \in q \cap p}(w_{q_t} * w_{p_t})
$$

#### 多向量检索 (Multi-Vector Retrieval. )
- 作为密集检索的扩展，多向量检索利用整个输出嵌入来表示query和passage：

$$
E_q = \text{norm}(\mathbf{W}_{\text{mul}}^T \mathbf{H_q}) 
$$

$$
E_p = \text{norm}(\mathbf{W}_{\text{mul}}^T \mathbf{H_p})
$$    

- 其中 $\mathbf{W}_{\text{mul}} \in \mathbb{R}^{d \times d}$ 是可学习的投影矩阵。

- 使用后期交互来计算细粒度的相关性分数：
$$
s_{\text{mul}} \leftarrow \frac{1}{N} \sum_{i=1}^N \max_{j=1}^M E_q[i] \cdot E_p^T[j]
$$
- $N$ 和 $M$ 是query和passage的长度。



每种方法都可以单独检索候选结果（多向量方法由于成本高，可以免于此步骤）。然后，根据集成的相关性分数对最终检索结果进行重新排名：

$$
S_{\text{rank}} \leftarrow w_1 \cdot S_{\text{dense}} + w_2 \cdot S_{\text{lex}} + w_3 \cdot S_{\text{mul}}
$$


### 自我认识蒸馏(Self-Knowledge Distillation)
#### 损失函数

嵌入模型经过训练以区分阳性样本和阴性样本。对于每种检索方法，与负样本相比，应为查询的正样本分配更高的分数。因此，进行训练过程以最小化 InfoNCE 损失，其一般形式由以下损失函数表示：

$$
\mathcal{L}_{s(\cdot)} = -\log \frac{\exp(s(q, p^*) / \tau)} {\sum_{p \in \{p^*, p'\}} \exp(s(q, p) / \tau)}
$$

这里，$p^*$ 和 $p'$ 代表查询 $q$ 的正样本和负样本

$s(\cdot)$ 是 $\{s_{\text{dense}}(\cdot)$,$s_{\text{lex}}(\cdot)$,$s_{\text{mul}}(\cdot)$ 中的任何函数。

- 不同检索方法的训练目标可能与各自的方法相互冲突。因此，原生多目标训练可能不利于嵌入的质量。为了促进多个检索函数的优化，作者建议在自我知识蒸馏之上统一训练过程。在最简单的形式中，积分可以是不同预测分数的加权和：

$$
S_{\text{inter}} \leftarrow w_1 \cdot S_{\text{dense}} + w_2 \cdot S_{\text{lex}} + w_3 \cdot S_{\text{mul}}.
$$

- 使用积分分数 $S_{inter}$作为老师，于是其中每种检索方法的损失函数被修改为:

$$
\mathcal{L}^{'}_* \leftarrow -p(s_{\text{inter}}) \ast \log p(s_*).
$$
- 这里，$p(\cdot)$ 是 softmax 激活；$s_*$ 是 $s_{\text{dense}}$、$s_{\text{lex}}$ 和 $s_{\text{mul}}$ 中的任何成员。我们进一步整合并归一化修改后的损失函数：
$$
\mathcal{L}' \leftarrow \left( \lambda_1 \cdot \mathcal{L}'_{\text{dense}} + \lambda_2 \cdot \mathcal{L}'_{\text{lex}} + \lambda_3 \cdot \mathcal{L}'_{\text{mul}} \right) / 3.
$$

最后，用 $\mathcal{L}$ 和的 $\mathcal{L}'$ 线性组合推导出自我知识蒸馏的最终损失函数：
$$
\mathcal{L}_{\text{final}} \leftarrow \left( \mathcal{L} + \mathcal{L}' \right) / 2
$$


#### 训练过程

训练过程包括一个**多阶段工作流**（如图所示）。首先，文本编码器（XLM-RoBERTa模型）使用大量无监督数据进行预训练，其中仅以基本形式的对比学习训练密集检索。

在第二阶段应用自知识蒸馏，其中嵌入模型被微调以建立三个检索功能。$\mathbf{W}_{\text{lex}}$的随机初始化导致$s_{\text{lex}}$准确性差和训练开始时的$\mathcal{L}_{\text{lex}}$高。

![](/images/m3-embedding训练过程.png)