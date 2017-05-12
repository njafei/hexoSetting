---
title: 《ES6入门》读书笔记之let和const命令
date: 2017-04-17 15:37:38
tags: 
	- React-Native
	- 读书笔记
categories: React-Native
---
正在读阮一峰的[《ECMAScript 6 入门》](http://es6.ruanyifeng.com/),本系列博客都是读书笔记。

# ES6 PlayGround
在介绍具体的内容之前，想给大家介绍一个好玩的playgroud，尤其合适边看书，边敲代码的同学们。[Traceur](https://google.github.io/traceur-compiler/demo/repl.html#%7B%0A%20%20let%20a%20%3D%201%3B%0A%20%20var%20b%20%3D%200%3B%0A%20%20console.log(a)%3B%0A%7D%0A%0Aconsole.log(b)%3B%0A%0A)
这个工具会在你敲完每行代码之后帮你执行检查是否有错误，然后翻译成es5并执行，然后再配合JS控制台，就是很棒的playground了。
使用：

 1. 打开网页
 2. command + option + J打开javascript控制台
 3. 在最左边的框里面输入代码，

效果如下：
左边：写代码的地方  中间：翻译之后的js代码，右边：控制台
![](http://on0hv7n2x.bkt.clouddn.com/es6%20playground.png)

# let命令

## 作用域和var不同

ES6中建议全部使用let来生命变量，let和var的作用用法类似，但是let声明的变量，作用域是在其所在的代码块。

```
{
	var a = 0;
  	let b = 1;
}

console.log(a);
console.log(b);//ReferenceError： b is not defined
```

看起来很简单吧，那我们来看下道题目吧



```
var a = [];
for(var i = 0; i< 10 ; i++){
  a[i] = ()=>{
  	console.log(i)
  }
}

a[7]();
```
和

```
let b = [];
for(let i = 0; i< 10 ; i++){
  b[i] = ()=>{
  	console.log(i)
  }
}

b[7]();

```

答案是：
a(7)() : 10
b(7)() : 7

你猜对了吗？0_0
我第一次看其实是答错了的，然后我把a[7] 和 b[7]打印了出来


```
function () {
      console.log(i);
    }
```

我理解是因为let和var的缘故，所以a[7]里面的i实际是就是一个static的变量，在i++之后变成了10，而b[7]里面的i就是7




## 变量提升
let 不会出现变量提升的现象。
首先，我们把let抛一边,看下什么是变量提升。先看个代码，猜猜它的结果：

```
var a = 5;
function f(){
  if(!a){
	  a = 100;
	}
  console.log(a);
}

f();
```
它的输出结果是多少？

答案： 5，很简单吧，好，我们再看一个相似的函数：

```
var a = 5;
function f(){
  if(!a){
	 var a = 100;
	}
  console.log(a);
}

f();
```

结果是多少呢？
答案是：100

为什么会这样呢？

#### 作用域
作为一个一直在使用OC的程序员，想要弄懂JS的作用域，开始会很别扭，因为两者的作用域是基于不同的标准或者模式。在介绍之前，我们先看下两个C和JS的小例子：

C：

```
#include <stdio.h>
int main() {
	int a = 1;
	printf("%d",a); //1
	{
		int a = 2;
		printf("%d",a); //2
	}
	printf("%d",a);/1
}
```

JS:

```
var a = 1;
console.log(a); // 1
{
    var a = 2;
    console.log(a); // 2
}
console.log(a); // 2

```

结果不一样了，为什么呢？ 刚才提到C和Js的作用域的模式不一样，C是基于块级的作用域（block-level scope），每个大括号括起来的都可以理解为一个小的作用域，如果变量在小的作用域里声明，那么在小的作用域中是会忽略外部同名的变量。

而在JS中，则是基于函数的作用域，即每个函数都有自己的作用域（function-level scope），所以上述的列子最后的结果就不一致了，在JS的列子中，a的值其实被覆盖。

C,C++,Java都是块级作用域，那么JS中，如何实现类似的效果呢？答案是使用闭包

```
var a = 0;
fucntion f(){
	var a = 1;//1
	console.log(a);//1
}
console.log(a);//0
```

在f()这个函数中，会再次定义一个只能在f()中起作用的a,从而实现了类似块级作用域的效果。

#### 变量提升
讲完了作用域，我们来看下什么是变量提升，还是先来个列子：

```
var a = 0;
f();
var b = 1;
```

这三行代码在解释器中会变成：

```
var a , b;
a = 0;
f();
b = 1;
```

所有var声明的变量的声明语句，都会被解释器给放到变量所在作用域的顶部。注意，只是把生命语句放到最上面，但是不会把赋值等位置提升，这就是所谓的变量提升。

函数也会有变量提升的现象，但是会根据声明方式的不同，有着不同的结果。创建函数的方法有两个： function f(){} 和 var f = function(){},他们会有什么不同呢？ 我们看下列子：

```
f1();// TypeError "foo is not a function"
f2();// will run

var f1 = function(){
	console.log('won't run');
}

function f2(){
	console.log('will run');
}
```

这段代码在解释器中：

```
var f1();
function f2(){
	console.log('will run');
}

f1();// TypeError "foo is not a function"
f2();// will run

var f1 = function(){
	console.log('won't run');
}
```

函数的变量提升，如果是`fucntion f()`的形式，怎会整个函数都提升到顶部，如果是`var f() = function(){}`的形式，则只会提升`var f()`到顶部。



如此，本节开头的两个例子就不难理解了。

```
var a = 5;
function f(){
  if(!a){
	  a = 100;
	}
  console.log(a);
}

f();
```
在解释器中是：

```
var a;
function f(){
  if(!a){
	  a = 100;
	}
  console.log(a);
}
a = 5;

f();
```


```
var a = 5;
function f(){
  if(!a){
	 var a = 100;
	}
  console.log(a);
}

f();
```
在解释器中是：

```
var a;
function f(){
  var a;	 
  if(!a){
	 a = 100;
	}
  console.log(a);
}
a = 5;
f();
```

而let 关键字是不具备变量提升的，所以它声明的变量，其实就是块级作用域。

最后一个例子：

```
var a = 0;
let b = 0;
{
	var a = 1;
	let b = 1;
}

console.log(a);//1
console.log(b);//0
```

而在ES5中不会报错的先使用再声明的模式，在ES6中用let的话，就会报错了。比如：

```

if (true) {
// TDZ开始
tmp = 'abc'; // ReferenceError
console.log(tmp); // ReferenceError
let tmp; // TDZ结束
console.log(tmp); // undefined
tmp = 123;
console.log(tmp); // 123
}

```

所以建议如下：

 - ES6中永远使用let
 - 所有变量的声明，都写在函数的顶部



这是我在看变量提升的时候，找到一篇质量很棒的blog，本篇的结构和内容也参考了很多[Javascript作用域和变量提升](https://segmentfault.com/a/1190000003114255)

# const
const的用法和let基本一致，不可重复定义，不会变量提升，作用域是块作用域。

要注意的是，const修饰一个对象的话，只会限定这个对象的地址不变，不会限定它的值不变，如：

```
const foo = {};
foo.pro = 'haha';//work
```

想要值不变的话，可以使用`Object.freeze()`方法：

```
const foo = Object.freeze({});
foo.pro = 'haha';//not work
```


如果想将对象全部冻结，要将里面的每个value都冻结：

```
let constantize = (obj) => {
	Object.freeze(obj);
	Object.keys(obj).forEach((key,value) => {
			if (typeof obj[key] === 'object' ) {
				constantize(obj[key]);
			}
		}
	)
}
```




























