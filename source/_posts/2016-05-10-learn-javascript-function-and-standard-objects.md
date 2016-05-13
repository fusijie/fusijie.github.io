---
title: JavaScript-函数和标准对象
tags: [函数, 标准对象]
date: 2016-05-10 21:46:47
categories: JavaScript
---

JavaScript 的函数是第一类对象，可以动态创建，可以做传参，可以做返回值等等，而且由于 JavaScript 函数级作用域的关系，匿名函数的使用十分广泛，所以使用起来非常的灵活。

### arguments
arguments 也是个奇葩的东西，类似 Array 又不是 Array，但是一般情况下当 Array 使用就可以了。

### return
因为句末分号的关系，return 后面的语句不要换行写。

<!-- more -->

### 函数级别的变量作用域
简简单单一个作用域，非要搞得这么恶心，还好 ES 6 引入了 let 关键字来处理块级作用域。但是不管怎样也没有 Python 的 变量作用域恶心。。。

### 变量提升
这个蛋疼的特性我也是无力吐槽了，所以请注意在函数的内部首先声明所有的变量。

### 全局作用域
不在函数内声明的变量就具有全局作用域。默认所有全局作用域都绑到 window 上。

### this
相信我说 this 是 JavaScript 最恶心的东西没人会反对。。。不说啥了，自己体会。

### apply, call
函数调用的两种方式，主要方便在可以控制 this 的指向，前者将参数打包为 Array 传入，后者将参数按序传入。

### 高阶函数
map，reduce，filter，sort，和 python 基本上是一样的，都是对 Array 进行处理。

### 闭包
本质上，闭包是一个携带环境状态的函数，所以在返回函数的时候不要引用任何的循环变量以及后续会发生变化的变量。

### delete
* 不是所有的属性和方法都能被删除。
* var 声明的变量，对象，数组都不能被删除。
* 删除数组的某个索引，不会影响数组的长度。
* 不要根据返回值判断是否删除成功。

### typeof
typeof 获取对象类型，返回小写字母开头的字符串。 

```
typeof null === 'object'
typeof [] === 'object'
typeof undefined === 'undefined' //判断某个变量是否有声明
```

### Copy

```
/* 检测对象类型
 * @param: obj {JavaScript Object}
 * @param: type {String} 以大写开头的 JS 类型名
 * @return: {Boolean}
 */
function is(obj, type)  {
  return Object.prototype.toString.call(obj).slice(8, -1) === type;
}

/* 复制对象
 * @param: obj {JavaScript Object} 原始对象
 * @param: isDeep {Boolean} 是否为深拷贝
 * @return: {JavaScript Object} 返回一个新的对象
 */
function copy(obj, isDeep) {
  var ret = obj.slice ? [] : {}, p, prop;
  // 配合 is 函数使用
  if(!isDeep && is(obj, 'Array')) return obj.slice();
  for(p in obj) {
    if(!obj.hasOwnProperty(p)) continue;
    prop = obj[p];
    ret[p] = (is(prop, 'Object') || is(prop, 'Array')) ? 
      copy(prop, isDeep) : prop;
  }
  return ret;
}
```

### 包装对象
int 和 Integer 的关系。不要使用包装对象。数字和字符串的转换，使用 parseInt，parseFloat，减0，toString，加空串。

另外对基础类型的处理需小心，如下，基础类型 string 会被**临时**提升为 String 类。

```
var a = 'abcdefg';
a.pro = 100;
console.log(a.pro);//undefined
var b = new String('abcdefg');
b.pro = 100;
console.log(b.pro);//100
```

### Date
特别注意月份索引从 0 开始。getTime() 返回从 1970.1.1 开始**的毫秒时间**。

### JSON
JSON.stringify 可以带参数 replacer（函数）和 space（空格）

```
var obj = {x:1, y: {a:2, b: 3}};
JSON.stringify(obj, function(k,v){return v;}, '    ');
"{
    "x": 1,
    "y": {
        "a": 2,
        "b": 3
    }
}"
```

### rest，let，const，箭头函数，generator
这些都是 ES6 标准，鉴于目前浏览器的支持情况，不介绍。

[rest](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)

[let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

[const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)

[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

[generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)