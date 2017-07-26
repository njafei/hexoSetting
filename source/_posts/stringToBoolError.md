---
title: Error RawText ** must be wrapped in an explicit <Text> component 问题解决
date: 2017-06-14 11:05:55
tags: [奇怪的bug,React-Native]
categories: bugFix

---

    
今天忽然遇到一个页面崩溃，查到错误如下：

```
Error: RawText "" must be wrapped in an explicit <Text> component.    
```
查了代码，发现好久都没有动这块的代码了，所以非常疑惑。最后通过2分法不停地查哪里出了问题，最终查到了这个语句：

```
return (
    <View>
        {test && test.string &&
        <Text>{test.string}</Text>
        }
    </View>
)
```

其实作用很简单，如果string有值，则展示string。但是这条语句为什么会报错呢？查了半天，发现是因为string的值是'',然后系统就报错了，类似这样：

```
let test = {string: ''};
return (
    <View>
        {test && test.string &&
        <Text>{test.string}</Text>
        }
    </View>
)
```

后来查了下github，发现很多人也遇到了类似的错误，解决办法如下,使用!!来判断string是否有值，因为这里其实你是希望将`string`当成`bool`来使用的。

```
let test = {string: ''};
return (
    <View>
        {test && !!test.string &&
        <Text>{test.string}</Text>
        }
    </View>
)
```

所以以后string的判断，都用!!去判断，否则出现string恰好为''的时候，就会崩溃。

参考文章：

 - [https://github.com/GeekyAnts/NativeBase/issues/186](https://github.com/GeekyAnts/NativeBase/issues/186)