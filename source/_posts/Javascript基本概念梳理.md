---
title: Javascript基本概念梳理
date: 2014-06-13 18:48:16
tags:
- javascript
- 面试
categories:
- 149
---
<p style="color: #362e2b;">javascript里的数据类型：</p>

<blockquote style="color: #362e2b;">原始类型：数字，字符串，布尔值。（原始值：null，undefined）

对象类型：键值对，数组，function，全局对象（MATH，JSON）

保留字：export，NaA。。

</blockquote>
<!--more-->
<span style="color: #362e2b;">包装对象的概念：</span>
<blockquote style="color: #362e2b;">字符串"aaa".len 字符串并不是对象，但是却可以调用它的属性，说明这只是一个临时对象，内部用new String（）来创建的临时的。</blockquote>
<p style="color: #362e2b;">原始类型是永远不可变的，所以可以比较他们的值，但是对象类型是可变的，不能比较他们的值.</p>
<p style="color: #362e2b;">Javascript原型和继承：</p>



Javascript里每个对象都和另外一个对象关联，这个对象就是__proto__（原型对象）注意这里的原型对象并不是prototype。

解释一下：这里的prototype指的是通过关键字new和构造函数调用创建的对象的原型就是构造函数的prototype属性。

对象实例的__proto__指向这个对象的prototype，而对象的__proto__为空。举个例子就是：


```javascript
var array = new Array();  
array.__proto__ === Array.prororype  //true  
Array.__proro //null
```
当然，也可以使用Object.getPrototypeOf()替代__proto__来使用来得到对象所继承的原型，举例说明：
```javascript
Object.getPrototypeOf(Array) === Array.__proto__;
```
Object.getPrototypeOf()来查看原型继承，例如：
```javascript
Object.getPrototypeOf(Array.prototype) // Object
```
可以看出Array的prototype继承Object所以Array也有他的方法例如totring()等。可以得到所有的对象都有一个共同的原型，就是Object但是Object只是一个构造函数，想要访问他，就只用Object.prototype来得到。


Javascript的实例属性和原型属性：
```javascript
function A(){};
A.prototype.title = '123'; // 原型属性
var a = new A();
console.log(a.title); // 123
a.title = '234';// 实例属性
console.log(a.title); // 234

a.hasOwnProperty('title'); // 只能访问到实例属性 不能访问原型属性

// for in的时候要加hasOwnProperty判断

```
Javascript实现继承的方式：

<p style="color: #362e2b;">自定义继承：</p>
```javascript
function A(){};  
function B(){};
// A 继承 B
A.prototype = new B();  
//or
var b = new B();
A.prototype = b;
// 两种写法差不多

```

例如，Object.getPrototypeOf()来查看自定义的继承
```javascript
function A(){};  
function B(){};  
A.prototype = new B();  
Object.getPrototypeOf(A.prototype) //B
```

<p style="color: #362e2b;">使用Object.create()实现继承：</p>


Object.create()接受一个参数，为对象的prototype，其实还有第二个参数用来描述熟悉的特性，在es5中直接实现了Object.create()这个方法不用自己写了
```javascript
Object.create = function (o) {  
  
     var F = function () {};  

     F.prototype = o;  

     return new F();  

 };
function A(){}; 

var b = Object.create(A.prototype);
```
Object.create()可以创建对象，当然也可以创建对象的子对象，可以这样理解
```javascript
var a = Object.create({a:1})
```
那么a就有了一个熟悉a，这样就可以理解为继承了，如果是一个函数，例如Array是一个函数对象
```javascript
var myArray = Object.create(Array.prototype)
```
那个myArray也就具有了Array的所有方法

myArray.push

自定义的函数
```javascript
function Acc(){}  
Acc.prototype.dd = 123;  
var accc = Object.create(Acc.prototype)；  
accc.dd //123
```
未完！