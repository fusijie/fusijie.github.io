---
title: JavaScript-基础介绍
date: 2016-05-06 00:26:23
categories: JavaScript
tags: [JavaScript, 基础, 读书笔记]
---

接触 JavaScript 也有快2年的时间了，因为工作的原因，涉猎的编程语言有点过多了（大概算算，有 c/c++，java，oc，js，lua，c#，python，shell），我本身是很喜欢研究编程语言的，特别在写了多门语言之后不自觉地就会进行比较，比较语言特性上的各种优劣，总归还是很有乐趣的。

为什么我突然又开始写起 JavaScript 教程?主要是 JavaScript 的使用比重一直在上升，使得我必须全面严谨地再研究一遍，当然实话就是 JavaScript 要统一天下，这从雨后春笋般的框架就可以看到。而我主攻的 Cocos 引擎相关的业务也围绕 JavaScript。但是学的语言多，避免不了的问题就是精力的分散，很多东西都无法深入，同时在语法上，标准库的使用也很容易混淆。

刚才说这是 JavaScript 教程，其实不能算，顶多就是个读书笔记，而且会很零散，可能会对一些回顾 JavaScript 的人会有空吧。最后补一句，云风说得对，JavaScript 真是一门恶心的语言。

<!-- more -->

### 句末分号

JavaScript 允许句末不用分号，实际上是浏览器中的 JavaScript 解析引擎会自动帮你加上，但是有过编程经验的人都知道，这些东西不能省就是不能省，不是规则，是规范。

### Number

JavaScript 不区分整数和浮点数，统一采用 Number 数据类型。Number 有几个属性，需要注意下，其中 NaN 只能用 isNaN() 做判断。

```
1/0;//Infinity
0/0;//NaN
10.5%3;//1.5
```

### char

JavaScript 没有 char 类型，单个字符也是字符串，长度为1。

### 两个感叹号

两个感叹号可以强转为布尔型。

### == 和 ===

前者会转换数据类型，后者不会，大部分情况下请使用 ===。要注意：

```
isNaN === isNaN;// false
```

同时也要注意浮点的问题

```
1/3 === (1 - 2/3); // false
Math.abs(1/3 - (1 - 2/3)) < 0.0000001; // true
```

### null 和 undefined

这也是一个很蛋疼的设计，主要的区别是 null 是存在的，而 undefined 是不存在的。

### 数组

创建数组可以使用 []，也可以使用 new Array()，从可读性和性能考虑，建议使用前者。

### var

允许不使用 var 来声明变量也是一个蛋疼的问题。如果一个变量没有通过 var 声明，那该变量就会被自动声明为全局变量。但是这句话的意思并不是说通过 var 声明了，就不是全局变量了，还是得看作用域。

### 严格模式

```
'use strict';
```

严格模式要求变量必须用 var 声明，不然直接报错。当然还有一些其他限制，后面会谈到。

### 对象

JavaScript 的对象是一组由键值对组成的无序集合，其中键是字符串。

如果键符合变量的命名要求，比如 abc，那么可以写成 obj['abc']，但是一般我们习惯写成 obj.abc。当然键也可以不符合变量命名要求，比如 a-b-c，比如 1，这个时候就不能用点号了，只能写成 obj['a-b-b'] 和 obj['1']。从这个意义上说，也可以理解数组是一种特殊的对象，arr[0] 只是 arr['0'] 的简写，而且数组可以在任意位置插入值。

### 关键字 in

in 可以用于判断某个属性是否是某个对象所有，而 hasOwnProperty 可以判断某个属性是否是自身或者继承而来。

### 字符串

记住字符串都是不可变的，所以所有对字符串的操作都是返回一个新字符串，如果对字符串某个索引赋值，不报错但也无效。

### Array

数组是可变的，所以需要注意的是对数组的操作到底是修改了数组本身还是返回了一个新数组。

如果判断一个对象是否是 Array？

```
var isArray = function(obj) { 
return Object.prototype.toString.call(obj) === '[object Array]'; 
}

```

或者

```
Array.isArray(obj);
```

### if...else... 多行条件判断

实际上 JavaScript 是没有 elseif 的，平时写的 else if 只是两层 if...else... 的嵌套，方便阅读。

### JavaScript 的循环

第一种是普通的 for 循环，由三个条件组成。

第二种是 for...in 循环，for...in 循环可以把一个对象的所有属性遍历出来，注意输出的键值是 字符串，当然也可以把一个数组的所有属性都遍历出来，和对象一样的是输出的键值是字符串而不是数字，不一样的是它不输出继承下来的属性而对象会输出所有属性包括继承的。

第三种是 forEach，但是只能遍历数组。（ES 5.1）

### Map，Set，for...of

这些都是 ES6 标准，鉴于目前浏览器的支持情况，不介绍。

[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
[Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
[for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)