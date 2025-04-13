---
title: 基于ragas和evalscope实现rag快速评测
date: 2025-04-7 15:47:31
tags:
categories: [ReadingNotes,rag]
---



---

### Ragas 评估工具概述

Ragas 是一个用于评估 RAG（Retrieval-Augmented Generation，检索增强生成）应用程序的工具库。它通过提供一系列工具来帮助用户轻松且自信地评估其 RAG 应用程序的性能。

Ragas 采用了一种新颖的评测数据生成方法。理想的评测数据集应该涵盖实际应用中遇到的各种类型的问题，包括不同难度等级的问题。然而，默认情况下，大语言模型（LLMs）通常不擅长创建多样化的样本，因为它们往往会遵循常见的路径。受到 Evol-Instruct 等工作的启发，Ragas 通过采用进化生成范式来实现这一目标。在这个过程中，具有不同特征的问题（如推理、条件、多个上下文等）会根据提供的文档集被系统地构建。这种方法确保了对管道中各个组件性能的全面覆盖，从而使评测过程更加稳健。

Ragas 的官网 [https://docs.ragas.io/](https://docs.ragas.io/) 介绍了如何通过大模型 API 接口初始化评测端模型，以实现评测指标的计算。然而，这种方式可能会受到网络连接等因素的影响，从而限制本地应用的效果。

### 在本地部署 Ragas 服务

为了克服网络连接问题并提升本地应用的性能，可以通过 [EvalScope](https://evalscope.readthedocs.io/zh-cn/latest/user_guides/backend/rageval_backend/ragas.html) 项目结合 [vLLM](https://vllm.hyper.ai/docs/getting-started/quickstart/) 来实现本地部署。

#### 使用 vLLM 构建本地 API 服务器

vLLM 是一个高性能的推理引擎，可以在本地为大语言模型构建 API 服务器。通过 vLLM，用户可以快速部署本地 Ragas 服务，从而避免网络连接问题对评估过程的影响。

以下是使用 vLLM 构建本地 API 服务器的步骤：

1. **安装 vLLM**：
   ```bash
   pip install vllm
   ```

2. **启动 vLLM 服务器**：
   ```bash
   vllm serve facebook/opt-125m
   ```
   默认情况下，服务器会在 `http://localhost:8000` 启动。可以通过 `--host` 和 `--port` 参数指定其他地址。

3. **使用 OpenAI API 兼容模式**：
   vLLM 服务器支持 OpenAI API 协议，可以直接替代 OpenAI API。可以使用 `curl` 或 Python 客户端进行查询。例如：
   ```bash
   curl http://localhost:8000/v1/completions \
       -H "Content-Type: application/json" \
       -d '{
           "model": "facebook/opt-125m",
           "prompt": "San Francisco is a",
           "max_tokens": 7,
           "temperature": 0
       }'
   ```

4. **在 EvalScope 中使用本地 vLLM 服务**：
   在 EvalScope 的配置中，将评测模型的 API 基础地址设置为本地 vLLM 服务器的地址。例如：
   ```python
   eval_task_cfg = {
       "eval_backend": "RAGEval",
       "eval_config": {
           "tool": "RAGAS",
           "eval": {
               "testset_file": "outputs/testset_with_answer.json",
               "critic_llm": {
                   "model_name": "facebook/opt-125m",
                   "api_base": "http://localhost:8000/v1",
                   "api_key": "EMPTY",
               },
               "embeddings": {
                   "model_name_or_path": "AI-ModelScope/m3e-base",
               },
               "metrics": [
                   "Faithfulness",
                   "AnswerRelevancy",
                   "ContextPrecision",
                   "AnswerCorrectness",
               ],
               "language": "chinese"
           },
       },
   }
   ```

