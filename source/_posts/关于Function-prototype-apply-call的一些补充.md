title: 关于Function.prototype.apply.call的一些补充
date: 2015-12-09 16:09:45
tags:
 - apply
 - call
---
宿主对象，在javascript中有三类对象，本地对象，内置对象和宿主对象。其他两类暂且不提，宿主对象是指什么呢（DOM BOM），控制台对象是文档对象模型的扩展，也被认为是宿主对象。那么，它们有什么缺陷呢？在IE9之前，宿主对象不是继承自Object，它们的方法也不继承自Function，IE9之后就大有改进了。

  看下IE8与IE9的document.getElementById

ie8:
![](http://ww1.sinaimg.cn/large/66474ec2gw1eyth2sjtxlj207p02i3yn.jpg)

ie9:
![](http://ww4.sinaimg.cn/large/66474ec2jw1eyth977uafj20ar02bjrt.jpg)

我们可以看到，ie9的document.getElementById是有Function.prototype上的方法的，所以说，IE9+的宿主对象它们继承了Object，方法继承了Function。

IE8不支持call，所以问题就来了，我们经常会有这样的需求,比如，重新控制台。

很多人想到了console.log.call，但是它不完美，现在你们知道了~~

好，想想解决办法吧：

1、使用Function.prototype.bind，但是...你得为不支持bind的浏览器做兼容
```javascript
Function.prototype.bind.call(console.log,console).apply(console,arguments)
```
```javascript
//摘自MDN
if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      // closest thing possible to the ECMAScript 5 internal IsCallable function
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(this instanceof fNOP && oThis
                                 ? this
                                 : oThis || window,
                               aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```
2、new Function+replace (new Function不是说尽量不要用么，还是算了吧！！)
```javascript
function log(){

    var fn = 'console.log(args)',

    args=[].slice.call(arguments);

 

    fn = new Function('args',fn.replace(/args/,args.join(',')));

    fn(arguments);

};
```
3、终极方法
```javascript
Function.prototype.apply.call(console.log,console,arguments);
```
这么一对比，第三种方案妥妥的胜出啊，不用考虑兼容，代码简短，虽然不是很好理解~~

说了这么多废话，Function.prototype.apply.call什么时候用，就是在这种应用场景。

如果还有其他的话，那就是那些奇葩面试题，比如：
```javascript
var f1=function(){console.log(1)};
var f2=function(){console.log(2)};
Function.prototype.call.call(Function.prototype.call,f2)//2
Function.prototype.call.call(f1,f2);//1
```