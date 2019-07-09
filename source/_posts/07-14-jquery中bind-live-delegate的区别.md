title: jquery中bind-live-delegate的区别
date: 2016-07-14
tags: 
- jquery
---


曾经，2年前，面试时被问过这个问题，当时一脸懵逼的表示，我只用过on()啊。。。

好吧，这个梗是个引子而已。。。我记得，live方法是从1.7开始就过时了。。。日常开发中也很少会再遇到1.7以下的jquery了，毕竟很多时候连jquery也看不到了，它的时代已经慢慢在衰落。。。就当它是一个知识点或者，也许在老代码重构中会用到呢~

*   bind这个比较简单，它是直接绑定在目标元素上的，没有事件委托
*   live是通过冒泡的方式绑定元素，事件默认是绑定到document上的
*   delegate,也是事件委托

看起来，live和delegate在绑定到document时，是等价的。其实不然。事实上，delegate从性能上就要优于live。

举个例子：  
`$('.item').live('click',function(){})`与`$(document).delegate('.item','click',function(){})`,前者需要查找.item然后转为jquery对象,而后者只需要查找document。

live还有哪些坑点呢，参考下css88中jquery文档对live方法的说明：

```
因为更高版本的jQuery提供了更好的方法，没有.live()方法的缺点，所以.live()方法不再推荐使用。特别是，使用.live()出现的以下问题：  
  
+ 在调用 .live() 方法之前，jQuery会先获取与指定的选择器匹配的元素，这一点对于大型文档来说是很花费时间的。  
  
+ 不支持链式写法。例如，$("a").find(".offsite, .external").live( ... ); 这样的写法是不合法的，并不能像期待的那样起作用。  
  
+ 由于所有的 .live() 事件被添加到 document 元素上，所以在事件被处理之前，可能会通过最长最慢的那条路径之后才能被触发。  
  
+ 在事件处理中调用 event.stopPropagation() 来阻止事件处理被添加到 document 之后的节点中，是效率很低的。因为事件已经被传播到 document 上。  
  
+ 还有一点，live方法有一个非常大的缺点，那就是它仅能针对直接的CSS选择器做操作，这使得它变得非常的不灵活  
  
+ .live() 方法与其它事件方法的相互影响是会令人感到惊讶的。例如，$(document).unbind("click") 会移除所有通过 .live() 添加的 click 事件!  

```

``

对于仍在使用.live()的页面，那么下面关于该方法在不同版中的区别，可能会对您有一定帮助：

在 jQuery 1.7 之前，如果想阻止通过 .live() 绑定的事件被冒泡到其它元素上，必须在事件处理中返回 false。调用 .stopPropagation() 是不起作用的。  
从 jQuery 1.4 开始，.live() 方法支持自定义事件，也支持所有 JavaScript 事件冒泡。它还支持一些原本不能冒泡的事件，包括 change, submit, focus 和 blur。  
在 jQuery 1.3.x 中，只能绑定如下 JavaScript 事件：click, dblclick, keydown, keypress, keyup, mousedown, mousemove, mouseout, mouseover, 和 mouseup.

```

