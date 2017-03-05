---
title: 详解JavaScript中的__proto__与prototype（上）
date: 2016-10-18 19:58:39
tags:
- JavaScript
categories:
- 学习笔记
- JavaScript
---

先看一段代码
```
Function instanceof Object // true 
Object instanceof Function // true 
Function instanceof Function //true 
Object instanceof Object // true 
Number instanceof Number //false

```
<!-- more -->
再来看一段
```
var obj = { a : 1 }; 
console.log(obj.__proto__ === Object.prototype); // true 

var str = new String('123'); 
console.log(str.__proto__ === String.prototype); // true 


function Point(){}; 
var Circle = Object.create(Point); 
console.log(Circle.__proto__ === Point); // true 
console.log(Circle.__proto__ === Point.prototype); // false 

var p = new Point(); 
console.log(Point.__proto__); // function() 
console.log(Point.prototype); // Point {} 
console.log(p.__proto__); // Point {} 
console.log(p.prototype); // undefined
```

很明显，\_\_proto\_\_与prototype看起来似乎很相似，但是实际上是不同的。
先了解一下各自是什么，再进一步探讨。
### prototype 显示原型
每一个函数在创建之后都会拥有一个名为prototype的属性，这个属性指向函数的原型对象。(通过Function.prototype.bind方法构造出来的函数是个例外，它没有prototype属性。)

### \_\_proto\_\_ 隐式原型
JavaScript中任意对象都有一个内置属性[[prototype]]，在ES5之前没有标准的方法访问这个内置属性，但是大多数浏览器都支持通过__proto__来访问。ES5中有了对于这个内置属性标准的Get方法Object.getPrototypeOf().
Note: Object.prototype 这个对象是个例外，它的\_\_proto\_\_值为null 

### <span id="two_rel">二者关系</span>
隐式原型指向创建这个对象的函数(constructor)的prototype
```
function test(){};
var a = new test();
console.log(a.__proto__ === test.prototype) ;// true
```


### 梳理
JavaScript里万物皆对象，方法（Function）是一种特殊的对象。
由上述的定义可知，任意对象都有都有\_\_proto\_\_,而prototype只有函数创建之后自己才有,通过该函数创建的对象是没有的。
```
function test(){};
var a = new test();
console.log(test.prototype); // test{}
console.log(a.prototype); // undefined
```

而Object.prototype这个特殊的对象的\_\_proto\_\_位于原型链金字塔的顶端,为null，如下图
{% asset_img 1.png 图形化原型链 %}
这里值得一提的是JavaScript的原型链中原型对象prototype之间是通过\_\_proto\_\_联系起来的
{% asset_img 3.png __proto__ %}
同时原型对象prototype中都有个预定义的constructor属性，用来引用它的函数对象。这是一种循环引用

```
  person.prototype.constructor === person //true
  Function.prototype.constructor === Function //true
  Object.prototype.constructor === Object //true
```
#### 这一点在本文后面的图中也有显示，<a href = "#last_pic">Click</a>
### 有两点需要注意：
> （1）注意Object.constructor===Function；//true 本身Object就是Function函数构造出来的
> （2）如何查找一个对象的constructor，就是在该对象的原型链上寻找碰到的第一个constructor属性所指向的对象

再看上面一截代码中的部分
```
function Point(){}; 
var Circle = Object.create(Point); 
console.log(Circle.__proto__ === Point); // true 
```
Object.create(),这是ES5中新增的方法，在这之前这被称为原型式继承,我们可以理解为 new Object(),这样我们就很好理解结果true了，实际上，上述代码可以等价理解为
```
function Point(){};
var Circle = new Point();
console.log(Circle.__proto__ === Point); // true 
```
等价的原因在上面<a href="#two_rel"><strong>二者关系</strong></a>有提到。
也可以从constructor来理解
```
console.log(Circle.constructor); //function Point()
console.log(Circle.constructor.prototype ===  Circle.__prototype) // true
```

### instanceof
instanceof 运算符用来测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性。
#### 语法
```
object instanceof constructor
```

#### 参数

object
    要检测的对象.

constructor
    某个构造函数

#### 描述

instanceof 运算符用来检测 constructor.prototype 是否存在于参数 object 的原型链上。

```
//设 L instanceof R 
//通过判断
 L.__proto__.__proto__ ..... === R.prototype ？
//最终返回true or false
```

也就是沿着L的\_\_proto\_\_一直寻找到原型链末端，直到等于R.prototype为止,如果一直到Object.prototyoe都没找到，则返回false。就此，我们结合下面这幅图再剖析一下文章开头部分的代码段就很容易理解了
<span id="last_pic"></span>
{% asset_img 2.jpg __proto__与prototype %}

### 参照知乎[知乎--js中__proto__和prototype的区别和关系？](http://www.zhihu.com/question/34183746)整理得出，加入部分个人见解，有错之处望各位大牛斧正






