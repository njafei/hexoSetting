---
title: Apple文档翻译之Event-Handing-Guide-for-iOS
date: 2017-03-27 15:38:29
tags: 
    - Apple文档
    - 基础知识
categories: 文档翻译
---

# About Events in iOS
用户会使用很多的方式来操作他们的iOS设备，比如点击屏幕或者摇晃屏幕。当用户正在操作硬件或者向App传递信息时，iOS会获取时间和方式。你的App给用户的反馈越自然、越直接，用户就会越有兴趣。

![](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Art/events_to_app_2x.png)

#### AT a Glance
Event是当用户有动作时，（`UIKit`）向App发出的通知。在iOS中，有很多形式的event：多点触摸，滑动、多媒体控制。最后一个事件被熟知为远程控制事件，因为它可以从一个附件中发出。

#### UIKit让你的App处理手势更轻松
iOS应用可以识别组合的触摸，然后直接以特定方式反馈，比如收缩的触摸会缩小内容，间隔触摸划过内容等。实际上，很多的手势太常见了，所以他们被内置在`UIKit`中。比如[UIControl](https://developer.apple.com/reference/uikit/uicontrol)子类，比如[UIButton](https://developer.apple.com/reference/uikit/uibutton)和[UISlider](https://developer.apple.com/reference/uikit/uislider)，回应特殊的手势-button的点击和slider的拖动。当你配置这些control，他们会在触摸发生时，发出一个message给目标。你也可以通过使用gesture recognizers在view上实现这个目标动作。当你给一个view添加一个getsture recognizer，这个view会像一个control一样给你指定的触摸回应。

Gesture recognizer提供了一个高度抽象的复杂事件处理逻辑。当使用触摸事件时，请优选gesture recognizer，因为它们很强大，可复用，适用性强。你也可以使用内置的getsure recognizer然后个性化它的行为。或者你可以创造一个新的gesture recognizer来识别一个新的触摸事件。

> 相关章节：[Gesture Recognizers](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html#//apple_ref/doc/uid/TP40009541-CH2-SW2)

##### 一个事件为了找到处理它的对象，要遍历一个特定的路径
当iOS recognizer是事件时，它首先会将事件传递给初始的看起来最相关的对象，比如touch发生的view。如果初始的对象不能处理事件，`iOS`会继续传递事件给更大范围的对象，直接它找到了一个对象有足够的上下文处理这个事件。这些对象的序列，就是被熟知的响应链（responder chain），iOS会顺着响应链传递事件，同时它也传递回应事件的责任。这种设计模式让事件处理更加动态和合作性高。
> 相关章节 [响应链 Responder Chain](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW2)

#### UIEvent封装了触摸，摇晃，远程控制事件
许多`UIEvent`是UIKit [UIEvent](https://developer.apple.com/reference/uikit/uievent)的实例。一个`UIEvent`包括了关于事件app需要使用的信息，以决定如何回应这个事件。比如当一个用户事件发生时，手指触摸屏幕或者滑动它的表面，iOS持续的发送event对象给app。每个event对象都有一个type-触摸、摇晃、远程控制来作为它的子类型。

> 相关章节： [多点触摸](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/multitouch_background/multitouch_background.html#//apple_ref/doc/uid/TP40009541-CH5-SW9)、[手势](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/motion_event_basics/motion_event_basics.html#//apple_ref/doc/uid/TP40009541-CH6-SW14)、[远程控制](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Remote-ControlEvents/Remote-ControlEvents.html#//apple_ref/doc/uid/TP40009541-CH7-SW3)


#### 当用户触摸界面时，App接收多点触摸事件
在你的App上，UIKit controls和gesture recognizers 也行足够你来应对触摸事件了。甚至你可以使用Gesture recognizer来定制你的view。根据经验，你会使用触摸事件当你的app回应和view本身绑定的事件，比如摸出画东西。在这些列子中，你只负责低优先级的触摸事件，事件touch method，在method中，你分析未经加工的触摸事件，然后适当回应。

> 相关章节 [多点触摸 multitouch Event](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/multitouch_background/multitouch_background.html#//apple_ref/doc/uid/TP40009541-CH5-SW9)


#### 当用户移动设备时，App接收到移动事件（Motion Event）
Motion事件提供了关于设备的地点、方向和移动的信息。通过回应Motion事件，你可以提供微量却强大的功能。加速器和陀螺仪数据能让你判断是倾斜、旋转，还是摇晃。

Motion事件有许多形式，你可以通过不同的framework来处理他们。当用户摇晃设备时，`UIKit`传递`UIEvent`对象给app。如果你想要app能接收高速、持续的加速器和陀螺仪数据，请使用`Core Motion framework`。

> 相关章节 [Motion Event](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/motion_event_basics/motion_event_basics.html#//apple_ref/doc/uid/TP40009541-CH6-SW14)


#### 当用户操作多媒体控制，用户收到远程控制事件

iOS controls 和外部附件可以给app发送远程控制事件。这些事件允许用户去控制音频和视频，比如通过耳机调整音量。

> 相关章节： [Remote Control Event](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Remote-ControlEvents/Remote-ControlEvents.html#//apple_ref/doc/uid/TP40009541-CH7-SW3)
