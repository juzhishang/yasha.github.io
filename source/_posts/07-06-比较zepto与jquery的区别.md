title: 比较zepto与jquery的区别
date: 2016-07-06
tags: 
 - zepto
 - jquery
---


*   1、zepto的$.each与$().each()

这两个方法不是同一个东西，$.each我们可以使用return false中断遍历，而$().each由于内部调用的是数组的every方法，即便return false还是会全部执行完。

*   2、zepto data 没有缓存

以前使用jQuery的时候，会尽量的去使用data方法，避免attr，因为jquey内部做了缓存。而zepto中的data方法竟然是内部调用了attr方法，只不过它在调用完后做了一次类型转换。这里还有一个要注意的点就是对于比较大的数值，应避免使用attr，防止在做类型转换的过程中数值的最后几位丢失。

*   3、zepto无法获取隐藏元素的width,height

对于隐藏元素，jquery可以获取其宽高，而zepto不可以。因为获取的是计算后的值，而zepto是通过调用元素的getBoundingClientRect()方法，当元素display为none时，getBoundingClientRect()返回的对象所有属性值都为0.

*   4、qsa

(1)对于选择器，zepto会对复杂的选择器直接使用querySelectorAll。而jquery中的sizzle虽然也是尽量使用querySelectorAll，但是它会用rbuggyQSA记录querySelectorAll的bug，没有匹配到兼容问题的才会使用querySelectorAll。

```
if ( support.qsa && (!rbuggyQSA || !rbuggyQSA.test( selector )) ) {   
 //newContext.querySelectorAll( newSelector )  
}  

```

querySelectorAll的参数需要符合规范，css中规定属性选择器中属性值应该是要加引号的：

```
http://www.w3.org/TR/css3-selectors/#attribute-selectors  
Represents an element with the att attribute whose value is exactly "val".  

```

所以不合规范的选择器，在jquery中可能能执行（因为rbuggyQSA做了过滤，sizzle做了兼容），而在zepto中是会报错的。

```
$('[data-pid=1234]')  
//zepto.min.js?v=6ec1e34eceb26faf0b345bf4b15cee4c:2 Uncaught DOMException: Failed to /execute 'querySelectorAll' on 'Document': '[data-pid=1234]' is not a valid selector.(…)  

```

```
$('[data-pid=1234]')  
//[]  

```

(2) 此外，如果zepto没有引入selector模块的话，zepto获取select元素的选中option不能用类似jQuery的方法$(‘option[selected]’),因为selected属性不是css的标准属性。

应该使用`$('option').not(function(){ return !this.selected })`

比如：  
jQuery:`$this.find('option[selected]').attr('data-v') * 1`

zepto:`$this.find('option').not(function() {return !this.selected}).attr('data-v') * 1`

但是获取有select中含有disabled属性的元素可以用 `$this.find("option:not(:disabled)")`因为disabled是标准属性。

参考网址：[https://github.com/madrobby/zepto/issues/503](https://github.com/madrobby/zepto/issues/503)

(3) jquery与zepto两者支持的css属性选择器是不同的：  
jquery:

```
[name|="value"]  
[name*="value"]  
[name~="value"]  
[name$="value"]  
[name="value"]  
[name!="value"  
[name^="value"]  
[name]  
[name="value"][name2="value2"]  

```

zepto由于它的复杂选择器需要符合css标准，是不支持[name|=”value”]这个的。

*   5、对宽度、高度的计算方式不同

首先zepto不支持innerWidth，innerHeight,outerWidth,outerHeight  
其次，zepto与jquery对于width的计算是不同的。zepto是由盒模型决定的，而jquery的width/height返回的是真实的高宽，不包含padding，border。

看个例子：

```
var box=document.createElement('div')  
document.body.appendChild(box)  
box.style.width='10px'  
box.style.padding='10px'  
box.style.border='10px soild'  
$(box).width()//30  
$(box).css('width')//10px  

```

而如果是jquery的话，$(box).width()的值应该是10。

*   6、offset的返回值

jquery返回width，height  
zepto还返回了top,left

*   7、动态创建dom

在将html字符串转为dom的时候，zepto支持指定该dom元素的一些属性，而jquery不支持。

参看zepto中fragment这个方法，它的第三个参数properties就是这么用的。

