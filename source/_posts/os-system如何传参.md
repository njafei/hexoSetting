---
title: os.system如何传参
date: 2017-05-26 11:21:12
tags: [python]
categories: python

---
今天写脚本的时候，正好希望在python脚本中调用另外一个python脚本，就使用了os.system来实现这个功能（当然，还有很多的办法，而且每种方法满足的需求不都一样，有兴趣的读者可以谷歌下）。

但是我需要给这个调用里面传入一个参数，网上查了半天，感觉都不太清晰，就写了这篇，简单介绍下。

os.system的定义是这样的

```
os.system("shell command argusFormat" % argus)
```

在双引号里面正常写命令，需要用到参数的地方，使用%s等格式代替，然后在双引号的后面加空格，加%号，然后在括号里写入所有的参数，用逗号隔开。

#### 单个参数

```
param = 'I'm param'
os.system("python haha.py %s" % (param))
```
#### 多个参数

```
paramA = 'I'm paramA'
paramB = 'I'm paramB'
os.system("python haha.py %s %s" % (paramA,paramB))
```

需要注意的是，shell中对于空格的要求特别严格，一定要注意别多或者少（写js的来写shell真的好难受0_0）。

#### python格式化


```
%s    字符串 (采用str()的显示)

%r    字符串 (采用repr()的显示)

%c    单个字符

%b    二进制整数

%d    十进制整数

%i    十进制整数

%o    八进制整数

%x    十六进制整数

%e    指数 (基底写为e)

%E    指数 (基底写为E)

%f    浮点数

%F    浮点数，与上相同

%g    指数(e)或浮点数 (根据显示长度)

%G    指数(E)或浮点数 (根据显示长度)

%%    字符"%"
```


 