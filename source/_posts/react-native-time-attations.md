---
title: react-native time定时器 防止内存泄露的注意点
date: 2017-07-27 15:50:21
tags: [注意事项, react-native]
categories: [react-native]
---



`time`是`react-native`提供的一个定时器，在实际使用中，经常会有使用不对，造成内存泄露的情况。很多`React Native`应用发生致命错误（闪退）是与计时器有关。具体来说，是在某个组件被卸载`（unmount）`之后，计时器却仍然在运行。

防止出问题的办法也很简单，在`unmount`的时候，增加卸载定时器的操作：

```
componentDidMount() {
    this.timer = setTimeout(
      () => { console.log('把一个定时器的引用挂在this上'); },
      500
    );
  }
  
componentWillUnmount() {
    // 请注意Un"m"ount的m是小写
    // 如果存在this.timer，则使用clearTimeout清空。
    // 如果你使用多个timer，那么用多个变量，或者用个数组来保存引用，然后逐个clear
    this.timer && clearTimeout(this.timer);
  }
```

这里的timer是在`DidMount`中赋值的，如果是多次赋值呢？比如：

```
export default class test extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Button title={'click5000'} onPress={this.alertInfo.bind(this,5000)} />
        <Button title={'click2000'} onPress={this.alertInfo.bind(this,2000)} />
        <Button title={'clean timer'} onPress={this.cleanTimer.bind(this)} />
      </View>
    );
  }

  alertInfo(time){
      this.timer = setTimeout(
          ()=>{
            alert('hah');
          },
          time,
      );
  }

  cleanTimer(){
    this.timer && clearTimeout(this.timer);
    console.log('timer cleared')
  }
}


AppRegistry.registerComponent('test', () => test);
```

这里要介绍下`setTimeout`的返回值，我们打断点可以看到，`this.timer`是一个number。根据stack上面的其他网友的回答[what-does-settimeout-return](https://stackoverflow.com/questions/10068981/what-does-settimeout-return)，setTimer会返回一个id，代表你已经向js的runtime系统中成功注册了一个定时器任务，这个id就是系统返回的id。

那如果是需要多次赋值，就一定要先将time clear掉，然后再赋值，或者使用多个参数来标志，否则之后就找不到上次的id，也就没办法clear了，同样可能造成内存泄露的情况。

例子如下：

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  Button
} from 'react-native';

export default class test extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Button title={'click5000'} onPress={this.alertInfo.bind(this,5000)} />
        <Button title={'click2000'} onPress={this.alertInfo.bind(this,2000)} />
        <Button title={'clean timer'} onPress={this.cleanTimer.bind(this)} />
      </View>
    );
  }

  alertInfo(time){
      this.timer = setTimeout(
          ()=>{
            alert('hah');
          },
          time,
      );
  }

  cleanTimer(){
    this.timer && clearTimeout(this.timer);
    console.log('timer cleared')
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

这里的timer多次赋值，虽然前面的值被后面的覆盖，但是前面的time仍然会起效果，如果不想前面的time work，需要clear掉，然后再赋值。

代码如下：

```
alertInfo(time){
		this.timer && clearTimeout(this.timer);
      this.timer = setTimeout(
          ()=>{
            alert('hah');
          },
          time,
      );
  }
```


综上，使用time的注意点：
 
 1. 记得unmount的时候，clear
 2. 多个timer要使用多个变量或者数组
 3. 多次赋值，记得把之前的值clear掉
 