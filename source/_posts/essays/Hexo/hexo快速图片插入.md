---
title: hexo+vscode实现快速图片插入
date: 2025-04-11 16:46:39
tags:
categories: [essays,Hexo]
---


我们通常使用hexo框架插入图片时，图片通常会被保存到一个绝对路径的文件夹下`\source\images`,
这样比较麻烦，这里介绍一种可以直接在插入保存到粘贴板中的方法。



#### 首先使用`hexo new`命令来创建文章，比如我使用
```
hexo new post hexo+vscode实现快速图片插入 -p essays/Hexo/hexo快速图片插入
```
该命令会新建一个md文件和与md文件同名的文件夹，我们就可以使用该文件夹来储存图片。


#### 然后在vscode的扩展中新增paste image插件，同时在vscode 的settings.json新加配置:
```
"pasteImage.path": "${currentFileDir}/${currentFileNameWithoutExt}" 

```
#### 然后使用快捷键ctrl-alt-v就可以快速保存图片并且插入图片url。


![](hexo快速图片插入/2025-04-11-17-18-08.png)


{% note info %}
如果图片无法正常加载，可使用hexo clean清除缓存
{%endnote%}