---
title: github pages + hexo 原理探究
date: 2017-03-18 21:07:15
categories: 闲来研究
---

缘起
----
利用一个下午搭建起了github pages + hexo 的静态博客，试用了下效果，还是挺满意的，至少比csdn的页面好看多了，哈哈。
搭建的过程中，也遇到了很多的问题，尤其是中间如何把hexo 和github连在一起，又是如何更新博客这块，有很多疑问，我自己尝试，还不小心把本地的hexo的文件删除了，不得不重新init了一个，配置都要重新来一遍，所以这篇文章着重探讨，github pages的流程到底是如何展开的。

简单介绍
----

先说下[github pages](https://pages.github.com/),github pages 是github提供给用户用来展示个人或者项目主页的静态网页系统。每个用户都可以使用自己的github项目创建，上传静态页面的html文件，github会帮你自动更新你的页面。

[hexo](https://hexo.io/zh-cn/)是一个用来生成静态界面的框架，使用hexo，你就可以直接使用mark down 来写文章，而不用关心前端样式的展现。

数据流
----

我们来看下流程图：
![数据流程图](http://on0hv7n2x.bkt.clouddn.com/github%20pages%20%E6%95%B0%E6%8D%AE%E6%B5%81.png)

这样看就很简单了，我们本地使用markDown语法写好文件，然后执行hexo或者其他静态网页生成工具，生成好静态文件，然后使用hexo等工具的发布功能，就会使用ssh来更新github项目的文件，即生成一个新的commit然后push。github检测到这个项目更新，就会更新你的网站的内容（会有缓存）。
