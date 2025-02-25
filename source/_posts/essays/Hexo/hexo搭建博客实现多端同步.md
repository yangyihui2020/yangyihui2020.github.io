---
title: hexo搭建博客实现多端同步
date: 2025-02-25 15:47:31
tags:
categories: [essays,Hexo]

---


# 基于hexo+github实现博客撰写的多端同步功能

hexo 本身自带的同步功能`hexo d`上传的只是网页部署文件，未上传网页源文件，所以如果我们要在另一台设备上对博客进行修改的话就会遇到困难。我们可以在在github的blog仓库新建一个分支，存储源文件，然后在不同设备间同步源文件。这样就可以实现博客在不同设备上的同步了。


## github对应的仓库新建分支

首先你需要在你博客对应的github仓库(比如我的就是`https://github.com/yangyihui2020/yangyihui2020.github.io`新建分支`hexo`。

## 在已有博客文件的旧设备上执行的操作 ：

首先你需要在你博客对应的github仓库(比如我的就是`https://github.com/yangyihui2020/yangyihui2020.github.io`新建分支`hexo`。

然后在本任意目录下执行 `git clone`。并切换到对应分支：
```
git clone git@github.com:yangyihui2020/yangyihui2020.github.io.git
cd yangyihui2020.github.io
git switch hexo
```

然后删除掉仓库目录（比如我的就是`yangyihui2020.github.io`)下除去.git之外的所有文件，并将原博客目录下的文件复制到该目录下。


然后上传到hexo分支

```
git add .
git commit -m "新建hexo分支"
git push 
```




## 在新设备上执行的操作 ：

在新设备，你需要配置好hexo所需要的环境。可以参考[文档 | Hexo](https://hexo.io/zh-cn/docs/) 。


然后在任意目录下新建一个目录`blogs`(这个名字随意),进入该目录执行`hexo init`。
然后清空该目录的文件，并执行`git clone`并切换到对应的分支:
```
git clone git@github.com:yangyihui2020/yangyihui2020.github.io.git
git switch hexo
```



然后你就可以在不同设备上实现hexo博客的源文件共享了。注意，每次写博客之前最好执行`git pull`来保持文件的同步，上传网页时同时执行一下`git push`

