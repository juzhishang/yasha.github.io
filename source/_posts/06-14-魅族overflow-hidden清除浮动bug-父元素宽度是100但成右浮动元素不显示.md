title: 魅族overflow-hidden清除浮动bug-父元素宽度是100但成右浮动元素不显示
date: 2016-06-14
tags: 
  - 移动端
---


场景如下：（left子元素中有设position:relative;）

```
.wrap{  
 overflow:hidden;  
}  
  
.left{  
 width:10rem;  
 height:2.4rem;  
 float:left;  
}  
  
.right{  
 float:right;  
 width:2.4rem;  
 height:2.4rem;  
}  

```

```
<div class="wrap">  
 <div class="left">我是左边div>  
 <div class="right">我是右边div>  
div>  

```

由于wrap是div元素，默认占宽100%，子元素分别是左浮动和右浮动，由于我们的页面是按16等份分的，左右子元素总宽度小于16rem，正常来讲，right元素不应该被挤下去。但是在魅族中发现right元素不显示。

解决办法：wrap中增加position:relative;

原理：未知。猜测类似IE6的bug

```
当overflow:hidden在ie6 7下无效时 表明在父标签下必有子标签的属性为position:relative   
解决方法1 删除 子标签的 position:relative;  
解决方法2 为父标签增加  position:relative  

```

