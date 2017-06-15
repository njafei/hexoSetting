---
title: 如何访问和修改一个对象的私有属性
date: 2017-06-15 18:03:36
tags: [iOS面试题]
categories: [iOS]

---


有两个思路：

 - KVC
 - runtime

先给出我们接下来要使用的类：

```
@interface Person : NSObject

@end


@interface Person()

@property (nonatomic, copy) NSString *name;

@end

@implementation Person

@end


```

# KVC
KVC是我比较推荐的，代码如下：

```
Person *person = [Person new];
    
[person setValue:@"new name" forKey:@"name"];

NSString *name = [person valueForKey:@"name"];

```

KVC是苹果推荐用来做类似事情的方法，所以这种需求，KVC解决是最好的，代码简洁，效率也比较高。
关于KVC的相关内容可以看下[KVC详解](https://njafei.github.io/2017/04/18/KVC/)

# runtime

runtime的思路就是先读取对象的所有属性，然后找到对象的属性，赋值。代码如下：

```
Person *person = [Person new];

unsigned int count = 0; //count记录变量的数量

Ivar *members = class_copyIvarList([person class], &count);
for (int i = 0; i < count; i++) {
    Ivar ivar = members[i];
    const char *memberName = ivar_getName(ivar);
    NSString *memberNameString = [NSString stringWithFormat:@"%s",memberName];
    
    if ([memberNameString isEqualToString: @"_name"]) {
        object_setIvar(person, ivar, @"newName");
    }
}
    
```

runtime的做法相对来讲代码比较多，也不够简洁，但是还是可以实现这个需求的。runtime的详细内容可以参考[iOS Runtime 详解](https://njafei.github.io/2017/05/04/runtime/)
