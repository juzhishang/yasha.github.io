---
title: 函数的name属性
date: 2018-08-31 11:23:57
tags:
 - es6
---

注：以下内容是我在阅读了《深入理解es6》《es6标准入门》《你不知道的JavaScript（下）》后所汇总整理，如有雷同存属笔记。

---

js有多种创建函数的方式：函数声明、函数表达式、Function构造器、bind方法。函数表达式又分具名和匿名。存取器、生成器、类都是函数。函数名可以是一个symbol值。

要区分上述不同的情况，函数的name值也就变复杂了。

下面罗列了几种`name`属性值的情况

+ 如果是直接给一个对象添加属性，属性值为匿名函数，该函数的`name`属性为空。如果是通过给一个对象赋值，对象中有一个属性是匿名函数，则该函数的`name`属性为属性名称
```js
var obj = {a: function() {}}
obj.a.name
// "a"
var obj = {}
obj.a = function() {}
obj.a.name
// ""

window.a = function() {}
a.name
// ""
```
（为什么都是对象，name值不一样，原因未知）
+ 如果是函数声明，`name`属性就是函数名
+ 如果是函数表达式，有函数名就取函数名，没有函数名就取被赋值为该匿名函数的变量的名称
+ 如果是引用，有函数名就取函数名，没有函数名就取被引用的那个函数的名称
```js
var a = function() {}
b = a
b.name
// "a"

var a = function fn(){}
a.name
// "fn"
var b = a
b.name
// "fn"
```
+ 如果是箭头函数，有赋值就取变量名，没有赋值就是''
```js
(() => {}).name
// ""
var fn = () => {}
fn.name
// "fn"
```
+ 如果是存取器，`name`属性值前面会带`set`或`get`
```js
var o = { 
  get foo(){}, 
  set foo(x){} 
}; 
var descriptor = Object.getOwnPropertyDescriptor(o, "foo"); 
descriptor.get.name; // "get foo"
descriptor.set.name; // "set foo";
```
+ 如果是通过`bind()`绑定的函数，会有前缀`bound`
```js
var a = function() {}
a.bind().name
// "bound a"
```
+ 如果是通过`Function`构造器创建的函数，`name`值是`anonymous`
```js
new Function().name
"anonymous"
```
+ 如果方法名是一个`Symbol`值，`name`属性返回的是`Symbol`的描述符：
```js
const key1 = Symbol('description');
const key2 = Symbol();
let obj = {
  [key1]() {},
  [key2]() {},
};
obj[key1].name // "[description]"
obj[key2].name // ""
```
+ 如果是类声明或类表达式，函数名就是类名
```js
class F {}
F.name
// "F"
var c = class F{}
c.name 
// "F"
```
+ 如果是生成器，函数名不带`*`，其他规则与普通函数没什么区别
+ 如果是`export default function() {}`默认导出，函数名是`default`


>注意：函数`name`属性的值不一定引用同名变量，它只是协助调试用的额外信息，所以不能使用`name`属性的值来获取对于函数的引用。

>其实，`name`这个属性早就被浏览器广泛支持，但是直到 ES6，才将其写入了标准。
但ES6 对这个属性的行为做出了一些修改。如果将一个匿名函数赋值给一个变量，ES5 的`name`属性，会返回空字符串，而 ES6 的`name`属性会返回实际的函数名。

```js
var f = function () {};

// ES5
f.name // ""

// ES6
f.name // "f"
```
##### 修改name属性

默认情况下，name属性不可改，但可配置，也就是说如果需要的话，可使用`Object.defineProperty()`来手动修改。
```js
var a = function() {}
a.name // "a"
Object.defineProperty(a, 'name', {
  configurable: true,
  value: 'b'
})
a.name // "b"
```
