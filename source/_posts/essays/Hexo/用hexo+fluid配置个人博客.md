---
title: 用hexo+fluid配置个人博客
date: 2025-02-22 20:47:31
tags:
categories: [essays,Hexo]

---

# 环境准备

 在开始之前，你需要先参考[文档 | Hexo](https://hexo.io/zh-cn/docs/)进行安装、建站。

## 然后设置你的主题为fluid：

- 进入博客目录执行命令：`npm install --save hexo-theme-fluid`

- 修改 Hexo 博客目录下`_config.yml`中的对应参数：

```yaml
       theme: fluid  # 指定主题为fluid
```

       

#  开始新文档创建

## 创建`关于页`

- 在博客目录下执行命令：`hexo new page about`

- 创建成功后修改 `/source/about/index.md`，在其中添加 `layout` 属性。

```yaml
    ---
    title: 标题
    layout: about
    ---
    
    这里写关于页的正文，支持 Markdown, HTML
```

{% note info %}
根据官方文档，`layout: about` 必须存在，并且不能修改成其他值，否则不会显示头像等样式。

一般`关于页`可以介绍你的个人资料

{% endnote %}


##  创建新的文章并为其归类。（这里的做法为个人偏好，仅供参考）

我现在需要写一篇名为`用hexo+fluid配置个人博客`的文章，我想给其分类到`随笔`下的`Hexo`小类中，这里我就用到了hexo的二级导航功能。具体做法如下：

### 生成“分类”页并添加tpye属性

- 进入博客所在文件夹。执行命令：`hexo new page categories`

- 修改博客目录下的`source/categories/index.md`为：

```yaml
  ---
  title: categories
  date: 2025-02-22 19:55:44
  layout: categories 	#添加这一行代码到原md文件中
  ---
```

### 创建`随笔\Hexo`类别：

- 在 `source/categories` 文件夹下创建一个名为 `essay` 的文件夹,再在`essay`文件夹内新建`Hexo`文件夹

- 在`source/categories/essay/和` `source/categories/essay/Hexo` 文件夹下均创建一个名为 `index.md` 的文件，内容如下

```
---
date: 2025-02-22 19:57:44
type: categories
---
```

- 修改主题目录下的`_config.yaml`文件中的menu属性，它将影响到网页中的菜单栏，修改为：

```yaml
    menu:
      - { key: "home", link: "/", icon: "iconfont icon-home-fill" }
      - { key: "随笔", link: "/categories/essay/", icon: "iconfont book icon-book",
        submenu: [{key: 'Hexo用户笔记', link: '/categories/essay/Hexo/'},],}
      # 设置你的多级目录以及设置key的值作为菜单栏中展示的分类名
      - { key: "about", link: "/about/", icon: "iconfont icon-user-fill",}
      #- { key: "archive", link: "/archives/", icon: "iconfont icon-archive-fill" }
      #- { key: "categories", link: "/categories/", icon: "iconfont icon-category-fill", }
      #- { key: "tag", link: "/tags/", icon: "iconfont icon-tags-fill" }
      #- { key: "links", link: "/links/", icon: "iconfont icon-link-fill" }
```


{% note info %}
注意，我这里注释掉了不必要的菜单栏
{% endnote %}


### 创建新的文章并归类:

- 在`source\_posts`目录下新建`essays\Hexo`目录结构

- 执行`hexo new 用hexo+fluid配置个人博客 -p essays\Hexo\用hexo+fluid配置个人博客`

- 修改`source\_posts\essays\Hexo\用hexo+fluid配置个人博客.md`中的内容为：

```yaml
---
title: 用hexo+fluid配置个人博客
date: 2025-02-23 14:36:55
tags:
categories: [essay,Hexo] #加上这一栏即可
---
```

  

  执行完上述一系列操作之后，你将得到这样一个界面：

  ![](/images/image-20250223144908444.jpg)



  然后你就可以编辑`用hexo+fluid配置个人博客.md`中的内容作为你的第一篇博客了。


  

  

  

  

  

  

  

  

  

  

  

  

  

  







