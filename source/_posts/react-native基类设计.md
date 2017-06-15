---
title: react-native基类设计
date: 2017-05-20 11:16:04
tags: [React-Native]
categories: React-Native

---

# 背景

前段时间发现iOS手机上，很多页面的字体样式会随着系统配置字体的大小而变换，使得界面不太美观。而这个其实只要给一个参数就可以解决，但是整个app中用该组件的地方非常多，开发挨个替换的话，工作量很大，而且很容易出现遗漏。

另外升级RN后，出现了`Android`手机上面的`Text`点击崩溃问题，底层的bug，前端需求的话，也是要花费相当精力而且容易遗漏。

那么，对于类似的问题，有没有好的解决方案呢？


# 解决思路

如果抽象以下，我们其实是要解决两个痛点：
	
- 风险控制问题
- 提高开发效率

#### 风险控制问题

这个是要解决`RN`框架界面突然爆发问题或者bug，我们应该是能及时、快速修复掉。目前其实`RN`还是可以热修复的，但是面对上面两个问题，其实没办法热修复，因为涉及到的点太多，工作量等太大。

所以，要解决风险控制问题，保证底层对于业务的组件是有控制力的，从而避免突然爆发问题而束手无策。

#### 提高开发效率

这里还要介绍下在`Image`这个组件遇到的问题。一个是当前所有的图片都是没有默认的占位图的，另外就是如果图片下载失败，`Image`就会是空白，只有做特殊处理才能显示占位图。如果每个业务开发在使用`Image`的时候，都要加几行这种类似的重复代码，效率是非常低的。

或者临时遇到诸如整体换颜色、字体等问题，都是非常痛苦的。

所以提出想要能把开发从这种重复低价值劳动中解放出来。


# 方案

基类的方案，有两个核心点：

 - 所有的页面都继承自基类
 - 所有的组件都不再从`react-native`库中取，而是从基类中取

如果一句话来概括的话，就是我们封装了一层`react-native`。业务开发使用的组件都是从我们自己库中取，而库中的控件，可以是定制的，也可以是简单从`react-native`中调用。


# 包含控件

这里是基类`Component`和其他几个开发过程中遇到的问题比较多的：

 - Component
 - Text
 - Image


具体的定制需求如下（还需要向开发搜集）：

#### Text

 - 默认字体等配置 
 - 是否自动根据系统字体设定大小 false
 - 解决number等的崩溃问题
 
#### Image

 - 加载失败的处理
 - 默认图片 （牛头图）
 - 自动更换webp的操作


# 好处
好处就是解决上文提的两个痛点：

 - 风险控制问题
 - 提高开发效率

# 缺点

基类如果出现bug，影响范围会比较大。

# 待确定的问题

 - 具体定制组件的需求还要搜集下

# 代码实现

#### Base Component

```
import React from 'react';

import {
    Button,
    DatePickerIOS,
    DrawerLayoutAndroid,
    FlatList,
    Image,
    KeyboardAvoidingView,
    ListView,
    Modal,
    Navigator,
    NavigatorIOS,
    Picker,
    PickerIOS,
    ProgressBarAndroid,
    ProgressViewIOS,
    RefreshControl,
    ScrollView,
    SectionList,
    SegmentedControlIOS,
    Slider,
    StatusBar,
    Switch,
    TabBarIOS,
    TextInput,
    ToolbarAndroid,
    TouchableHighlight,
    TouchableNativeFeedback,
    TouchableWithoutFeedback,
    View,
    ViewPagerAndroid,
    VirtualizedList,
    WebView,
    StyleSheet,
    Platform,
    AsyncStorage,
    TouchableOpacity,
    DeviceEventEmitter,
    ActivityIndicator,
    NativeModules,
    findNodeHandle,
    ActionSheetIOS,
    AdSupportIOS,
    Alert,
    AlertIOS,
    Animated,
    AppRegistry,
    AppState,
    BackAndroid,
    CameraRoll,
    Clipboard,
    DatePickerAndroid,
    Dimensions,
    Easing,
    Geolocation,
    ImageEditor,
    ImagePickerIOS,
    ImageStore,
    InteractionManager,
    Keyboard,
    LayoutAnimation,
    Linking,
    NativeMethodsMixin,
    NetInfo,
    PanResponder,
    PermissionsAndroid,
    PixelRatio,
    PushNotificationIOS,
    Share,
    Systrace,
    TimePickerAndroid,
    ToastAndroid,
    Vibration,
} from 'react-native';

import Text from './BaseText.js'

class BaseComponent extends React.Component {

    render(){
        return (
            <View>

            </View>
        )
    }
}

export default  BaseComponent;


module.exports = {
    //自定义的组件
    BaseComponent,
    Text,
    //使用系统的组件
    Button,
    DatePickerIOS,
    DrawerLayoutAndroid,
    FlatList,
    Image,
    KeyboardAvoidingView,
    ListView,
    Modal,
    Navigator,
    NavigatorIOS,
    Picker,
    PickerIOS,
    ProgressBarAndroid,
    ProgressViewIOS,
    RefreshControl,
    ScrollView,
    SectionList,
    SegmentedControlIOS,
    Slider,
    StatusBar,
    Switch,
    TabBarIOS,
    TextInput,
    ToolbarAndroid,
    TouchableHighlight,
    TouchableNativeFeedback,
    TouchableWithoutFeedback,
    View,
    ViewPagerAndroid,
    VirtualizedList,
    WebView,
    Platform,
    AsyncStorage,
    TouchableOpacity,
    DeviceEventEmitter,
    ActivityIndicator,
    NativeModules,
    findNodeHandle,
    ActionSheetIOS,
    AdSupportIOS,
    Alert,
    AlertIOS,
    Animated,
    AppRegistry,
    AppState,
    BackAndroid,
    CameraRoll,
    Clipboard,
    DatePickerAndroid,
    Dimensions,
    Easing,
    Geolocation,
    ImageEditor,
    ImagePickerIOS,
    ImageStore,
    InteractionManager,
    Keyboard,
    LayoutAnimation,
    Linking,
    NativeMethodsMixin,
    NetInfo,
    PanResponder,
    PermissionsAndroid,
    PixelRatio,
    PushNotificationIOS,
    Share,
    StyleSheet,
    Systrace,
    TimePickerAndroid,
    ToastAndroid,
    Vibration,
}
```

#### 定制Text：

```
import React from 'react';
import {
    Text,
    StyleSheet,
} from 'react-native';



class BaseText extends React.Component{
    constructor(props){
        super(props);
    }

    render(){

    let {style,...others} =this.props;

    return (
            <Text {...others}
                  allowFontScaling={false}
                  style={[styles.defaltStyle,style]}
            />
        )
    }
}

const styles = StyleSheet.create({
    fontSize: 12,
})

export default  BaseText;

```

#### 子类使用:

```
import {
    Text,
    BaseComponent,
} from '../../../../../common/BaseComponent'

render() {

        return (
            <Text>这是Text的内容</Text>
        )
}
```


# 计划

#### 第一期

首先可以先把紧迫的诸如`Text`、`Image`等待解决的bug和样式等做好，基类写好，替换诸如某个模块，如果上线之后没有问题，可以替换所有的RN页面。

#### 第二期

 - 将默认样式和诸如快捷写法等集成到基类中，提高大家书写RN代码的效率，也方便以后更换等。
 - 考虑一些AOP的使用，比如打点，统计等。