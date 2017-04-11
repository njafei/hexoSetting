---
title: 如何给同一个电脑上的不同git项目设置不同的name和email
date: 2017-04-11 14:38:15
tags: git
categories: 小知识
---

最近在自己的电脑上同时使用github和公司的git仓库，带来了一个问题就是之前只是设置了全局的name和email，但是两边的代码需要使用不同的user，每次都要手动去改，然后我搜索了下，发现其实我们可以给每个git项目，单独配置一个name和email的。规则如下：

> 如果项目由独立配置，则使用独立配置，如果没有独立配置，则使用全局配置

命令就很简单了：
全局name和email配置：

```
$ git config --global user.name gitaccount
$ git config --global user.email gitaccount@example.com
```

给单独的git项目设定配置：

```
$ cd gitFolder
$ git config --global user.name gitaccount
$ git config --global user.email gitaccount@example.com
```

这样，如果本地有多个git user 或者多个项目的话，使用起来就比较方便了。
