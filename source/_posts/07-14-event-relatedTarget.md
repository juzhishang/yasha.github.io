title: event-relatedTarget
date: 2016-07-14
tags: 
 - 事件
---


来看下event对象的relatedTarget，它与mouseover、mouseout事件有关。

它的字面意思为相关目标，对mouseover事件而言，事件的主目标是获得光标的元素，而相关目标就是那个失去光标的元素。类似地，对mouseout事件而言，事件的主目标是失去光标的元素，而相关目标则是获得光标的元素。

举个例子，mouseover事件。当鼠标从A移到B,relatedTarget为A，而在IE中，并没有relatedTarget，而是用fromElement来表示。mouseout事件，当鼠标从A移到B，relatedTarget为B，在IE值，用toElement。

这个属性只对于mouseover和mouseout事件才有值；对于其他事件，这个属性的值是null。

在zepto中，这个属性被用来实现mouseenter和mouseleave。

