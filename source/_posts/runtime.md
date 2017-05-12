---
title: iOS Runtime 详解
date: 2017-05-04 15:45:12
tags: 
		- Apple文档
		- 基础知识
categories: iOS基础

---

# 什么是runtime

依照苹果文档的说法，`runtime`是：

>  The Objective-C language defers as many decisions as it can from compile time and link time to runtime. 
（尽量将决定放到运行的时候，而不是在编译和链接过程）

如何理解这段话呢，我们首先要知道，一段代码从写完到最终执行的过程中发生了什么。

#### 从代码到可执行文件的过程

这是《深入理解计算机系统(第2版)》里面的一张截图：

![](http://on0hv7n2x.bkt.clouddn.com/compileToLinkToRun.png)

主要过程我们可以简化成三个：

	- 编译
	- 链接
	- 运行
 
编译：将代码转换成底层可执行的语言（如汇编），简单来讲，就是把你能看懂的语言，转换成系统底层可以看懂的东西，这中间通常会有优化，先预处理，再编译。

链接：在编译的过程中，如果有调用其他的类的方法等，是不会检查或者报警的，编译的时候会默认你已经实现了。而链接就是去检查调用的方法或者类等是否确实存在。

运行：执行最终的可执行文件

如果是普通的C语言代码，我们使用的是传统的编译运行，那么一个函数的执行内容，在编译阶段其实就确定了，执行的时候只要去执行对应的内存地址的程序就好。

而在`runtime`中，编译阶段只能确定最终要执行的函数名，但是具体执行的时候，执行的是什么程序，是在运行的时候才能确定，大大增加了程序的灵活性。

# Objective-C runtime介绍
#### 简介
Objective-C是一门运行时语言，这意味着代码执行可以更加灵活：我们动态的创建一个新的类，还可以转发消息给其他的消息。（消息转发是runtime的一个重要组成部分，后面会介绍）。

这种特性要求Objective-C语言会尽可能把决定从编译和链接的阶段延迟到运行时`(runtime)`阶段，因此Objective-C不仅有一个编译器，还有一个`runtime`系统来执行被编译过的代码。

#### 版本和平台

runtime是有个两个版本的: legacy 、 modern
在Objective-C 1.0使用的是legacy，在2.0使用的是modern。这里简单介绍下区别：

 - 在legacy runtime，如果你改变了实例变量的设计，需要重新编译它的子类。支持 32bit的OS X 程序
 - 在modern runtime，如果你改变了实例变量的设计，不需要重新编译它的子类。支持iphone程序和OS X10.5之后的64bit程序

因为legacy是如此的古老，我们基本可以忽略legacy版本。

# Runtime交互
有三种方式可以使用Runtime：

 - Objective-C 源代码
 - NSObject 方法
 - Runtime 方法

#### Objective-C 源代码

一般来说，runtime都是默默地在后台运行工作，我们只是写Objective-C源代码，就使用了runtime。当我们编译包含Objective-C的类和方法时，编译器就会生成包含runtime特性的数据结构和方法。数据结构中包含了从类、category、协议中定义的的信息，如：selector、变量等等。主要的runtime方法是发送信息的方法[Message](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html#//apple_ref/doc/uid/TP40008048-CH104-SW1),在下一章节会讲到。

#### NSObject方法

Cocoa中大部分的类都是继承自`NSObejct`，所以他们都会集成了`NSObject`定义的方法。（值得注意的例外是`NSProxy`类）因此`NSObject`的方法就决定了其他类的行为。（当然，在少数情况下，这样说并不正确，在这些情况下，`NSObject`会仅仅定义了方法，没有实现必须的代码。比如`description`，如果子类没有重写，那么会返回类名和地址，这是因为`NSObject`没法获得更多的信息。）

一些`NSObject`类的方法可以直接查询`runtime`系统的信息，从而获得自身的信息。比如`class`，`isKindeOfClass`，`respondsToSelector`等方法。

#### Runtime方法

Runtime系统是一个共享的library，包含了许多方法和数据结构，地址在`/usr/include/objc`。其中的很多方法，允许你使用C来重写编译行为，其他的方法是通过`NSObject`类使用。有了这些方法，我们就可以为`runtime`系统增加接口，或者工具。


# Message
先抛出来一个问题，这句话代表什么？

`[receiver message]`

receiver执行message函数？ 是这个作用,但是more than that，等价于这行代码

```
objc_msgSend(receiver, @selector(message))
```

所以其实message是iOS中非常重要的一环，尤其是在动态绑定中。下面我们具体看下：

#### objc_msgSend
上面讲了，iOS中的函数调用其实是给实例发送了message，有参数的函数其实执行了：

```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```

`objec_msgSend`的方法定义如下：

```
OBJC_EXPORT id objc_msgSend(id self, SEL op, ...)
```

那么，消息转发的过程究竟发生了什么呢？
我们先看下这里用到的三个类：对象(object)，类(class)，方法(method)

```
//对象
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

//类
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;

//方法列表
struct objc_method_list {
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

//方法
struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}

```

 1. 系统首先找到消息的接收对象，然后通过对象找到它的类。
 2. 在它的类中查找`method_list`，是否有selector方法。
 3. 没有则查找父类的`method_list`
 4. 找到对应的`method`，执行它的`IMP`
 5. 转发IMP的return值

selector和IMP、method等的区别，可以参考我的另一篇博客[Method,SEL,IMP](https://njafei.github.io/2017/05/03/Method-SEL-IMP/)

>  注意：编译器会生成messaging方法，所以你永远都不应该手动调用这个方法。

#### dispatch table
messaging的关键在于编译器给每个类创建的数据结构，每个类的数据结构都包含两个要素：

 - 执行父类的指针
 - 一个类分发表(dispatch table)。表中有所有的相关方法和这些方法的地址和id。

> runtime系统，要求对象必须等价于`objc_object `,`NSObject`和`NSProxy`都自动包含`isa`属性。

结构和原理如图所示：

![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif)

当消息被发送给对象的时候，消息就按照上图的路径寻找对应的`selector`，直到达到`NSObject`，如果在`NSObject`中还是没有找到对应的方法，则会走到消息转发机制中，下文会介绍。

#### cache
为了加速消息分发， 系统会对方法和对应的地址进行缓存，就放在上述的`objc_cache`，所以在实际运行中，大部分常用的方法都是会被缓存起来的，`runtime`系统实际上非常快，接近直接执行内存地址的程序速度。

# Message Forwarding
如果在`dispatch table`中没有找到对应的`method`呢？ 系统仍然会给你补救的机会：

 - resolveInstanceMethod/resolveClassMethod
 - fast forwarding
 - normal forwarding

#### resolveInstanceMethod
系统没有在`dispatch_table`中找到对应的方法，会看你是否重写了`resolveInstanceMethod`，`resolveClassMethod`,这里两个方法是否用来添加动态方法的，一个是实例方法，一个是类方法。

举个例子：我们有个ClassA

```
#import <Foundation/Foundation.h>

@interface ClassA : NSObject

@end

@implementation ClassA

@end
```

希望执行ClassA并不存在的foo方法

```
ClassA *a = [ClassA new];
[a performSelector:@selector(foo)];
```

系统会直接报错：

```
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[ClassA foo]: unrecognized selector sent to instance 0x600000004e00'
```

如果我们想用`resolveInstanceMethod `来补救，该怎么做呢？

```
#include <objc/runtime.h>

void foo(id self, SEL _cmd) {
    NSLog(@"resolveInstanceMethod add method foo ");
}

@implementation ClassA

+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(foo)) {
       class_addMethod([self class], sel, (IMP)foo, "v@:");
       return YES;
    }
    return [super resolveInstanceMethod:sel];
}

```

> 注意：一定要记得import `<objc/runtime.h>`，否则会报错：`class_addMethod` is valid in C99

执行下，log：

```
resolveInstanceMethod add method foo 
```

这里的return YES 或者 return NO,是告诉系统是否实现了这个方法，如果return YES，但是并没有增加方法，还是会报错，并且不会走到forward，因为系统默认你已经在这一步做了resolveInstanceMethod这个事情。


#### forwardingTargetForSelector
如果上一步骤的`resolveInstanceMethod ` return no，系统会走`forwardingTargetForSelector `，这一步被称为快速转发，是因为相对下面要介绍的normal fastward，这一步直接转发了消息，而normal fastward生成了NSInvocation，相对直接转发慢一些。

先看下如何实现，比如，我想把消息转发给有能力的classB：

```
@interface ClassB : NSObject

- (void)foo;

@end

@implementation ClassB



- (void)foo
{
    NSLog(@"ClassB foo run");
}
@end

```

A中需要实现`forwardingTargetForSelector`方法：

```
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(foo)){
        ClassB *b = [ClassB new];
        return b;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

run一下，log：

```
ClassB foo run
```

苹果的文档里，讲述了这一个消息转发的出发点，其实是为了实现类似C多继承的功能。我们知道，在C中如果一个类想要具有多个类的功能，是可以直接继承多个类的。而Objective-C是单继承，如果想实现类似的功能，就用消息转发，将消息转发给有能力处理的类。苹果是这样描述他们的思想的：C的多继承，是加法，在多继承的同时，其实也增加了很多不需要的功能，而苹果通过消息转发，实现了减法的思想，只留有用的方法，而不去增加过多内容。

#### forwardInvocation
如果你的类没有实现`forwardingTargetForSelector`方法，系统会调用`methodSignatureForSelector`方法，如果这个方法返回一个函数的签名，则执行`forwardInvocation`方法，否则执行`doesNotRecognizeSelector`。

如果希望在这一步补救，如何做呢？

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    return [ClassB instanceMethodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
    SEL sel = invocation.selector;
    ClassB *b = [ClassB new];

    if([b respondsToSelector:sel]) {
        [invocation invokeWithTarget:b];
    }
    else {
        [self doesNotRecognizeSelector:sel];
    }
}

```

#### 流程图
我自己画了个消息转发的流程图：

![](http://on0hv7n2x.bkt.clouddn.com/iOS-message-forwarding.png)


# 其他
#### 隐藏参数
刚才讲了，在一个对象执行一个函数的时候，其实是：

```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```
那其实在函数中，`receiver`和`selector`是两个隐藏的参数，这两个参数是可以使用的。

```
- (void)run
{
    [self performSelector:_cmd];  //self: 当前对象  _cmd : "run"
}
```

##### 获取method的地址
如果你要连续执行同一个method，但是觉得每次都要遍历一遍分发表会效率低，可以直接获取地址(methodForSelector)，然后直接执行函数.

```
void (*setter)(id, SEL, BOOL);
int i;
 
setter = (void (*)(id, SEL, BOOL))[target
    methodForSelector:@selector(setFilled:)];
for ( i = 0 ; i < 1000 ; i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```

个人觉得，这样意义不大，因为其实系统会做缓存。


# runtime实际应用

runtime的应用，主要有几种：

 - AOP,切面编程，做打点
 - method swizzling,黑魔法做崩溃等的保护

因为主要是使用`method swizzling`来做，我将会在之后的博客中专门介绍。


参考文章：

 - 关于编译和链接，可以看下 [http://www.cprogramming.com/compilingandlinking.html](http://www.cprogramming.com/compilingandlinking.html) 这篇文章。
 - [ObjCRuntimeGuide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)
 - [http://stackoverflow.com/questions/3900549/what-is-runtime](http://stackoverflow.com/questions/3900549/what-is-runtime)
 - [http://tech.glowing.com/cn/objective-c-runtime/](http://tech.glowing.com/cn/objective-c-runtime/)
 - [http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/](http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/)
 - [http://tech.glowing.com/cn/objective-c-runtime/](http://tech.glowing.com/cn/objective-c-runtime/)


