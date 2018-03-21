---
title: js常见问题总结归纳
date: 2018-02-28 14:08:08
tags:
    - js
category:
    - js
---

#### 一、使用 typeof bar === "object" 来确定 bar 是否是对象的潜在陷阱是什么？如何避免这个陷阱？
> 首先`typeof bar === "object"`是检测`bar`是否是对象的可靠方法，但在javascript中`null`也被认为是对象，因此以下代码在控制台中将输出`true`:
```
var bar = null;
console.log(typeof bar == "object");
```
> 所以知道了`null`的问题，同时检测`bar`是否是`null`,就可以避免这一问题啦：
```
console.log((bar !== null) && (typeof bar === "object"));
```
> 当`bar`是一个数组的时候，例如，当`var bar = []`的时候；在很多情况下，这是期望行为，因为数字是真正的对象，但如果你想数组也返回`false`时，可以修改上述方案：
```
console.log((bar !== null) && (typeof bar === "object") && (toString.call(bar) !== "[object Array]"));
```
#### 二、下面的代码将输出什么到控制台？为什么？
```
(function(){
    var a = b = 3;
})();

console.log(typeof a == "undefined");
console.log(typeof b == "undefined");
```
> `var a = b = 3;`这句代码等同于：`b = 3; var a = b;`这时候变量`b`是不加`var`声明的变量，也就是全局变量，在函数内部没有`var`声明的变量是隐式全局变量。所以控制台打印的结果为：`true`和`false`。
> 如果再深入一步：在严格模式（使用`use strict`）,结果又会怎样？
> 在严格模式下运行的结果是：`报错:b is not undefined`;这正是严格模式的特点，避免不必要的`bug`(避免全局变量污染)。
#### 三、关于`this`的指向问题，以下代码将会输出什么？
```
var myObject = {
    foo:"bar",
    func:function(){
        var self = this;
        console.log(this.foo);
        console.log(self.foo);
        (function(){
            console.log(this.foo);
            console.log(self.foo);
        }());
    }
};
myObject.func();
```
> 在外部函数中，`this`和`self`都是指向`myObject`,所以两者都可以正确的引用和访问`foo`,在内部函数中，`this`不再指向`myObject`，结果是`this.foo`没有在内部函数中被定义。在`ECMA5`之前，在内部函数中的`this`将指向全局的`window`对象；反之，因为作为`ECMA5`，内部函数中的功能`this`是未定义的。所以控制台将输出：`bar ;bar ;undefined ;bar ;`
#### 四、`use strict`有什么意义和好处？
> `use strict` 是一种在`JavaScript`代码运行时自动实行更严格解析和错误处理的方法。
> 严格模式的一些主要优点包括：

1.使调试更加容易。那些被忽略或默默失败了的代码错误，会产生错误或抛出异常，因此尽早提醒你代码中的问题，你才能更快地指引到它们的源代码。

2.防止意外的全局变量。如果没有严格模式，将值分配给一个未声明的变量会自动创建该名称的全局变量。这是`JavaScript`中最常见的错误之一。在严格模式下，这样做的话会抛出错误。

3.消除` this `强制。如果没有严格模式，引用`null`或未定义的值到` this` 值会自动强制到全局变量。这可能会导致许多令人头痛的问题和让人恨不得拔自己头发的`bug`。在严格模式下，引用 `null`或未定义的` this `值会抛出错误。

4.不允许重复的属性名称或参数值。当检测到对象中重复命名的属性，例如：
`var object = {foo: "bar", foo: "baz"};`）
或检测到函数中重复命名的参数时,例如：
`function foo(val1, val2, val1){}`）
严格模式会抛出错误，因此捕捉几乎可以肯定是代码中的bug可以避免浪费大量的跟踪时间。

5.使` eval() `更安全。在严格模式和非严格模式下，` eval()` 的行为方式有所不同。最显而易见的是，在严格模式下，变量和声明在` eval()` 语句内部的函数不会在包含范围内创建（它们会在非严格模式下的包含范围中被创建，这也是一个常见的问题源）。

6.在` delete `使用无效时抛出错误。` delete` 操作符（用于从对象中删除属性）不能用在对象不可配置的属性上。当试图删除一个不可配置的属性时，非严格代码将默默地失败，而严格模式将在这样的情况下抛出异常。
#### 五、小心`javascript`自动插入分号机制？看下面的代码，它们会返回什么？
```
function foo1(){
    return {
        bar:"hello"
    };
}
function foo2(){
    return
        {
            bar:"hello"
        };
}
console.log("foo1 returns:");
console.log(foo1());
consoel.log("foo2 returns:");
console.log(foo2());
```
> 以上代码将会打印出：
```
foo1 returns:
Object {bar:"hello"}
foo2 returns:
undefined
```
> 原因是这样的，当碰到 `foo2()`中包含`return`语句的代码行（代码行上没有其他任何代码），分号会立即自动插入到返回语句之后。请仔细留意上面两个函数中`return`的不同之处，`foo2`函数的`return`是单独一行的。也不会抛出错误，因为代码的其余部分是完全有效的，即使它没有得到调用或做任何事情（相当于它就是是一个未使用的代码块，定义了等同于字符串` "hello"`的属性 `bar`）。所以，在使用`return`语句的时候，要留意`javascript`的这个特点，尽可能不要将`return`关键字写在独立的一行，避免不必造成不必要的错误。
> 在`《JavaScript语言精粹》`这本书里，这个“自动插入分号”机制被划入到了`JavaScript`的毒瘤里面，与之并列的前面的全局变量。
#### 六、`NaN `是什么？如何测试一个值是否等于` NaN `？
>` NaN `属性代表一个“不是数字”的值。这个特殊的值是因为运算不能执行而导致的，不能执行的原因要么是因为其中的运算对象之一非数字。例如：` "abc" / 4`，要么是因为运算的结果非数字。例如：除数为零。虽然` NaN` 意味着“不是数字”，但是它的类型，不管你信不信，是` Number`：
```
console.log(typeof NaN === "number")   // true
```
> 此外， NaN 和任何东西比较，甚至是它自己本身，结果是false：
```
console.log(NaN == NaN)   // false
```
> 一种半可靠的方法来测试一个数字是否等于` NaN`，是使用内置函数 `isNaN()`，但即使使用` isNaN()` 依然并非是一个完美的解决方案。
> 一个更好的解决办法是使用` value !== value`，如果值等于`NaN`，只会产生`true`。因为只有`NaN` 这货，才会自己不等于自己。
> 另外，`ES6`提供了一个新的` Number.isNaN()` 函数，这是一个不同的函数，并且比老的全局` isNaN() `函数更可靠。
#### 七、以下代码的运行结果是什么？
1.`console.log(1 + "2" + "2");   // "122";`

2.`console.log(1 + +"2" + "2");  // "32";`
> 根据运算的顺序，要执行的第一个运算是` +"2" `（第一个` "2" `前面的额外 `+ `被视为一元运算符），因此，`JavaScript`将` "2"` 的类型转换为数字，然后应用一元 + 号（即将其视为一个正数）。其结果就是得到一个数字` 2 `。

3.`console.log(1 + -"1" + "2");  // "02";`

4.`console.log(+"1" + "1" + "2");   // "112";`

5.`console.log("a" - "b" + "2");    // "NaN2";`

6.`console.log("a" - "b" + 2);     // NaN;`

#### 八、关于逻辑运算符，下面代码的运行结果是什么？
```
console.log( 0 || 1);   // 1;
console.log( 1 || 2);   // 1;
console.log( 0 && 1);   // 0;
console.log( 1 && 2);   // 2;
```
> 在`JavaScript`中， ` || ` 和 ` && ` 都是逻辑运算符，用于在从左至右计算时，返回第一个可完全确定的“逻辑值”。

#### 九、以下代码输出的结果是什么？为什么？
```
var a = {},
    b = {key:"b"},
    c = {key:"c"};
a[b] = 123;
a[c] = 456;
console.log(a[b]);
```
> JavaScript在设置对象的属性的时候，会暗中字符串化参数值；在以上代码中b和c都是对象，把它们设置为对象a的参数，它们都将被转换为 "[object Object]", 所以a[b]和a[c]都相当于a['object Object'];所以a[c]会将a[b]的值覆盖掉，因此，设置或引用 a[c] 和设置或引用 a[b] 完全相同。所以得到的答案是  456  ；

#### 十、说明以下代码的异同？
```
line-height:15px;
line-height:150%;
line-height:1.5;
line-height:1.5em;
```
> 行高为150%时，会根据父元素的字体大小先计算出行高值然后再让子元素继承。所以当line-height:150%时，子元素的行高等于`父元素的fontSize值 * 150% `;
> 当line-height:1.5em时，会根据父元素的字体大小先计算出行高值然后再让子元素继承。所以当line-height:1.5em时，子元素的行高等于`父元素的fontSize值 * 1.5em`;
> 当line-height:1.5时，会根据子元素的字体大小动态计算出行高值让子元素继承。所以，当line-height:1.5时，子元素行高等于`子元素的fontSize值 * 1.5 = 45px`;如果没有子元素，则以自身的fontSize值为准。
#### 十一、关于this指向，以下代码将输出什么？
```
var person = {
    _name: 'I am John',
    sayHello: function (){
        return this._name;
    }
};

var sayHello = person.sayHello;

console.log(sayHello());
console.log(person.sayHello());
```
代码运行的结果是：
```
undefined
I am John
```
> 在执行` sayHello() `的时候，当访问到` this._name `时，此时的`this`已经不再是` person ` 对象，而是全局窗口对象，也就是` widnow `对象。与此同时，` widnow `对象并不存在` _name  `属性，所以返回的是` undefined `。

#### 十二、关于变量提升，以下代码将输出什么？看下面的代码，输出的结果是什么？并解释你的答案。
```
function test() {
   console.log(a);
   console.log(foo());

   var a = 1;
   function foo() {
      return 2;
   }
}

test();//输出的结果是什么？
```
> 这段代码的执行结果是`undefined `和`2`。

原因是：js代码在执行之前都有预解析，预解析的时候`var`声明的变量和函数声明都被提升到了函数体的顶部。但是`var`声明的变量的提升只提升变量名不提升赋值。所以上面的代码相当于：

```
function test() {
   var a;//提升到执行环境顶部
   function foo() {
      return 2;
   }

   console.log(a);
   console.log(foo());

   a = 1;
}

test();
```
