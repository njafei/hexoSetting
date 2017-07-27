---
title: listView和FlatList的flexGrow默认值为1
date: 2017-07-26 19:27:06
tags: [React-Native,奇怪的bug]
categories: [React-Native]
---


今天遇到了适配的问题，有个列表，需要自适应高度，按理说默认应该就是自适应的，但是在实际中发现，其会和另外一个视图1：1 ，然后就发现只有设置其`flexGrow: 0`的时候，它才会自动适配高度，说明它的flexGrow默认值为1.

看下具体的列子：

```
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  ListView,
} from 'react-native';

export default class testListView extends Component {
    constructor(props) {
        super(props);
        const ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
        this.state = {
            dataSource: ds.cloneWithRows([
                'John', 'Joel', 'James', 'Jimmy', 'Jackson', 'Jillian', 'Julie', 'Devin'
            ])
        };
    }

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.blackView} />
        <ListView
            dataSource={this.state.dataSource}
            renderRow={(rowData) => <Text>{rowData}</Text>}
            style={styles.list}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flexGrow: 1,
    backgroundColor: '#F5FCFF',
  },
  list:{
    backgroundColor: 'red',
  },
  blackView: {
    flexGrow: 1,
    backgroundColor: 'black',
  }
});

AppRegistry.registerComponent('testListView', () => testListView);

```

这个UI看起来是这样： 

![](http://on0hv7n2x.bkt.clouddn.com/Simulator%20Screen%20Shot%202017%E5%B9%B47%E6%9C%8826%E6%97%A5%20%E4%B8%8B%E5%8D%887.20.31.png)

如果style中的list改成这样，就好了：

```
list:{
    backgroundColor: 'red',
    flexGrow: 0,  
},

```

显示成： 


![](http://on0hv7n2x.bkt.clouddn.com/Simulator%20Screen%20Shot%202017%E5%B9%B47%E6%9C%8826%E6%97%A5%20%E4%B8%8B%E5%8D%887.21.58.png)

综上，我怀疑`FlatList和ListView`的`flexGrow`默认值是1.有遇到类似问题的，不妨试试这样解决。

版本： "react-native": "0.43.4"
