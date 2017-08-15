---
layout: post
title:  Jekyll个人博客搭建
date:   2017-8-15 20:28:50
categories: Other
tags: Blog Github
author: renshuo
mathjax: true
---

* content
{:toc}

个人博客搭建过程记录

<!--more-->

之前一直犹豫要不要搭建个人博客，主要是自己写的东西比较少，而且感觉写东西的时间可能不那么容易挤出来。后来终于下定决心做一个。

## 使用jekyll的原因

目前个人博客的搭建主要有两种，一种是使用Hexo，另一种是Jekyll。我选择Jekyll主要原因是方便，甚至可以不需要在本地配置环境，而且jekyll的文章是直接放到`_posts`目录中的，方便以后修改。

## 搭建过程

我使用的是最简单也是最笨的方法，在别人代码的基础上进行修改。

### 资源

这是我找的一些感觉很好的资源：

- [Jekyll搭建个人博客 ](http://baixin.io/2016/10/jekyll_tutorials1/)
- [用Jekyll搭建的Github Pages个人博客 ](http://louisly.com/2016/04/used-jekyll-to-create-my-github-blog/)
- [Github Pages + Jekyll 独立博客一小时快速搭建&上线指南 - Bebop.C的博客 | Bebop.C Blog ](http://playingfingers.com/2016/03/26/build-a-blog/)
- [Gaohaoyang/gaohaoyang.github.io ](https://github.com/Gaohaoyang/gaohaoyang.github.io/blob/master/README-zh-cn.md)
- [mmistakes/minimal-mistakes: A flexible two-column Jekyll theme. Perfect for personal sites, blogs, and portfolios hosted on GitHub or your own server. ](https://github.com/mmistakes/minimal-mistakes)

###创建仓库

首先在github上创建一个新的仓库，命名要求是`{用户名}.github.io`，将fork或者download的代码放进去，在仓库设置启用GitHub Pages，这样就可以直接使用`{用户名}.github.io`来访问博客地址了；

我使用的是 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io)，最主要的原因是在每篇博客的侧边栏是有文章的目录的，而且作者还给出了详细的搭建介绍文档；

### 主要修改

首先将作者的文章清空，也就是将`_posts`文件夹清空；

将`favicon.ico`换成自己喜欢的图片，这个是网站访问时，标签页显示的图标；

可以将`page`文件夹中自己不需要的页面删掉，然后修改`Collection`和`About`文件中的内容；

将博客中有关作者的信息修改成自己的信息，主要修改`_config.yml`文件；

### 我的修改

参照[Brianway's personal site](https://github.com/brianway/brianway.github.io)的做法，将文章归档页面(`page->category.html`)进行了修改，年份中又有对月份的归档；

之前作者是使用四个回车`\n\n\n\n`作为首页上显示的文章简介和文章正文的分隔标志的，我改成了`<!--more-->`，这个问题也有人提出过，可能是作者比较忙，还没来得及更新文档；

## 其它

因为之前太懒，现在有好多学习的“债”要还，最久感觉时间总是很紧张，可能没有那么多的时间写博客，更多的可能是把自己找到的一些不错的资源总结一下，但是还是希望自己能多写一些，毕竟在我看来写博客是一件很酷的事。