title: 探究clear:both
date: 2016-08-19
tags: 
 - css
---


小伙伴们必定不会对下面这段清除浮动的代码感到陌生：

```
  
.clearfix:after{   
 content: '';  
 display: block;  
 clear: both;  
 height: 0;  
 visibility: hidden;  
}  
*/  
.clearfix{ /*兼容 IE*/  
 zoom: 1;  
}  

```

今天，我们不谈BFC和IE6的haslayout，来探究clear:both这个看似不起眼的属性。

先看一个demo（非常规用法）：

```
  
<html>  
 <head>head>  
 <body>  
 <div class="wrap">  
 <div class="left">  
 我是左边，我比较高  
 div>  
 <div class="right">  
 我是右边,我比较矮  
 div>  
 div>  
 body>  
html>  

```

```
.wrap {  
 background-color: #edd;  
}  
  
.left {  
 float: left;  
 height:200px;  
 width:70%;  
 background-color:#dee;  
}  
.right {  
 background-color:#333;  
 color:#fff;  
}  
.right:after{  
 content:'';  
 display:block;  
 clear: both;  
}  

```

你以为它应该是这样的。。。

<iframe style="width: 100%; height: 220px" src="http://sandbox.runjs.cn/show/mkak3o8h" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

但实际上它是这样的。。。

<iframe style="width: 100%; height: 220px" src="http://sandbox.runjs.cn/show/03vjjz78" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### what-happend

在W3C中对[`clear:both`的解释](https://www.w3.org/TR/CSS2/visuren.html#propdef-clear)如下：

> both  
> Requires that the top border edge of the box be below the bottom outer edge of any right-floating and left-floating boxes that resulted from elements earlier in the source document.

大致意思是说，如果一个元素被添加了clear:both，它的上边框边缘应该在前面左浮动或者右浮动的元素的外边距的下面。

在这里。`.right`元素并未被指定为浮动，但是我们在`.right`的after伪元素上清除浮动，它带来了一些副作用：观察它的computed style，发现它的值是`200px`。就好像高度自动被填充了（设置`height:auto`也有此现象）。

##### 如何解决？？当然是把clearfix加到·wrap上啦。 "如何解决？？当然是把clearfix加到·wrap上啦。")如何解决？？当然是把clearfix加到`·wrap`上啦。

```
.wrap {  
 background-color: #edd;  
}  
  
.left {  
 float: left;  
 height:200px;  
 width:70%;  
 background-color:#dee;  
}  
.right {  
 background-color:#333;  
 color:#fff;  
    
}  
  
.wrap:after{  
 content:'';  
 display:block;  
 clear: both;  
 height:2px;  
 background-color:#f00;  
 width:100%;  
}  

```

可能有小伙伴说，可以为.right添加`overflow:hidden/auto`，或者指定一个具体的高度。我曾经在stackoverflow上也看到过这样的答案，那么，你要失望了，因为结果会是这样：

![指定overflow](http://ysha-01.oss-cn-shanghai.aliyuncs.com/QQ20160819073718.png)

或者是这样的

![指定height值](http://ysha-01.oss-cn-shanghai.aliyuncs.com/QQ20160819074532.png)

你还记得最初的目的是为了解决浮动么？╮(﹁_﹂)╭

好吧，这个例子告诉我们，清除浮动不是你想加就能加(⊙o⊙)… 所以最稳妥的办法还是把`clearfix`加在`.wrap`上。

细心的小伙伴会发现，上面两张图中伪元素的位置不一致啊。加了overflow:hidden的伪元素，它的上边框并没有在`.left`的下面。我们继续来看[W3C文档](https://www.w3.org/TR/CSS2/visuren.html#propdef-clear)：

> This property indicates which sides of an element’s box(es) may not be adjacent to an earlier floating box. The ‘clear’ property does not consider floats inside the element itself or in other block formatting contexts.

##### [](#原因在这里，块级格式化上下文！！！clear属性不会影响其它块级格式化上下文。

设置为`overflow:hidden;`的`.right`元素已经形成自己的BFC，不再影响`.wrap`。

众所周知，触发BFC的条件其中就包含了`overflow:hidden`。介于这篇文章BFC不是重点，我们就简单列下BFC的触发条件，到此结束吧^^。

触发BFC的条件：

*   float 除了none以外的值

*   overflow 除了visible 以外的值（hidden，auto，scroll ）

*   display (table-cell，table-caption，inline-block)

*   position（absolute，fixed）

*   fieldset元素

