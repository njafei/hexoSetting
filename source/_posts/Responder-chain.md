---
title: iOS 响应链 Responder Chain
date: 2017-03-29 15:12:33
tags: 基础知识
categories: iOS基础
---

# Event
iOS设备和用户的交互其实有很多种方式，包括触摸屏幕，摇晃设备，多媒体控制（音量等）等。当设备检测到这三种事件中的一种的时候，iOS就会捕获当前事件的时间和内容，然后发出消息，通知App发生了事件，这个就是`Event`。
![UIEvent category](http://on0hv7n2x.bkt.clouddn.com/UIEvent_Type.png)


# Responder
有了`Event`之后，系统需要找到能够处理这个`event`的对象，而能够处理event的对象，就是`UIResponder`.我们常用的`UIApplication`、`UIView`、`UIViewController`，其实都是`UIResponder`的实例。`UIResponder`为了要处理特定的事件，要实现`corresponding`方法。比如touch事件，那么`Responder`就要实现`touchesBegan：withEvent`,`touchedMoved:withEvent:`等方法。

![UIResponder](http://on0hv7n2x.bkt.clouddn.com/UIResponder.png)

# Responder Chain
如果有当前的界面有多个Responder的时候，到底是谁来响应呢？这里其实是根据特定的一个顺序，依次寻找响应的对象，这个就叫响应链（`Responder Chain`）机制了。在`Responder Chain`中，系统首先`UIkit`会按照一定的规则（下节会讲到）来找到当前的`first responder`，通常是事件发生的`view`，我们叫做`initial view`,然后根据这个view是否有处理的能力（比如touch事件就是看是否实现了上述的`touchesBegan：withEvent`等方法），如果当前的`responder`无法处理该事件，那么会响应链的顺序来查找下个响应者，通常的顺序是： view -> super view -> viewController -> window -> UIApplicaiton，如下图：
![ResponderChain](http://on0hv7n2x.bkt.clouddn.com/ResponderChain.png)

如果到了响应链的最后一个节点，还是没有找到响应者，那么系统就会抛弃这条event。

# Hit-Test View，Hit-Testing
那么系统是如何找到`first responder`的呢？答：使用`Hit-Testing`。
当触摸事件发生时，iOS会使用`Hit-Testing`超找当前事件所在的`view`，它会从`UIApplication`开始，依次往下寻找事件发生的`view`，直到找到这个`view`，顺序和上述的响应链，其实是相反的，我怀疑可能响应链就是在`Hit-Testing`的过程中建立的，但是没有查到相关的资料。

我们看个苹果的官方列子：
![](http://on0hv7n2x.bkt.clouddn.com/Hit-Testing.png)

如果触摸事件发生在D中，那么系统会依次检查：

1. 触摸区域在A中，遍历它的所有subViews
2. 触摸区域在C中，遍历它的所有subViews
3. 触摸事件在D中

D就是所谓的`Hit-Test View`，也是上述的响应链中的`first responder`


#### 参考文档
* [EventHandlingiPhoneOS 官方文档忽然实效了](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html)
* [Responder object](https://developer.apple.com/library/content/documentation/General/Conceptual/Devpedia-CocoaApp/Responder.html#//apple_ref/doc/uid/TP40009071-CH1-SW2)
* [iOS Events and Responder Chain](https://www.zybuluo.com/MicroCai/note/66142)
* [iOS中的响应链（The Responder Chain）](http://www.jianshu.com/p/05cbcd774f45)
