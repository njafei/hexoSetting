---
title: React_Native拆分bundle之patch拆包
date: 2017-04-06 10:26:41
tags: React-Native
categories: React-Native
---

# 为什么要拆包
## 背景介绍
随着RN的包越来越大，第一次载入RN包的时长越来越长，用户需要等待的时间也就越长，体验较差。另外多个团队开发的话，互相之间的依赖也是个大问题，出现编译不过的话，就会出现水桶效应，所有的团队都要等待这个有问题的团队，从而拉低了整体的效率。

另外我一直希望，能够将React-Native的业务功能，做成类似小程序一样：即用即载入，随时可以更新。想想我们的app里面包含了多少个用户也许永远用不到的功能，还有当我们希望上一个新功能的时候，一定要等待新的版本的审核，这给运营等带来了巨大的麻烦和风险。如果用户点击某个功能，然后马上载入线上的webBundle，用户之后就可以直接使用我们的最新功能了，以后再次进入的时候，也无需等待，那该多好。

## 拆包目标
所以我们拆包的目标就很明确了：

	1. 优化载入时间，提高用户体验
	2. 解开依赖关系，提高开发效率
	3. 实现webBundle，即用即载入

示意图如下：
![](http://on0hv7n2x.bkt.clouddn.com/React-Native%E6%8B%86%E5%8C%85%E6%96%B9%E6%A1%88%E7%BB%93%E6%9E%84%E5%9B%BE.jpeg)


## 国外国内app拆包情况
上面啰嗦了为什么想要去拆包，好像是蛮有必要的O_o。但是当我去看国内外著名的app使用React-Native的情况时，发现真的是泾渭分明：国内基本都拆包了，包括携程、QQ音乐等，而国外没有拆包的，比如React-Native的创造者FaceBook，虽然他们的包大小已经到了10M。

不禁让我很疑惑，难道国外没有这个需求么？为什么拆包和热更新等几乎国内的硬需求，他们却好像完全没有这方面的需求。希望有读者知道的话可以告知我~


# 拆包的几种方案
在讲具体的方案之前，我们先看下，React-Native的包，究竟是如何打出来，然后是怎么载入到native中的。

## 如何打包
这里我直接使用QQ技术团队的一张图：
![](http://on0hv7n2x.bkt.clouddn.com/React-NativeJsBundle%E6%89%93%E5%8C%85%E6%B5%81%E7%A8%8B)

## 如何载入
这里主要讲下iOS React-Native0.39版本的情况。
RN提供了两种形式来载入：

```
1. - (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
2. - (void)loadSourceForBridge:(RCTBridge *)bridge
                 onProgress:(RCTSourceLoadProgressBlock)onProgress
                 onComplete:(RCTSourceLoadBlock)loadCallback;

```
第一种数据是默认模式，第二种是可以控制载入中的各个步骤。这块可以看下RCTBridgeDelegate。

## 主流方案
在网上查了相关的资料，主流的方案基本都是把Main.jsbundl拆分成基础包common.jsbundle+业务包bundle，和上面拆包目标基本一样，不赘述。

具体的拆分思路就很不一样了：

	1. 侵入RN代码，修改打包流程，使得打出来的包就是基础+业务包，如QQ音乐
	2. 在RN打包的基础上，实现新的打包方案，如携程 moles-Packer
	3. Patch方案，打包流程不变，生成基础包后，根据diff来生成每个业务不同的patch包

# patch方案
因为方案1和方案要随着RN的升级，不断调整，成本比较高，而且要投入较多的人力，所以我们先看下方案3。

先说下patch，patch就是根据特定算法，讲两个不同的事物diff比较，然后生成的包含两个事物差别的包。我们这里使用的是google的`diffAndPatch`算法。

![](http://on0hv7n2x.bkt.clouddn.com/React-Native%E7%94%9F%E6%88%90%E5%8C%85.png)

## 基础包common.jsbundle
首先我们先生成基础包common.jsbundle.这里我们写一个空的js文件，只包含react-native头文件common.ios.js：

```
import React from 'react'; 
import {} from 'react-native'; 

```
然后我们基于这单个文件打包，打出来的包就是只包含react-native基础框架的bundle，我们成为common.jsbundle.

注：

RN打包过程中会做混淆，所有的类最终都变成了代号为数字的function，所以这里顺序就非常重要，而对基础包的引用顺序就要严格和common.ios.js一样了，这里建议所有的业务代码直接引用common.ios.js文件。另外如果有公共组件等，也都可以放到common.ios.js文件中，这样就会被包含在基础包中了。

## 业务包 business.patch

每条业务线的代码，都需要单独维护自己的indexBisiness.js,打包的时候，入口文件就是这个index，这样就打出来了一个业务business.bundle。
然后使用diff，计算出业务patch。这样就算出了patch，我们叫做business.patch

## native方案

现在我们已经有了common.jsbundle + bisiness1.patch + business2.patch + ...
如果打开了bisiness1中的home.js，我们首先要将common.jsbundle和bisiness1.patch使用算法合并，计算出最终bisiness1.jsbundle.然后通过上面讲到的native载入方案载入具体的bundle。

根据当前的bridgeName生成bisiness bundle

```
- (NSString *)getNewBundle
{
  
  NSString *commonBundlePath = [[NSBundle mainBundle] pathForResource:@"common" ofType:@"jsbundle"];
  NSString *commonBundleJSCode = [[NSString alloc] initWithContentsOfFile:commonBundlePath encoding:NSUTF8StringEncoding error:nil];
  
  NSString *patch1Path = [[NSBundle mainBundle] pathForResource:self.bridgeName ofType:@"patch"];
  NSString *patch1JSCode = [[NSString alloc] initWithContentsOfFile:patch1Path encoding:NSUTF8StringEncoding error:nil];
  
  
  DiffMatchPatch *diffMatchPatch = [[DiffMatchPatch alloc] init];
  NSArray *convertedPatches = [diffMatchPatch patch_fromText:patch1JSCode error:nil];
  
  NSArray *resultsArray = [diffMatchPatch patch_apply:convertedPatches toString:commonBundleJSCode];
  NSString *resultJSCode = resultsArray[0]; //patch合并后的js
  
  
  NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
  NSString *docDir = [paths objectAtIndex:0];
  NSString *newPath = [NSString stringWithFormat:@"%@/%@.jsbundle",docDir,self.bridgeName];
  
  if (resultsArray.count > 1) {
    [resultJSCode writeToFile:newPath atomically:NO encoding:NSUTF8StringEncoding error:nil];
    return newPath;
  }
  else {
    return @"";
  }
  
}

```

加载bisiness bundle：

```
- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
    NSString *path = [[NSBundle mainBundle] pathForResource:self.bridgeName ofType:@"jsbundle"];
    NSURL *jsBundleURL = [NSURL URLWithString:path];
  
  return jsBundleURL;
}
```

这样，code就基本完成了。如果想看所有的代码，请看底部的githubDemo


## 方案优缺点

优点：

	1. 技术方案简单，实现快
	2. 稳定、不用担心RN升级问题
	3. 业务互相独立
	4. 方便后面做web bundle

缺点：

	1. 内存占用大
	2. 打包会变大
	3. 业务之间资源和代码没法互相引用


## 优化和拓展计划

	1. 打包可以不用patch的方案，采用脚本逐行写入
	2.web bundle 可以直接基于这个方案做
	3.如果解决了函数命名和依赖的问题，就可以采用一个bundle策略

## Demo in github
[ReactNativeSeperateBundle](https://github.com/njafei/ReactNativeSeperateBundle
)


 1. cd ReactNativeSeperateBundle
 2. npm install
 3. run project
