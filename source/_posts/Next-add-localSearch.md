---
title: Next增加搜索localSearch
date: 2017-10-19 16:06:51
tags: 博客搭建和功能增强

---

Next本身增加localSearch很简单，三步即可：

## 安装
安装 hexo-generator-searchdb，在站点的根目录下执行以下命令：

```
$ npm install hexo-generator-searchdb --save
```

编辑 站点配置文件，新增以下内容到任意位置：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

编辑 主题配置文件，启用本地搜索功能：

```
# Local search
local_search:
  enable: true
  
```

## 遇到问题，一直loading

发布了之后，点击搜索一直在loading，查了其他人的文章[Hexo next 主题的 local search 功能失效，点击搜索链接无法弹出叠加层
](https://www.v2ex.com/amp/t/298727)，发现是`search.xml`出现了问题。

排查问题步骤：

1. command + option + J 打开调试器
2. 点击NetWork，发现卡在了`search.xml`上面
3. 尝试debug `serach.xml`文件，打开 [https://njafei.github.io/search.xml](https://njafei.github.io/search.xml)
4. 看到警告： `error on line 92 at column 35: Input is not proper UTF-8, indicate encoding ! 0x10 0xE6 0x88 0x96`
5. 编码问题，找到对应的文章，先对其进行`utf-8`编码，然后放到sublimeText中，我发现中间多了一个类似于`DEL`的乱码，删除即可
6. 重新发布，搜索可用


## 缓存问题

search有时候会有缓存，这时候，可以打开对应的`search.xml`文件[https://njafei.github.io/search.xml](https://njafei.github.io/search.xml)，刷新几下即可

> PS： 如果你想刷新你的文件，把前面的链接替换即可。
