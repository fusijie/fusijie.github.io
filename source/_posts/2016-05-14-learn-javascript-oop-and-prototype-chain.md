---
title: JavaScript-OOP 和原型链
tags: [OOP, 面向对象, 原型链, 继承]
date: 2016-05-14 16:24:37
categories: JavaScript
---

JavaScript 世界里，一切都是对象。

### 设计思想

JavaScript 继承机制比较特殊，经典 OOP 的开发者可能一开始会懵逼，无法理解为什么没有类和实例的概念，无法理解为什么使用原型链，这要从 JavaScript 设计之初讲起。

JavaScript 在设计的时候，目的很简单，只是想完成一些简单的操作，所以 [Brendan Eich](http://brendaneich.com/) 觉得没必要设计得很复杂。但是 C++ 和 Java 盛行，JavaScript 免不了收到 OOP 的影响，所以有了开头那句话，只是继承机制怎么处理？C++ 和 Java 使用的方式是 new Object()，为了保证 JavaScript 的简易和降低入门难度，他并不打算引入类的概念，反正 C++ 和 Java 调用 new 命令的时候都会调用 Object 的构造函数，那就简化一下，直接 new construct() 。

<!-- more -->

### \_\_proto\_\_

JavaScript 实现继承的方式有很多种，最简单的，将对象 a 的 \_\_proto\_\_ 指向对象 b，则对象 a 就继承于对象 b。但是不建议直接使用 \_\_proto\_\_ 来修改原型，可以通过 Object.create(obj) 传入原型返回一个继承于 obj 的对象。**请注意， obj.\_\_proto\_\_ 即是对象 obj 的原型，但是这个属性并不是标准属性，可以用来查看，但是请不要使用它。**

当访问某一个对象的属性，JavaScript 会顺着该对象的原型链查找直至找到该属性为止，如下实现，By [John Resig](http://ejohn.org/blog/objectgetprototypeof/)：

```
function getProperty(obj, prop) {
  if (obj.hasOwnProperty(prop))
    return obj[prop]
  else if (obj.__proto__ !== null)
    return getProperty(obj.__proto__, prop)
  else
    return undefined
}

```

### prototype

对于经典 OOP 的用户来说，类和实例的区别一目了然，但是到了 JavaScript 就会发现不管用了，主要的原因就是 JavaScript 采用的是原型链的继承方式，而 JavaScript 本身也没有类的概念，看看本文的第一句话。

要理解 prototype 其实不难，记住两点：

* 所有的对象都有 constructor 属性，指向这个对象的构造函数，比如 var arr =[1,2,3]，其实就是 var arr = new Array(1,2,3)，所以 arr 的构造函数就是 Array，同样的 Object 也是一个构造函数。

* 所有的构造函数都有 prototype 属性，指向了一个对象。这个对象就是使用该构造函数生成的对象的原型。

所以尝试一下理解以下代码：

定义一个数组，没问题。

```
var arr = [1,2,3];
```

如上第一点说的，arr 的构造函数就是 Array。

```
arr.constructor === Array; //true
```

如上第二点说的，构造函数有一个 prototype 属性，指向了一个对象，也就是 arr 的原型。

```
arr.constructor.prototype; //arr 的原型
```

综合以上两点，可知 Array.prototype 就是所有 arr 的原型，从控制台可以看到 Array.prototype 身上挂了一堆方法，比如 push，join等等，如果我们想扩展 Array 的方法，没错，就是 Array.prototype.myfunction = function () {...}，只是对 arr 进行 for 循环的时候会被打印出来。

```
Array.prototype;//arr 的原型
```

刚才提的__proto__它指向的是对象的原型，所以其实是一个东西。

```
arr.__proto__ ===  arr.constructor.prototype;//true
```

再来看一个：

```
arr.constructor.prototype.constructor === Array;//true
```

看到这可能有人就懵逼了。。。没错，这是一个设定，也就是说它可以不按套路出牌。这是被强制指向 Array的，其实本来它应该是指向 Object。因为从原型链上来看，arr.constructor.prototype 实际上是一个对象，它的原型就是一个 Object 对象，所以它指向的是 Object 构造函数，但是套路就是套路。所以在重新定义 prototype 的时候需要重定向 constructor 到其构造函数，而不是 Object。可参考 [Ruanyifeng's blog](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html)。

和上一行一样。

```
Array.prototype.constructor === Array;//true
```

再看一看这个，就证明了 Array 原型链上是 Object。

```
Object.prototype.isPrototypeOf(Array.prototype);//true
Array.prototype.__proto__ === Object.prototype;//true
```

最后写一个无关的，但是帮助理解，Array 和 Object 都是构造函数，当然也是对象，所以它也有 constructor，函数的构造函数是 Function。

```
arr.constructor.constructor === Function;//true
```

当原型链处在最顶层的时候，即 Object.prototype，这时候情况又有点不一样了。

```
Object.prototype.constructor.prototype === Object.prototype;//true，你可以玩一年
Object.prototype.__proto__ === null;//true，别问我为什么/doge
```

下面放图，很明显的是虽然 \_\_proto\_\_ 不是标准属性，但是相对来说却是**少一点套路多一点真诚**的属性。当然这个例子比较复杂也比较特殊（Array 是 JavaScript 内置对象），如果只是简单的 obj 继承于 Object，会更直观清晰一点，这里不再分析。

<div align="center"><img src="/images/posts/2016-05-14-learn-javascript-oop-and-prototype-chain/prototype.png" alt="" border="0" /><br></br></div>


### 构造函数

聊到这，我们再来看看 new 的时候到底做了什么？其实主要是三点，以 f 为 构造函数：

* var obj = new Object();//分配内存
*  obj.\_\_proto\_\_ = f.prototype;//指向原型
* f.call(obj);//调用构造函数，传入 this

```
// New implementation
function New (f) {
	var n = { '__proto__': f.prototype };
	return function () {
		f.apply(n, arguments);
		return n;
	};
}

// Test
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype = {
  print: function () { console.log(this.x, this.y); },
  constructor: Point
};

var p2 = New (Point)(10, 20);
p2.print(); // 10 20
console.log(p2 instanceof Point); // true

```

### 为什么扩展方法要写在原型上？

刚才我们有提过如果要在 Array 上扩展方法，这么做：

```
Array.prototype.myfunction = function () {...}
```

为什么呢，我们再来看个例子：

```
var f = function(){
    this.sayHello = function(){
        console.log('hello');
    }
}

var a = new f();
var b = new f();
a.sayHello === b.sayHello;//false，
```

因为 new 的时候实际上调用到了 f.call(obj)，所以对于每一个 obj 都会有自己的一份 sayHello。而实际上这是不需要的，sayHello 完全可以共用。所以 sayHello 可以挂在原型上。

```
var f = function(){}

f.prototype.sayHello = function () {
    console.log('hello');
}

var a = new f();
var b = new f();
a.sayHello === b.sayHello;//true

```

### 谨慎处理原型链上的引用对象

什么意思呢，举个例子：

```
var f = function(){};
f.prototype.arr = [1,2,3];//[1,2,3]
var a = new f();
a.arr;//[1,2,3]
a.arr.push(4);//如果修改了 arr，因为 arr 是挂在原型链上的，又是引用对象，所以这里会改掉原型链上的 arr
f.prototype.arr;//[1,2,3,4]
var b = new f();
b.arr;//[1,2,3,4] 这时候再 new 个 f 的对象出来，arr已经被改动了。
```

按照 OOP 的思想，类是不应该被对象实例所影响的，原型是不可以被改变的。

再来看一个例子：

```
var f = function(){};
f.prototype.x = 10;
var a = new f();
a.x;//10
var b = new f();
b.x;//10
a.x = 20;//修改 a.x 为 20
a.x;//20
b.x;//10，再看看 b.x 发现仍然是10
a.constructor.prototype.x === 10;//true，原来修改 a.x 只是在 a 对象上增加了一个 x 的属性，并不会影响到原型
f.prototype.arr = [1,2,3];
a.arr;//[1,2,3]
a.arr = [4,5,6];//同样的，这里也是在 a 对象上增加了一个 arr 属性，并不会影响到原型
a.constructor.prototype.arr === a.arr;//false
```

### John Resig 的 JavaScript 继承实现

参见 John Resig 的博客说明 [Simple JavaScript Inheritance](http://ejohn.org/blog/simple-javascript-inheritance/)，这套 class 的实现也是 Cocos2d-JS 采用的类继承方案。

### class

这是 ES6 标准，鉴于目前浏览器的支持情况，不介绍。

[class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/class): 实际上 class 是对原型继承的封装，它简化了原型继承的操作且避免了原型继承代码的分散。