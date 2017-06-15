---
title: block
date: 2017-06-13 13:20:58
tags: Apple文档
categories: iOS基础

---

其实使用`block`的时间也蛮久了，比如使用`__block`,`weak`防止循环引用，`copy`修饰等注意点也都知道，但是一直没有去看过官方的文档，仔细看下所有的点。今天就把官方文档撸一遍，深入了解下。


# 使用场景
苹果在介绍`block`的使用场景时，是这么说的：

> You use a block when you want to create units of work (that is, code segments) that can be passed around as though they are values. Blocks offer more flexible programming and more power. 

当你想要做一连串的工作时（代码片段），block可以被当成值来传递，从而使程序更加灵活、易用。

我觉得，这个类似于`JS`的`callback`，这样写看起来比较直观，不像代理，满天飞，易读性不好。

# 声明和使用

正常使用`block`类似于这样：

```
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
    return num * multiplier;
};
```

以前其实我自己在写block的时候，很容易迷糊，因为格式容易记不住，但是看了下面的图，仔细了解下它的结构，以后就好记了

![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Art/blocks.jpg)

首先是它的返回值，然后是它的name，接着是参数的类型，声明和实现结构是一样的，只是实现里面会多形参。


看下如何使用：

```
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
    return num * multiplier;
};
 
printf("%d", myBlock(3));
// prints "21"

```

在苹果提供`block`之后，它提供了大量的`block`的函数用法， 比较多的应该就是数据等的遍历。我们自己在写代码的时候，通常数据请求、alert等也都会封装成`block`的形式，因为真的是易读性比较好。

# block和变量

在`block`中，是可以直接引用外部的变量的，比如：

```
int a = 0;
block(a);
```

但是不能直接修改外部的变量，比如：

```
int a = 0;
block = ^(int a){
	a = 5;//error
}
```

#### __block
如果你想要在`block`中改变外部变量的值的话，需要使用`__block`来修饰，为什么这个关键词可以呢？我们来看下苹果的解释

> __block variables live in storage that is shared between the lexical scope of the variable and all blocks and block copies declared or created within the variable’s lexical scope. Thus, the storage will survive the destruction of the stack frame if any copies of the blocks declared within the frame survive beyond the end of the frame (for example, by being enqueued somewhere for later execution). Multiple blocks in a given lexical scope can simultaneously use a shared variable.
> 
As an optimization, block storage starts out on the stack—just like blocks themselves do. If the block is copied using Block_copy (or in Objective-C when the block is sent a copy), variables are copied to the heap. Thus, the address of a __block variable can change over time.
> 
There are two further restrictions on __block variables: they cannot be variable length arrays, and cannot be structures that contain C99 variable-length arrays.

`__block`会把变量放到`block`等的存在的内存中，这样在`block`存在期间，可以直接修改变量，不同的`block`都使用这个共享的变量。

`block`默认是存在栈中的，但是如果`block`被拷贝，则会放到堆中

`__block`有两个注意点，不能是可变化长度的数据，也不能是[C99标准可变长度的数组](https://en.wikipedia.org/wiki/Variable-length_array)


# 注意点

 1. 为什么要用copy
 2. 为什么要避免循环引用，如何避免
 3. 避免两个错误模式

#### 为什么要用copy
如上面所述，`block`默认是存在栈中的，但是如果`block`被拷贝，则会放到堆中，所以即使你使用strong，其实系统也是会copy一份的，用copy是为了让自己知道，这个`block`在使用中是会被copy一份的。那么，为什么呢？这因为如果`block`在栈中，则他的作用域就是栈中所在的作用域，如果在作用域外调用栈的内容，则会崩溃，所以要用copy，就会将其复制到堆中，调用就不会出现问题。当然，用strong也不会有任何问题，但是copy更能让我们使用的时候，知道使用的原因。

#### 循环引用，如何避免
先看下代码：

```
self.printBlock = ^(block)(NSString *) = ^(NSString *name){
	[self print: name];
}
```

在这里，self会对block强引用，而block中，又会对self强引用，所以系统在回收内存的时候，这两个对象都没有办法回收内存，就造成了内存的泄露了。

怎么避免呢？

```
__weak typeof(self) weakself = self
self.printBlock = ^(block)(NSString *) = ^(NSString *name){
	__strong __typeof(self) strongSelf = weakSelf;//里面使用strong，防止执行block的时候 self被销毁
	[weakself print: name];
}
```

#### 避免两个错误模式

苹果在文档中，特意提了两个错误的模式，让大家不要使用，如下：

```
void dontDoThis() {
    void (^blockArray[3])(void);  // an array of 3 block references
 
    for (int i = 0; i < 3; ++i) {
        blockArray[i] = ^{ printf("hello, %d\n", i); };
        // WRONG: The block literal scope is the "for" loop.
    }
}

void dontDoThisEither() {
    void (^block)(void);
 
    int i = random():
    if (i > 1000) {
        block = ^{ printf("got i at: %d\n", i); };
        // WRONG: The block literal scope is the "then" clause.
    }
    // ...
}
```

这两个用法本身是可以执行的，但是效率太差。本来block是内联的，所以应该先定义，再使用，而不是上文的用法，每次都重新定义一个。

PS： 内联函数的效率非常高，可以理解为内联是之前把代码片段嵌入到使用的地方，而非内联的函数就是要调用函数了

