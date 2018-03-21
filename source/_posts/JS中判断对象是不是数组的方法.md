---
title: JS中判断对象是不是数组的方法
date: 2018-02-02 09:57:40
tags:
    - js
category:
    - js
---

今天有同事突然问到如何判断一个对象是不是数组，当时只想到啦`instanceof`操作符，后来又搜集了一些文章，总结出以下方法。
#### 一、typeof 操作符
在js中使用`typeof`操作符来判断`function string number boolean undefined`等是没有问题的。但是如果要检测Array就没有作用了，因为`typeof`判断`array`和`null`的结果都是`object`。
```
alert(typeof null); // "object"
alert(typeof function () {
return 1;
}); // "function"
alert(typeof 'abcd'); // "string"
alert(typeof true); //boolean
alert(typeof 1); // "number"
alert(typeof a); // "undefined"
alert(typeof undefined); // "undefined"
alert(typeof []); // "object"
```
#### 二、instanceof 操作符
```
var arr = [];
console.log(arr instanceof Array);
```
`arr instanceof Array`的含义是：判断`Array`的`prorotype`属性是不是在`arr`的原型链上，是返回`true`,否返回`false`;

#### 三、对象的constructor属性

除了instanceof，每个对象还有constructor的属性，利用它似乎也能进行Array的判断
```
var arr = [];
console.log(arr.constructor === Array);
```

#### 四、instanceof 和 constructor 的缺陷
通过使用 instanceof 和 constructor 来判断对象是不是数组，在大多数情况下是可以的，但实际上也还是有缺陷的，当页面上有多个`frame`,需要在多个iframe中来回切换时，这种方法的缺陷就表现出来啦。
> 由于每个iframe都有一套自己的执行环境，跨frame实例化的对象彼此是不共享原型链的，因此导致`instanceof`和`constructor`失效。
```
var iframe = document.createElement('iframe'); //创建iframe
document.body.appendChild(iframe); //添加到body中
xArray = window.frames[window.frames.length-1].Array;
var arr = new xArray(1,2,3); // 声明数组[1,2,3]
alert(arr instanceof Array); // false
alert(arr.constructor === Array); // false
```

#### 五、Object.prototype.toString
Object.prototype.toString的行为：首先，取得对象的一个内部属性[[Class]]，然后依据这个属性，返回一个类似于"[object Array]"的字符串作为结果(看过ECMA标准的应该都知道，[[]]用来表示语言内部用到的、外部不可直接访问的属性，称为“内部属性”)。利用这 个方法，再配合call，我们可以取得任何对象的内部属性[[Class]]，然后把类型检测转化为字符串比较，以达到我们的目的。

```
function isArrayFn (o) {
return Object.prototype.toString.call(o) === '[object Array]';
}
var arr = [1,2,3,1];
alert(isArrayFn(arr));// true
```

call改变toString的this引用为待检测的对象，返回此对象的字符串表示，然后对比此字符串是否是'[object Array]'，以判断其是否是Array的实例。为什么不直接o.toString()?嗯，虽然Array继承自Object，也会有 toString方法，但是这个方法有可能会被改写而达不到我们的要求，而Object.prototype则是老虎的屁股，很少有人敢去碰它的，所以能一定程度保证其“纯洁性”：)

JavaScript 标准文档中定义: [[Class]] 的值只可能是下面字符串中的一个： Arguments, Array, Boolean, Date, Error, Function, JSON, Math, Number, Object, RegExp, String.
这种方法在识别内置对象时往往十分有用，但对于自定义对象请不要使用这种方法。

#### 六、Array.isArray()
ECMAScript5中引入了Array.isArray()方法，用于专门判断一个对象是不是数组，是返回`true`，不是返回`false`;目前所有主流浏览器和IE9+都对其进行了支持，IE8及以下浏览器不支持该方法。

#### 七、一种最佳的判断方法
综合以上方法的优缺点，可以有一个最佳的判断数组的写法：
```
var arr = [1,2,3,1];
var arr2 = [{ abac : 1, abc : 2 }];
function isArrayFn(value){
    if (typeof Array.isArray === "function") {
        return Array.isArray(value);
    }else{
        return Object.prototype.toString.call(value) === "[object Array]";
    }
}
console.log(isArrayFn(arr));// true
console.log(isArrayFn(arr2));// true
```
