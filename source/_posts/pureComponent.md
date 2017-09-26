---
title: React-Native优化之PureComponent
date: 2017-07-27 18:56:18
tags: [react-native,component]
categories: [react-native]

---

React15.3的发布中包含了PureComponent，这个类最重要的用法是为了优化React的性能，下面我们将看下它是如何优化的。

# Component VS PureComponent
首先要看Component的生命周期：
![](http://on0hv7n2x.bkt.clouddn.com/component-lifecycle.jpg)

当props或者state改变的时候，会执行`shouldComponentUpdate`方法来判断是否需要重新render组建，我们平时在做页面的性能优化的时候，往往也是通过这一步来判断的。Component默认的`shouldComponentUpdate`返回的是true，如下：

```
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

而PureComponent的`shouldComponentUpdate`是这样的：

```
if (this._compositeType === CompositeTypes.PureClass) {
  shouldUpdate = !shallowEqual(prevProps, nextProps) || ! shallowEqual(inst.state, nextState);
}
```

这里的比较，只会做潜比较，即比较两者的内存地址是否相同，而对于其值是否发生变化，则不会理会。我们通过以下的例子来看下：

# 例子

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { PureComponent,Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  Button
} from 'react-native';

export default class test extends PureComponent {
  constructor(props){
    super(props);
    this.state = {
       number : 1,
       numbers: [],
    };
  }

  render() {
    return (
      <View style={styles.container}>
        <Button title={'number + 1'} onPress={this.numberAdd.bind(this)} />
        <Text>number value: {this.state.number}</Text>
        <Button title={'numbers + 1'} onPress={this.numbersAdd.bind(this)} />
        <Text>numbers length: {this.state.numbers.length}</Text>
      </View>
    );
  }

  numberAdd(){
      this.setState({number: ++this.state.number });
  }


  numbersAdd(){
    let numbers = this.state.numbers;
    numbers.push(1);
    this.setState({numbers: numbers});
    console.log(this.state.numbers);
  }


}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

AppRegistry.registerComponent('test', () => test);
```

界面如下：

![](http://on0hv7n2x.bkt.clouddn.com/screenShotSimulator%20Screen%20Shot%202017%E5%B9%B47%E6%9C%8827%E6%97%A5%20%E4%B8%8B%E5%8D%886.48.31.png)

这里去点击number+1 和 numbers+1都不会有任何页面的变化。

# 如何让PureComponent重绘
那如果PureComponent变化的时候(这其实不符合我们的初衷)，我们要怎么做呢？这里有两个办法：

 1. 重写shouldUpdateComponent方法
 2. props或者state增减参数

代码如下：

```
numbersAdd(){
    let numbers = this.state.numbers;
    numbers.push(1);
    this.setState({numbers: numbers});
    console.log(this.state.numbers);

    this.setState({newState: 1});
  }
```

这样，shouldComponentUpdate的返回值也会是true。

# 总结

综上，PureComponent非常适合于不变的组件，尤其是和数据、业务无关的纯展示组件，因为它的节省了大量比较的工作。但是对于大部分的业务来说，界面很少会有不变的组件，所以使用的场景会比较少，但是如果遇到，请尽情使用！

