---
title: Apple文档翻译之UIResponder
date: 2017-03-21 15:20:54
tags: 
categories: 文档翻译
---


UIResponder是回应和处理事件的抽象接口,UIResponder的实例，包括：UIApplication、UIViewController、UIView（包含UIWindow），组成了UIKit处理事件的核心。当事件发生时，UIKit会把事件派发给UIResponder去处理。

有许多种事件，包括触摸、手势、遥控和点击事件。为了处理特定的事件，一个responder（响应）必须重写corresponding方法。比如，要想处理touch事件，一个responder要实现touchesBegan：withEvent,touchedMoved:withEvent:等方法。在触摸事件的列子中，responder使用UIKit提供的事件信息来追踪触摸的改变，然后适时地更新界面。

除了处理事件，UIKit responder还负责转发那边没有被处理的事件给app的其他部分。如果一个指定的responder没有处理事件，它就会转发这个事件到响应链的下一个responder。UIKit动态地管理响应链，使用预定的规则来决定下一个收到事件的对象。比如：一个view转发事件到它的superView，或者Rootview转发事件给它的view controller。

Responder处理UIEvent和其他通过输入界面的自定义输入，最典型的就是系统的键盘。当用户点击UITextField和UITextView，这个view变成了第一个responder，然后展示它的输入界面：键盘。同样的，你可以创造一个自定义的输入界面，然后在其他responders活跃的时候展示它。为了把输入界面和responder联系在一起，要把view赋值给responder的属性。


原文链接  [UIResponder](https://developer.apple.com/reference/uikit/uiresponder#//apple_ref/occ/cl/UIResponder)

如果有任何建议和优化的地方，欢迎给我留言。
