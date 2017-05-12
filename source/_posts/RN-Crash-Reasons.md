---
title: RN 最容易crash的代码用法及应对措施（持续更新）
date: 2017-04-17 10:37:55
tags: React-Native
categories: React-Native
---

## 变量保护

出现最多的就是在使用redux来做数据层，使用this.props的属性没有去查询时候为undefined，这种情况，基本是必crash

比如下例：

```
//show user name
<Text>{this.props.userInfo.name}</Text>
```

如果userInfo为undefined的话，就会崩溃，错误如下：
> TypeError: Cannot read property 'name' of undefined

在这里name为undefined的时候反而没有问题，因为name是一个简单的属性，直接赋值给Text是没有问题的。

那如何避免这种问题呢？在赋值前加下判断会比较好：

```
let name = this.props.userInfo && this.props.userInfo.name ? this.props.userInfo.name : '';

//show user name
<Text>{this.props.userInfo.name}</Text>

```

这样基本可以避免崩溃的问题了。

但是如果都这样判断，实际是比较复杂的，所以如果你的业务比较简单，我建议可以直接在render做一个大的保护，即没有数据的时候，不去render这些业务内容.
思路如下：

```
render(){
	if(!this.props.userInfo){
		return (
			<EmptyView />
		)
	} else {
		return (
			//注意，如果name的层级更深，还是建议做保护
			<Text>{this.props.userInfo.name}</Text>
		)
	}
}


```


## 定时器
定时器其实在iOS中也是一个非常容易出问题的地方，crash率会比较高。究其原因，我想是主要是因为定时器存在一个事件发生的延后性（废话嘛0_o）,但是很多时候会忘记，当定时任务真的发生的时候，语境变化了吗？如果语境都已经被dealloc了，定时任务仍然被激活，系统就会愤怒地罢工了。

比如：

```
componentDidMount() {
    setTimeout(
      () => { console.log('这就可能会崩溃'); },
      500
    );
  }
```

如果500ms之内，这个component就会Unmount了，那直接回崩溃。RN官方的建议如下：

#### TimerMixin
为了解决这个问题，我们引入了TimerMixin。如果你在组件中引入TimerMixin，就可以把你原本的setTimeout(fn, 500)改为this.setTimeout(fn, 500)(只需要在前面加上this.)，然后当你的组件卸载时，所有的计时器事件也会被正确的清除。

这个库并没有跟着React Native一起发布。你需要在项目文件夹下输入npm i react-timer-mixin --save来单独安装它。

```
var TimerMixin = require('react-timer-mixin');

var Component = React.createClass({
  mixins: [TimerMixin],
  componentDidMount: function() {
    this.setTimeout(
      () => { console.log('这样我就不会导致内存泄露!'); },
      500
    );
  }
});
```

#### 代码保护（推荐）
Mixin属于ES5语法，对于ES6代码来说，无法直接使用Mixin。如果你的项目是用ES6代码编写，同时又使用了计时器，那么你只需铭记在unmount组件时清除（clearTimeout/clearInterval）所有用到的定时器，那么也可以实现和TimerMixin同样的效果。例如：

```
import React,{
  Component
} from 'react';

export default class Hello extends Component {
  componentDidMount() {
    this.timer = setTimeout(
      () => { console.log('把一个定时器的引用挂在this上'); },
      500
    );
  }
  componentWillUnmount() {
    // 如果存在this.timer，则使用clearTimeout清空。
    // 如果你使用多个timer，那么用多个变量，或者用个数组来保存引用，然后逐个clear
    this.timer && clearTimeout(this.timer);
  }
};
```

我今天看了我们项目的代码，发现几乎没有人做保护，代码copy的现象，真的是令人发指，可能很多人都没仔细看过官方的文档。。。





