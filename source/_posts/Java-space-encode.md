---
title: Java对空格的encode格式问题
date: 2017-09-22 16:57:41
tags: 小知识
---

今天发现了一个非常奇怪的问题，服务端给了一个encode的url，内容是位“1 1”：

encode之后是

```
1+1
```

前端decode之后是

```
1+1
```

可是在其他的浏览器等decode之后的结果是：

```
1 1
```

发现java的encode有个坑，在java中，encode遵循的标准是[rfc1738](http://www.faqs.org/rfcs/rfc1738.html)，而在iOS中，encode遵循的标准是[rfc2396](http://www.faqs.org/rfcs/rfc2396.html)，两者对于空格的encode不同

```
rfc1738 => +
rfc2396 => %20
```

所以今后和Java服务端合作的时候，如果encode之后的文案出现了+号这种莫名奇妙的bug，记得让他们检查下encode的代码，可以通过替换来解决这个问题。