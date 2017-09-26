---
title: ImageOptm 自动化无损优化图片
date: 2017-09-26 11:01:58
tags: 自动化

---

iOS控制包的大小对于公司来说，是个非常重要的事情。因为苹果公司对于包的大小超过100M的，不会允许用户使用移动网络来下载，这很可能造成商业上的损失。

而在控制包的大小中，一个很重要的原则就是禁止大图片。之前研究其他公司的ipa包的时候，就出现过一个icon高达1M的事故。而平时，虽然程序员们百般小心，难免被设计师暗算给张大图（玩笑），所以靠人终究不是一个可持续的保证质量的方法。

平时用的无损压缩最多的工具是`ImageOptm`，它可以无损压缩图片（即用户看起来感官无变化，而图片尽可能小），而且它是提供命令行的，[ImageOptm command line](https://imageoptim.com/command-line.html).

我们就可以通过Jenkins的定时任务，每天凌晨去优化所有的图片，然后再自动commit上传。

具体任务就很简单了，大概几个步骤：

1、Jenkins开个定时任务，更新代码

2、执行ImageOptm脚本

```
/Applications/ImageOptim.app/Contents/MacOS/ImageOptim  $WORKSPACE/
```

3、上传代码