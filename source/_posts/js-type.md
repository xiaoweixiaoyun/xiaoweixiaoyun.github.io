---
title: Js的数据类型
date: 2021/09/16
tags: 前端,JS,类型
categories: JS
description: Javascript数据类型分为基本数据类型与引用数据类型
keywords: 前端,JS,类型
cover: /img/md/js.jpg
---

# 数据类型分类
## 基本类型
>字符串（String）、数字(Number)、布尔(Boolean)、空（Null）、未定义（Undefined）、Symbol。

## 引用类型
>对象(Object)、数组(Array)、函数(Function)，还有两个特殊的对象：正则（RegExp）和日期（Date）

![JavaScript 数据类型](/img/md/pictures/Javascript-DataType.png)

注：Symbol 是 ES6 引入了一种新的原始数据类型，表示独一无二的值。

# 数据类型判断

## typeof

>基本数据类型中：Number，String，Boolean，undefined 以及引用数据类型中Function ,可以使用typeof检测数据类型,分别返回对应的数据类型小写字符。
另：用typeof检测构造函数创建的Number，String，Boolean都返回object
基本数据类型中：null 。引用数据类型中的：Array，Object，Date，RegExp。不可以用typeof检测。都会返回小写的object
```javascript
console.log(
    typeof 100, //"number"
    typeof 'abc', //"string"
    typeof false, //"boolean"
    typeof undefined, //"undefined"
    typeof null, //"object"
    typeof [1,2,3], //"object"
    typeof {a:1,b:2,c:3}, //"object"
    typeof function(){console.log('aaa');}, //"function"
    typeof new Date(), //"object"
    typeof /^[a-zA-Z]{5,20}$/, //"object"
    typeof new Error() //"object"
    typeof new Number(100), //'object'
    typeof new String('abc'),// 'string'
    typeof new Boolean(true),//'boolean'
);
```

## instanceof

>除了使用typeof来判断，还可以使用instanceof。instanceof运算符需要指定一个构造函数，或者说指定一个特定的类型，它用来判断这个构造函数的原型是否在给定对象的原型链上。
```javascript
console.log(
    100 instanceof Number, //false
    'dsfsf' instanceof String, //false
    false instanceof Boolean, //false
    undefined instanceof Object, //false
    null instanceof Object, //false
    [1,2,3] instanceof Array, //true
    {a:1,b:2,c:3} instanceof Object, //true
    function(){console.log('aaa');} instanceof Function, //true
    new Date() instanceof Date, //true
    /^[a-zA-Z]{5,20}$/ instanceof RegExp, //true
    new Error() instanceof Error //true
)
```

基本数据类型中：Number，String，Boolean。字面量值不可以用instanceof检测，但是构造函数创建的值可以，如下：
```javascript
var num = new Number(123);
var str = new String('dsfsf');
var boolean = new Boolean(false);
```

还需要注意null和undefined都返回了false，这是因为它们的类型就是自己本身，并不是Object创建出来它们，所以返回了false。

## constructor

>onstructor是prototype对象上的属性，指向构造函数。根据实例对象寻找属性的顺序，若实例对象上没有实例属性或方法时，就去原型链上寻找，因此，实例对象也是能使用constructor属性的。

如果输出一个类型的实例的constructor，就如下所示：
```javascript
console.log(new Number(123).constructor)
//ƒ Number() { [native code] }
```

可以看到它指向了Number的构造函数，因此，可以使用num.constructor==Number来判断一个变量是不是Number类型的。
```javascript
var num  = 123;
var str  = 'abcdef';
var bool = true;
var arr  = [1, 2, 3, 4];
var json = {name:'wenzi', age:25};
var func = function(){ console.log('this is function'); }
var und  = undefined;
var nul  = null;
var date = new Date();
var reg  = /^[a-zA-Z]{5,20}$/;
var error= new Error();

function Person(){
  
}
var tom = new Person();

// undefined和null没有constructor属性
console.log(
    tom.constructor==Person,
    num.constructor==Number,
    str.constructor==String,
    bool.constructor==Boolean,
    arr.constructor==Array,
    json.constructor==Object,
    func.constructor==Function,
    date.constructor==Date,
    reg.constructor==RegExp,
    error.constructor==Error
);
//所有结果均为true

```
除了undefined和null报错之外，其他类型都可以通过constructor属性来判断类型。

## 使用Object.prototype.toString.call()检测对象类型

>可以通过toString() 来获取每个对象的类型。为了每个对象都能通过 Object.prototype.toString() 来检测，需要以 Function.prototype.call() 或者 Function.prototype.apply() 的形式来调用，传递要检查的对象作为第一个参数，称为thisArg。

```javascript
var toString = Object.prototype.toString;

toString.call(123); //"[object Number]"
toString.call('abcdef'); //"[object String]"
toString.call(true); //"[object Boolean]"
toString.call([1, 2, 3, 4]); //"[object Array]"
toString.call({name:'wenzi', age:25}); //"[object Object]"
toString.call(function(){ console.log('this is function'); }); //"[object Function]"
toString.call(undefined); //"[object Undefined]"
toString.call(null); //"[object Null]"
toString.call(new Date()); //"[object Date]"
toString.call(/^[a-zA-Z]{5,20}$/); //"[object RegExp]"
toString.call(new Error()); //"[object Error]"
```

这样可以看到使用Object.prototype.toString.call()的方式来判断一个变量的类型是最准确的方法。

# 封装一个获取变量准确类型的函数

```javascript
function gettype(obj) {
  var type = typeof obj;
  //如果不是object类型的数据，直接用typeof就能判断出来
  if (type !== 'object') {
    return type;
  }
  //如果是object类型数据，准确判断类型必须使用Object.prototype.toString.call(obj)的方式才能判断
  return Object.prototype.toString.call(obj).replace(/^\[object (\S+)\]$/, '$1');
}
```
