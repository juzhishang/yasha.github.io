title: focus与focusin的区别
date: 2016-07-14
tags: 
 - 事件
---


1、.focusin()指的是当一个元素，或者其内部任何一个元素获得焦点的时候会触发这个事件,因此我们可以在父元素上检测子元素获取焦点的情况。.focus()只在元素本身被触发。

2、focus事件本身是不冒泡的，但是focusin可以，动态添加元素时，就不需重新绑定焦点事件，通过冒泡就能触发（相应的，blur不冒泡，focusout冒泡）

3、我们可以通过`'focusin' in window` 来判断当前浏览器是否支持focusin

_不支持冒泡的还有mouseenter和mouseleave_

