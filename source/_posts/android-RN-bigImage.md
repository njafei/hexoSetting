---
title: Android上的RN不能展示尺寸大的图，怎么办？
date: 2017-05-18 17:44:01
tags: [React-Native,Android]
categories: React-Native
---


# 背景
最近遇到一个营销的需求，一个简单的界面，上面是一个计数器，下面是一张介绍图。所以下面直接用了`Image`标签来做。但是在测试中发现了一个问题，部分的`Android`手机上面展示不出来图片。使用`Android studio`检查了下log，发现了如下的错误：

> OpenGLRenderer: Bitmap too large to be uploaded into a texture (750x4520, max=4096x4096

这个因为图片的尺寸过大（注意，不是大小哦），导致`Android`手机不能渲染，那么怎么解决这个问题呢？

# 解决
这里推荐使用`webView`，RN提供的`webView`功能还算强大，可以把它作为一个容易来装图片，最重要的是，大部分的`Android`手机对`webView`的优化还是非常不错的。而且`webView`还支持多种格式，你甚至可以把`html`代码写入一个字符串中传给`webView`。具体请参考wiki:[http://reactnative.cn/docs/0.43/webview.html#content](http://reactnative.cn/docs/0.43/webview.html#content)


实现代码如下：

```
let leftNum = this.getLeftNumberText();
        const HTML = `
            <!DOCTYPE html>\n
            <html>
              <head>
                <title>Hello Static World</title>
                <meta http-equiv="content-type" content="text/html; charset=utf-8">
                <meta name="viewport" content="width=320, user-scalable=no">
                <style type="text/css">
                    body{
                        margin:0;
                        padding:0;
                        
                    }
                    img{
                        width:100%;
                    }
                </style>
              </head>
              <body>
                <img src="https://m.tuniucdn.com/fb2/t1/G1/M00/F1/51/Cii9EFkAaZ-IRgGNAATB18ldk0UAAJzuQN-p1cABMHv15.jpeg"/>
              </body>
            </html>
            `;

        let image =  (
            <Image style={styles.image}
                   source={{uri: 'https://m.tuniucdn.com/fb2/t1/G1/M00/F1/51/Cii9EFkAaZ-IRgGNAATB18ldk0UAAJzuQN-p1cABMHv15.jpeg'}}/>);

        let webView = (
            <WebView
                style={styles.image}
                automaticallyAdjustContentInsets={false}
                source={{html:HTML}}
                javaScriptEnabled={true}
                domStorageEnabled={true}
                decelerationRate="normal"
                startInLoadingState={true}
                scalesPageToFit={false}
                onLoadEnd = {()=>{console.log('loading end')}}
                onError = {()=>{console.log('loading error')}}
            />
            )

        let ImageShow = Platform.OS==='android' ? webView: image;
        return (
            <View style={styles.bgView}>
                <Header title = '牛大头'></Header>
                <ScrollView style={styles.scrollView}>
                    <View style={styles.tipBgView}>
                        <Image source = {require('./images/icon-ask-home-tip.png')}
                        style={styles.tipIcon}/>
                        <Text style={styles.tipText}>{leftNum}</Text>
                    </View>
                    {ImageShow}
                </ScrollView>
            </View>
        )
```

# 总结

其实我觉得， 如果是营销界面，最好是公司提供一整套可以配置的H5界面，这样可以直接使用后台的`CMS`后台去配置出来一个界面，节省人力，效率高。

如果一定要`Native`或者`RN`来做的话，像我们这次`H5`前端人力比较紧张，那就可以`Native`的开发上。直接写H5界面放到`webView`中，还是比较方便的。当然如果是比较复杂的，还是要`Native`实现的，这里仅提供一种思路。

前端大融合，指日可待啊。