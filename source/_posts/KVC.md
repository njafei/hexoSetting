---
title: KVC详解
date: 2017-04-18 17:58:01
tags:
    - 基础知识
    - Apple文档
categories: iOS基础
---


# 综述

### 关于
正常访问或者修改一个对象的属性，都是通过getter和setter方法，但是Cocoa仍然提供了一个间接访问属性的方法：KVC（Key-Value Coding）。 只要对象支持`NSKeyValueCoding`协议，我们就可以通过KVC来间接访问或者修改属性和属性中的更深层的属性。

KVC也是许多Cocoa技术的基础，比如：

- [KVO](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)
- [Cocoa bingdings](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CocoaBindings/Concepts/WhatAreBindings.html)
- [Core Data](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075)
- [AppleScript](https://developer.apple.com/library/content/documentation/AppleScript/Conceptual/AppleScriptX/AppleScriptX.html#//apple_ref/doc/uid/10000156i)。

### 使用KVC的对象

所有继承`NSObject`的类都支持KVC，`NSObject`实现了`NSKeyValueCoding `和必要的方法。通过KVC，可以做到以下功能：

- 获取对象属性 比如使用 `valueForKey` 和 `setValue:forkey:`来获取和修改属性
- 操作Collection类型的的属性，比如`NSArray`、`NSSet`
- 使用Collection运算符
- 非对象的values
- 路径搜索的步骤（本文不会讲这个，请自己看文档）

下面，我们来挨个看下上面的5种用法

# 获取对象属性

属性可以分为3个类型：
- Attributes（简单属性），比如string，int，bool等简单类型
- To-one relationships（单一关系），比如一个`Person`类的实例
- To-many relationships（多个关系），比如`NSArray`或者`NSSet`

来看下例子,这个一家人的类： 


```
@interface Person : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;

@end


@interface Family : NSObject

@property (nonatomic) NSNumber* numbers;              // An attribute
@property (nonatomic) Person* boss;                         // A to-one relation
@property (nonatomic) NSArray< Person* >* members; // A to-many relation

@end

```

### 通过key与keyPath读取属性
在介绍如何读取之前，我简单说下key和keyPath。
key可以简答理解为对象某个属性的名称，而keyPath是又`.`区分的一串string，用来读取更深层的value。举个列子：对于Family来说，他的key有几个：`numbers`、`boss`、`members`。我们可以通过这几个key来读取他对应的属性。但是如果我们想要读取`boss`的name，一种办法是先读取`boss`，然后读取`boss`的name，另一个办法就是我们通过keyPath,直接读取`family`的`boss.name`，这个`boss.name`就是keyPath。

通过key和keyPath读取属性，有以下几个方法：

- `valueForKey:` 读取key
- `valueForKeyPath:` 读取keyPath 
- `dictionaryWithValuesForKeys:` 批量读取


还是通过例子来看，我们先初始化几个实例出来：

```
Person *father = [Person new];
father.name = @"Jack";
father.age = 50;

Person *mother = [Person new];
mother.name = @"rose";
mother.age = 45;


Family *family = [Family new];
family.numbers = @(2);
family.boss = father;
family.members = @[father, mother];
```

我们分别用上面的三个方法来读取属性：

```
NSString *fatherName = [father valueForKey:@"name"];
NSLog(@"father name: %@,",fatherName);

NSString *bossName = [family valueForKeyPath:@"boss.name"];
NSLog(@"boss name: %@,",bossName);

NSDictionary *names = [family dictionaryWithValuesForKeys:@[@"numbers",@"members"]];
```

> 注意，如果是NSArray、NSSet等类型，不能包含nil，而是转皇城NSNull。系统会在使用的时候自动转换。

### 通过key和keyPath修改属性

有以下几种方法：

- setValue:forkey:
- setValue:forKeyPath:
- setValuesForKeysWithDictionary:

还是直接上例子：

```
[father setValue:@"Jack is gone" forKey:@"name"];

[family setValue:@"boss's name" forKeyPath:@"boss.name"];

[family setValuesForKeysWithDictionary:@{@"numbers":@(3),@"boss":father}];
```

### 使用KVC简化你的代码

讲了半天KVC的使用，那么我们什么时候用？怎么用比较适合呢？看个例子:

如果你有类column，identifier可能是name、age、favoriteColor，你要根据identifier的不同去展示不同的属性。正常写法：

```
- (id)tableView:(NSTableView *)tableview objectValueForTableColumn:(id)column row:(NSInteger)row
{
id result = nil;
Person *person = [self.people objectAtIndex:row];

if ([[column identifier] isEqualToString:@"name"]) {
result = [person name];
} else if ([[column identifier] isEqualToString:@"age"]) {
result = @([person age]);  // Wrap age, a scalar, as an NSNumber
} else if ([[column identifier] isEqualToString:@"favoriteColor"]) {
result = [person favoriteColor];
} // And so on...

return result;
}
```

简化写法：

```
- (id)tableView:(NSTableView *)tableview objectValueForTableColumn:(id)column row:(NSInteger)row
{
return [[self.people objectAtIndex:row] valueForKey:[column identifier]];
}
```


# 获取Collection对象属性

在上面我们看到，其实使用`valueForKey:`和`valueForKeyPath`是可以获取`NSArray`等类型的，但是我们获取的是一个不可变的类型，如果我们希望去修改`key`或者`keyPath`对应的数组的时候，怎么做呢？

使用：

- mutableArrayValueForKey: 和 mutableArrayValueForKeyPath:
- mutableSetValueForKey: and mutableSetValueForKeyPath:
- mutableOrderedSetValueForKey: and mutableOrderedSetValueForKeyPath:

来个例子：

```
NSMutableArray *members = [family mutableArrayValueForKey:@"members"];

Person *son = [Person new];
father.name = @"Jack's son";
father.age = 15;

[members addObject: son];
```

# 使用Collection运算符

### 基本组成
我们在实际的使用时，会有很多类似于数组平均数、最大值等计算的需求，除了自己写算法计算之外，如果有类似于数据库的快捷操作符，那该多好啊！

现在，我们就来看下KVC提供的数组等的快捷计算符。

正常使用`keyPath`的时候，我们通常会使用类似`person.son.name`等来读取属性。`KVC`同样提供了`@`操作符和一些基本的操作，我们可以放到`keyPath`中，就可以在return之前执行这个操作，然后再返回值了，听起来有点变扭，我们先看下结构：

![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/art/keypath.jpg)

操作符是由 `@`符号和操作函数名组成的

在操作符前面的都叫 `left key path `，这里指定接收消息的对象，为空的话，就是执行`keyPath`的对象

在操作符后面的都叫 `right key path`, 这里执行操作的对象，除了数组的`@count`之外，不能为空

### 聚合运算
这样讲，还是有点虚，我们看个例子：

```
@interface SaveRecond : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger money;

@end


@interface Bank : NSObject

@property (nonatomic) NSArray< SaveRecond* >* reconds;

@end
```

初始化下数据：

```
SaveRecond *save1 = [SaveRecond new];
save1.name = @"Jack";
save1.money = 10;

SaveRecond *save2 = [SaveRecond new];
save2.name = @"rose";
save2.money = 20;

SaveRecond *save3 = [SaveRecond new];
save3.name = @"Jack";
save3.money = 30;


Bank *bank = [Bank new];
bank.reconds = @[save1, save2,save3];
```

如果我们想计算这些储户的平均值，可以直接遍历bank的reconds，然后计算，那么，如果用KVC的operation，该如何做呢？一句话解决：

`NSObject *count = [bank valueForKeyPath:@"reconds.@count"];`

这句话的意思就是 我希望向bank的reconds属性，发送@count消息，执行@count运算。

我们再来看个有`right key path`的。这里我希望计算出所有记录的平均储蓄值。

`NSObject *avg = [bank valueForKeyPath:@"reconds.@avg.money"];`
这句话的意思是，我希望向bank的reconds属性，发送@avg消息，执行money的avg运算。

同理，计算符还有很多，我这里就不一一介绍了，简单列下，只要学过数据库的，应该没啥问题（没学过？感觉学习下呀0_0）

- @avg
- @count
- @max
- @min
- @sum

### 数组运算
刚才用的都是聚合之后的计算，我们看下如何对数组进行计算。比如：我想要知道一共有个用户(去重)。

`    NSArray *array = [bank valueForKeyPath:@"reconds.@distinctUnionOfObjects.name"];`

这句话的含义是 我希望向bank的reconds属性发送distinctUnionOfObjects消息，执行基于name的distinctUnionOfObjects操作。

这里的运算符有两个：

- distinctUnionOfObjects 去重
- unionOfObjects 不去重

### 嵌套运算操作

如果是想操作一个数组的数组(@[array,array,array])，要怎么样做呢？比如我有两个reconds数组，我想看一共有多少个用户(去重)。

```
NSArray *arrayOfArray = @[bank.reconds,bank.reconds];
NSArray *array = [arrayOfArray valueForKeyPath:@"@distinctUnionOfArrays.name"];
```

这里的运算符有三个：

- @distinctUnionOfArrays
- @unionOfArrays
- distinctUnionOfSets

### 非对象的values

当我们获取的是一个非对象的value，比如int、Bool等值，Cocoa会自动转换成NSNumber等对象
包括：
bool、char、double、float、int、long等等，这个自己去查文档吧

如果value是一个结构体该如何呢？ 比如NSPoint、NSRange、NSRect、NSSize等还是会返回本类，但是其他的结构体，会被返回一个NSValue

# 检查属性
当我要使用`setValueForKey`等方法去修改值时，怎么知道是不是合法呢？
KVC也提供了方法
- validateValue:forKey:error
- validateValue:forKeyPath:error

这个方法返回一个bool值，有三种情况：
1. value合法，return YES
2. value不合法，但是可以给value重新赋值了一个新的object，return YES
3. value不合法，且不可以赋值挽救，return NO

最坑人的是，这个方法要自己实现，否则默认返回YES。Are you kidding？ 那我干嘛要用你的这个方法。。。


