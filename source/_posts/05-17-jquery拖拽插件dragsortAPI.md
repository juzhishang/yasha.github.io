title: jquery拖拽插件dragsortAPI
date: 2016-05-17
tags:
  - 移动端
---


#### API说明

dragsort中传入ops配置项，ops参数如下：

*   itemSelector:可被拖拽列表的css选择器，默认是’li’或者容器的第一个子元素标签。

*   dragSelector:CSS选择器内的元素的列表项的拖动手柄。默认是’li’或者容器的第一个子元素标签。

*   dragSelectorExclude:CSS选择器的元素内的dragSelector不会触发dragsort的。默认值是”input, textarea, a[href]”。

*   dragEnd:拖动结束后将被调用的回调函数.

*   dragBetween:设置为“true”，如果你要启用多组列表之间拖动选定的列表。 默认值是false。

*   placeHolderTemplate:拖动列表的占位符模板。默认值是`”<li>&nbsp;</li>”`或者根据容器的第一个子元素标签创建.

*   scrollContainer:CSS选择器的元素，作为滚动容器，例如溢出的div设置为自动。 默认值是“window“.

*   scrollSpeed:滚动速度，如果设置为”0″以禁用滚动。默认值是”5″.

#### example

```
$("ul").dragsort({  
 dragSelector: "li",  
 dragEnd: function() { },  
 dragBetween: false,  
 placeHolderTemplate: "&nbsp;"  
});  

```

#### 坑点：

1、如果初始容器内没有item，$(selector).dragsort(ops)执行后，动态创建的item无法拖拽。

2、如果初始容器内有item,$(selector).dragsort(ops)也已经执行。再次调用$(selector).dragsort(ops)item将无法拖拽。涉及到多组拖拽列表，动态添加item的情况，需要判断该容器中是否有item，是否被绑定过。容器的data-listidx属性可以帮我们判断这个问题。

3、$(selector).dragsort(ops)每次调用，都会把当前的selector重新排data-listidx，所以如果存在两个组,其中一个无item，$(selector).dragsort(ops)只添加了一个data-listidx，值为0。另一个在添加item后执行$(selector).dragsort(ops)，它的data-listidx值也为0。会存在重复idx的情况，这样的话，多组之间的拖拽必然是有问题的。

#### 源码解读

```
// jQuery List DragSort v0.5.2  
// Website: http://dragsort.codeplex.com/  
// License: http://dragsort.codeplex.com/license  
  
(function($) {  
  
 $.fn.dragsort = function(options) {  
 //如果选项是destroy  
 if (options == "destroy") {  
 //选择器触发dragsort-uninit  
 $(this.selector).trigger("dragsort-uninit");  
 return;  
 }  
 //获取配置  
 var opts = $.extend({}, $.fn.dragsort.defaults, options);  
 //列表数组初始化  
 var lists = [];  
 //列表，上一次位置  
 var list = null,  
 lastPos = null;  
  
 //index & 容器  
 this.each(function(i, cont) {  
 //如果容器是表格，且容器子元素为1，且子元素为tbody  
 //if list container is table, the browser automatically wraps rows in tbody if not specified so change list container to tbody so that children returns rows as user expected  
 if ($(cont).is("table") && $(cont).children().size() == 1 && $(cont).children().is("tbody"))  
 cont = $(cont).children().get(0); //容器改为tbody  
  
 var newList = {  
 draggedItem: null, //被拖拽元素  
 placeHolderItem: null, //占位元素  
 pos: null, //位置  
 offset: null, //偏移  
 offsetLimit: null, //最大偏移  
 scroll: null, //滚动  
 container: cont, //容器  
  
 init: function() {  
 //set options to default values if not set  
 //如果容器的子元素为0，设置opts。tagName为li,否则，获取容器第一个子元素的小写标签名，这里说明，容器必须是列表的直接父元素  
 opts.tagName = $(this.container).children().size() == 0 ? "li" : $(this.container).children().get(0).tagName.toLowerCase();  
 if (opts.itemSelector == "") //设置itemSelector  
 opts.itemSelector = opts.tagName;  
 if (opts.dragSelector == "") //设置dragSelector  
 opts.dragSelector = opts.tagName;  
 if (opts.placeHolderTemplate == "") //设置占位符模板，就是一个空的item  
 opts.placeHolderTemplate = " + opts.tagName + ">&nbsp; + opts.tagName + ">";  
  
 //listidx allows reference back to correct list variable instance  
 //为容器设置data-listidx的值为index  
 //鼠标按下时为被拖拽的元素绑定mousedown的回调，绑定dragsort-uninit方法的回调，回调函数是this.uninit  
 $(this.container).attr("data-listidx", i).mousedown(this.grabItem).bind("dragsort-uninit", this.uninit);  
 this.styleDragHandlers(true);  
 },  
  
 uninit: function() {  
 //获取容器data-listidx属性的index,以index为下标获取了lists数组中的list  
 var list = lists[$(this).attr("data-listidx")];  
 //容器解绑mousedown和dragsort-uninit之前绑定的回调  
 $(list.container).unbind("mousedown", list.grabItem).unbind("dragsort-uninit");  
 list.styleDragHandlers(false);  
 },  
 //获取容器中的子元素  
 getItems: function() {  
 return $(this.container).children(opts.itemSelector);  
 },  
 //设置拖拽元素的cursor值  
 styleDragHandlers: function(cursor) {  
 //获取容器中的子元素，map的作用是遍历后返回新数组  
 this.getItems().map(function() {  
 //如果当前item是配置中的可拖拽元素，返回当前item  
 //获取当前item下找到的可拖拽元素  
 return $(this).is(opts.dragSelector) ? this : $(this).find(opts.dragSelector).get();  
 })  
 .css("cursor", cursor ? "pointer" : ""); //返回的数组集合中，设置这些元素的css cursor值  
 },  
  
 grabItem: function(e) {  
 //获取容器data-listidx属性的index,以index为下标获取了lists数组中的list  
 var list = lists[$(this).attr("data-listidx")];  
 //获取点击元素的最近匹配的父容器中的第一个item  
 var item = $(e.target).closest("[data-listidx] > " + opts.tagName).get(0);  
 //list获取item集合，如果item中至少有一个等于list中的item，说明在可移动的item中  
 var insideMoveableItem = list.getItems().filter(function() {  
 return this == item;  
 }).size() > 0;  
  
 //if not left click or if clicked on excluded element (e.g. text box) or not a moveable list item return  
 //不是鼠标左击或者点击在排除元素或者它最近的父元素是排除元素或者不在不在可移动的列表里  
 if (e.which != 1 || $(e.target).is(opts.dragSelectorExclude) || $(e.target).closest(opts.dragSelectorExclude).size() > 0 || !insideMoveableItem)  
 return;  
  
 //prevents selection, stops issue on Fx where dragging hyperlink doesn't work and on IE where it triggers mousemove even though mouse hasn't moved,  
 //does also stop being able to click text boxes hence dragging on text boxes by default is disabled in dragSelectorExclude  
 e.preventDefault();  
  
 //change cursor to move while dragging  
 //拖拽元素为目标对象  
 var dragHandle = e.target;  
 //如果拖拽元素是配置中可拖拽元素  
 while (!$(dragHandle).is(opts.dragSelector)) {  
 //如果拖拽元素是容器，返回  
 if (dragHandle == this) return;  
 //设置拖拽元素为它的父节点  
 dragHandle = dragHandle.parentNode;  
 }  
 //设置拖拽元素的data-cursor属性为它的css属性cursor的值，所以是暂存原来的cursor属性  
 $(dragHandle).attr("data-cursor", $(dragHandle).css("cursor"));  
 //设置拖拽元素的cursor为move  
 $(dragHandle).css("cursor", "move");  
  
 //on mousedown wait for movement of mouse before triggering dragsort script (dragStart) to allow clicking of hyperlinks to work  
 var listElem = this; //这里this指向容器  
 //定义trigger方法  
 var trigger = function() {  
 list.dragStart.call(listElem, e); //调用dragStart方法，this指向listElem  
 $(list.container).unbind("mousemove", trigger); //容器的container解绑mousemove  
 };  
 //container mousemove时触发trigger方法，鼠标弹起时解绑trigger,拖拽元素回写原来的cursor属性  
 $(list.container).mousemove(trigger).mouseup(function() {  
 $(list.container).unbind("mousemove", trigger);  
 $(dragHandle).css("cursor", $(dragHandle).attr("data-cursor"));  
 });  
 },  
  
 dragStart: function(e) {  
 //这里的list是全局定义的那个数组  
 //如果list不为null 且 list.draggedItem不存在  
 if (list != null && list.draggedItem != null)  
 list.dropItem(); //调用list.dropItem方法  
  
 list = lists[$(this).attr("data-listidx")]; //获取当前容器  
 //设置draggedItem为事件目标的最近的父级item  
 list.draggedItem = $(e.target).closest("[data-listidx] > " + opts.tagName)  
  
 //record current position so on dragend we know if the dragged item changed position or not, not using getItems to allow dragsort to restore dragged item to original location in relation to fixed items  
 //设置其data-listidx为容器的data-listidx属性值+'-'+该item在容器中的index，比如0-1  
 list.draggedItem.attr("data-origpos", $(this).attr("data-listidx") + "-" + $(list.container).children().index(list.draggedItem));  
  
 //calculate mouse offset relative to draggedItem  
 ////获取marginTop与marginLeft值  
 var mt = parseInt(list.draggedItem.css("marginTop"));  
 var ml = parseInt(list.draggedItem.css("marginLeft"));  
 //设置鼠标相对于拖拽元素的offset  
 list.offset = list.draggedItem.offset();  
 list.offset.top = e.pageY - list.offset.top + (isNaN(mt) ? 0 : mt) - 1;  
 list.offset.left = e.pageX - list.offset.left + (isNaN(ml) ? 0 : ml) - 1;  
  
 //calculate box the dragged item can't be dragged outside of  
 //如果dragBetween存在  
 if (!opts.dragBetween) {  
 //如果容器的outerHeight为0，item的数量*item的outerWidth/容器的outerWidth获取行数，行数再去乘以item的outerHeight，不为0的话就直接返回容器的outerHeight  
 var containerHeight = $(list.container).outerHeight() == 0 ? Math.max(1, Math.round(0.5 + list.getItems().size() * list.draggedItem.outerWidth() / $(list.container).outerWidth())) * list.draggedItem.outerHeight() : $(list.container).outerHeight();  
 list.offsetLimit = $(list.container).offset(); //设置offset限制，不能拖拽超出容器  
 list.offsetLimit.right = list.offsetLimit.left + $(list.container).outerWidth() - list.draggedItem.outerWidth();  
 list.offsetLimit.bottom = list.offsetLimit.top + containerHeight - list.draggedItem.outerHeight();  
 }  
  
 //create placeholder item  
 //获取拖拽元素的高宽  
 var h = list.draggedItem.height();  
 var w = list.draggedItem.width();  
 //如果item为tr  
 if (opts.tagName == "tr") {  
 //遍历tr中的每个元素，设置宽度  
 list.draggedItem.children().each(function() {  
 $(this).width($(this).width());  
 });  
 //占位符元素为拖拽元素的复本并设置其data-placeholder属性为true  
 list.placeHolderItem = list.draggedItem.clone().attr("data-placeholder", true);  
 list.draggedItem.after(list.placeHolderItem); //在拖拽元素的后面一个位置插入占位符元素  
 //遍历占位符元素的每一个子元素，设置边框宽度为0，宽度+1高度+1，内容为0  
 list.placeHolderItem.children().each(function() {  
 $(this).css({  
 borderWidth: 0,  
 width: $(this).width() + 1,  
 height: $(this).height() + 1  
 }).html("&nbsp;");  
 });  
 } else { //如果不是tr  
 list.draggedItem.after(opts.placeHolderTemplate); //在拖拽元素的后面一个位置插入占位符模板  
 list.placeHolderItem = list.draggedItem.next().css({  
 height: h,  
 width: w  
 }).attr("data-placeholder", true); //设置占位符高宽及其data-placeholder属性为true  
 }  
  
 //如果item为td  
 if (opts.tagName == "td") {  
 var listTable = list.draggedItem.closest("table").get(0); //获取最近的父元素表格  
 //创建一个空表格，添加到body中，获取空表格的子元素也就是tbody，将被拖拽的元素append到tbody中  
 $(" + listTable.id + "" style="border-width: 0px;" class="dragSortItem " + listTable.className + "">").appendTo("body").children().append(list.draggedItem);  
 }  
  
 //style draggedItem while dragging  
 var orig = list.draggedItem.attr("style"); //获取拖拽元素的style属性值  
 list.draggedItem.attr("data-origstyle", orig ? orig : ""); //暂存到data-origstyle中  
 //设置拖拽元素为绝对定位，透明度0.8，z-index999，以及相应的高宽  
 list.draggedItem.css({  
 position: "absolute",  
 opacity: 0.8,  
 "z-index": 999,  
 height: h,  
 width: w  
 });  
  
 //auto-scroll setup  
 //设置容器的scroll对象，存储移动坐标和最大的坐标  
 list.scroll = {  
 moveX: 0,  
 moveY: 0,  
 maxX: $(document).width() - $(window).width(),  
 maxY: $(document).height() - $(window).height()  
 };  
 //设置定时器list.scroll.scrollY,间隔时间10毫秒  
 list.scroll.scrollY = window.setInterval(function() {  
 //如果容器不是window  
 if (opts.scrollContainer != window) {  
 //设置其容器的scrollTop为原来的scrollTop值+鼠标移动的Y值  
 $(opts.scrollContainer).scrollTop($(opts.scrollContainer).scrollTop() + list.scroll.moveY);  
 return;  
 }  
 var t = $(opts.scrollContainer).scrollTop(); //获取容器的scrollTop  
 //如果滚动距离Y大于0，并且容器的scrollTop小于maxY  
 //或者moveY小于0（向下移动？）且容器的scrollTop大于0、  
 //这是判断有没有滚出限制  
 if (list.scroll.moveY > 0 && t < list.scroll.maxY || list.scroll.moveY < 0 && t > 0) {  
 //设置容器的scrollTop为原有的+移动的  
 $(opts.scrollContainer).scrollTop(t + list.scroll.moveY);  
 //设置拖拽元素的top值为它的offset的top值+滚动Y值+1  
 list.draggedItem.css("top", list.draggedItem.offset().top + list.scroll.moveY + 1);  
 }  
 }, 10);  
 //处理同Y  
 list.scroll.scrollX = window.setInterval(function() {  
 if (opts.scrollContainer != window) {  
 $(opts.scrollContainer).scrollLeft($(opts.scrollContainer).scrollLeft() + list.scroll.moveX);  
 return;  
 }  
 var l = $(opts.scrollContainer).scrollLeft();  
 if (list.scroll.moveX > 0 && l < list.scroll.maxX || list.scroll.moveX < 0 && l > 0) {  
 $(opts.scrollContainer).scrollLeft(l + list.scroll.moveX);  
 list.draggedItem.css("left", list.draggedItem.offset().left + list.scroll.moveX + 1);  
 }  
 }, 10);  
  
 //misc  
 //遍历所有容器，调用容器的createDropTargets方法，调用容器的buildPositionTable方法，这里很奇怪，为什么要对其他的容器也遍历一遍  
 $(lists).each(function(i, l) {  
 l.createDropTargets();  
 l.buildPositionTable();  
 });  
 list.setPos(e.pageX, e.pageY); //设置被拖拽元素的位置  
 $(document).bind("mousemove", list.swapItems); //document绑定mousemove方法  
 $(document).bind("mouseup", list.dropItem); //mouseup时绑定dropItem方法  
 if (opts.scrollContainer != window) //如果滚动容器不是window  
 $(window).bind("wheel", list.wheel); //window绑定wheel方法  
 },  
  
 //set position of draggedItem  
 //设置被拖拽元素的位置  
 setPos: function(x, y) {  
 //remove mouse offset so mouse cursor remains in same place on draggedItem instead of top left corner  
 var top = y - this.offset.top; //鼠标的位置-被拖拽元素的offset值，就是鼠标在被拖拽元素中的位置  
 var left = x - this.offset.left;  
  
 //limit top, left to within box draggedItem can't be dragged outside of  
 if (!opts.dragBetween) { //如果不是dragBetween  
 top = Math.min(this.offsetLimit.bottom, Math.max(top, this.offsetLimit.top));  
 left = Math.min(this.offsetLimit.right, Math.max(left, this.offsetLimit.left));  
 }  
  
 //adjust top & left calculations to parent offset  
 //获取被拖拽元素的最近的非相对定位且不是body的父元素的offset  
 var parent = this.draggedItem.offsetParent().not("body").offset(); //offsetParent returns body even when it's static, if not static offset is only factoring margin  
 if (parent != null) { //如果parent存在  
 top -= parent.top;  
 left -= parent.left;  
 }  
  
 //set x or y auto-scroll amount  
 if (opts.scrollContainer == window) { //如果容器是window  
 y -= $(window).scrollTop();  
 x -= $(window).scrollLeft();  
 y = Math.max(0, y - $(window).height() + 5) + Math.min(0, y - 5);  
 x = Math.max(0, x - $(window).width() + 5) + Math.min(0, x - 5);  
 } else { //如果不是window  
 var cont = $(opts.scrollContainer);  
 var offset = cont.offset(); //获取容器的offset  
 //计算x,y  
 y = Math.max(0, y - cont.height() - offset.top) + Math.min(0, y - offset.top);  
 x = Math.max(0, x - cont.width() - offset.left) + Math.min(0, x - offset.left);  
 }  
 //如果x为0，moveX为0，如果不为0，moveX为x*滚动速度/x的绝对值  
 list.scroll.moveX = x == 0 ? 0 : x * opts.scrollSpeed / Math.abs(x);  
 list.scroll.moveY = y == 0 ? 0 : y * opts.scrollSpeed / Math.abs(y);  
  
 //move draggedItem to new mouse cursor location  
 //设置被拖拽元素的top值与left值  
 this.draggedItem.css({  
 top: top,  
 left: left  
 });  
 },  
  
 //if scroll container is a div allow mouse wheel to scroll div instead of window when mouse is hovering over  
 wheel: function(e) {  
 if (list && opts.scrollContainer != window) { //如果容器存在且滚动元素不是window  
 var cont = $(opts.scrollContainer); //缓存容器  
 var offset = cont.offset(); //获取容器的offset值  
 e = e.originalEvent;  
 //如果鼠标在容器中  
 if (e.clientX > offset.left && e.clientX < offset.left + cont.width() && e.clientY > offset.top && e.clientY < offset.top + cont.height()) {  
 var deltaY = (e.deltaMode == 0 ? 1 : 10) * e.deltaY; //计算步长？？  
 cont.scrollTop(cont.scrollTop() + deltaY); //滚动设置容器的scrollTop值  
 e.preventDefault();  
 }  
 }  
 },  
  
 //build a table recording all the positions of the moveable list items  
 //建表记录所有可移动item的位置  
 buildPositionTable: function() {  
 var pos = [];  
 //获取当前容器中所有的item中不是被拖拽和占位符的元素，遍历  
 this.getItems().not([list.draggedItem[0], list.placeHolderItem[0]]).each(function(i) {  
 var loc = $(this).offset(); //获取每个元素的offset()  
 //记录他们的位置和元素  
 loc.right = loc.left + $(this).outerWidth();  
 loc.bottom = loc.top + $(this).outerHeight();  
 loc.elm = this;  
 pos[i] = loc;  
 });  
 this.pos = pos;  
 },  
 //释放item?  
 dropItem: function() {  
 //如果被拖拽元素不存在，返回  
 if (list.draggedItem == null)  
 return;  
  
 //list.draggedItem.attr("style", "") doesn't work on IE8 and jQuery 1.5 or lower  
 //list.draggedItem.removeAttr("style") doesn't work on chrome and jQuery 1.6 (works jQuery 1.5 or lower)  
 //获取ata-origstyle属性  
 var orig = list.draggedItem.attr("data-origstyle");  
 //回写属性style  
 list.draggedItem.attr("style", orig);  
 //如果原属性为空  
 if (orig == "")  
 //移除style  
 list.draggedItem.removeAttr("style");  
 list.draggedItem.removeAttr("data-origstyle");  
  
 //设置拖拽元素的cursor值为pointer  
 list.styleDragHandlers(true);  
  
 //占位符前面添加被拖拽元素，然后移除占位符  
 list.placeHolderItem.before(list.draggedItem);  
 list.placeHolderItem.remove();  
  
 //移除什么鬼，没看明白  
 $("[data-droptarget], .dragSortItem").remove();  
  
 //清除浮动  
 window.clearInterval(list.scroll.scrollY);  
 window.clearInterval(list.scroll.scrollX);  
  
 //if position changed call dragEnd  
 //如果被拖拽元素的属性data-origpos不等于当前容器的idx-容器中被拖拽元素的index,说明位置变了  
 if (list.draggedItem.attr("data-origpos") != $(lists).index(list) + "-" + $(list.container).children().index(list.draggedItem))  
 if (opts.dragEnd.apply(list.draggedItem) == false) { //if dragEnd returns false revert order  
 var pos = list.draggedItem.attr("data-origpos").split('-'); //获取位置数组  
 var nextItem = $(lists[pos[0]].container).children().not(list.draggedItem).eq(pos[1]); //获取下一个item  
 if (nextItem.size() > 0) //如果下一个item存在  
 nextItem.before(list.draggedItem); //再它1前面插入被拖拽的元素  
 else if (pos[1] == 0) //was the only item in list//如果是唯一元素  
 $(lists[pos[0]].container).prepend(list.draggedItem);  
 else //was the last item in list  
 //如果是最后一个元素  
 $(lists[pos[0]].container).append(list.draggedItem);  
 }  
 //移除原位置属性  
 list.draggedItem.removeAttr("data-origpos");  
 //dragItem设为null  
 list.draggedItem = null;  
 //解绑事件  
 $(document).unbind("mousemove", list.swapItems);  
 $(document).unbind("mouseup", list.dropItem);  
 //如果滚动容器不为window,解绑wheel  
 if (opts.scrollContainer != window)  
 $(window).unbind("wheel", list.wheel);  
 return false;  
 },  
  
 //swap the draggedItem (represented visually by placeholder) with the list item the it has been dragged on top of  
 swapItems: function(e) {  
 if (list.draggedItem == null) //如果没有拖拽元素，返回  
 return false;  
  
 //move draggedItem to mouse location  
 list.setPos(e.pageX, e.pageY); //将拖拽元素移动到鼠标位置  
  
 //retrieve list and item position mouse cursor is over  
 var ei = list.findPos(e.pageX, e.pageY); //获取鼠标所处位置的item的index  
 var nlist = list;  
 //如果ei为-1（未找到）且dragBetween为true  
 for (var i = 0; ei == -1 && opts.dragBetween && i < lists.length; i++) {  
 //在其它容器中找  
 ei = lists[i].findPos(e.pageX, e.pageY);  
 nlist = lists[i];  
 }  
  
 //if not over another moveable list item return  
 //如果还是没有找到，返回  
 if (ei == -1)  
 return false;  
  
 //save fixed items locations  
 //返回鼠标所在item的容器的非被拖拽子元素  
 var children = function() {  
 return $(nlist.container).children().not(nlist.draggedItem);  
 };  
 //找到children中不是item的元素，设置index  
 var fixed = children().not(opts.itemSelector).each(function(i) {  
 this.idx = children().index(this);  
 });  
  
 //if moving draggedItem up or left place placeHolder before list item the dragged item is hovering over otherwise place it after  
 //如果向左或向上移动，新的item要加在占位符的前面  
 if (lastPos == null || lastPos.top > list.draggedItem.offset().top || lastPos.left > list.draggedItem.offset().left)  
 $(nlist.pos[ei].elm).before(list.placeHolderItem);  
 //否则加载占位符后面  
 else  
 $(nlist.pos[ei].elm).after(list.placeHolderItem);  
  
 //restore fixed items location  
 //重新存储固定定位元素的位置  
 fixed.each(function() {  
 var elm = children().eq(this.idx).get(0); //获取固定定位元素  
 if (this != elm && children().index(this) < this.idx) //如果当前元素不是固定元素并且非拖拽元素中this的下标小于idx属性  
 $(this).insertAfter(elm); //this添加到固定元素后面  
 else if (this != elm) //否则，添加在固定元素之前  
 $(this).insertBefore(elm);  
 });  
  
 //misc  
 //遍历所有容器，调用createDropTargets方法和buildPositionTable方法  
 $(lists).each(function(i, l) {  
 l.createDropTargets();  
 l.buildPositionTable();  
 });  
 lastPos = list.draggedItem.offset(); //设置最近一次的位置  
 return false;  
 },  
  
 //returns the index of the list item the mouse is over  
 //传入鼠标的位置  
 findPos: function(x, y) {  
 //遍历pos对象  
 for (var i = 0; i < this.pos.length; i++) {  
 //如果鼠标的位置在某个item内，返回这个item的index  
 if (this.pos[i].left < x && this.pos[i].right > x && this.pos[i].top < y && this.pos[i].bottom > y)  
 return i;  
 }  
 return -1;  
 },  
  
 //create drop targets which are placeholders at the end of other lists to allow dragging straight to the last position  
 createDropTargets: function() {  
 if (!opts.dragBetween) //如果不是dragBetween，返回.感觉between是用于跨容器拖拽  
 return;  
  
 $(lists).each(function() { //遍历所有的容器  
 var ph = $(this.container).find("[data-placeholder]"); //获取当前容器的占位符  
 var dt = $(this.container).find("[data-droptarget]"); //获取当前容器的droptarget，这个不知道是什么鬼  
 if (ph.size() > 0 && dt.size() > 0) //如果占位符存在droptarget也存在  
 dt.remove(); //移除droptarget  
 else if (ph.size() == 0 && dt.size() == 0) { //如果占位符和droptarget都不存在  
 if (opts.tagName == "td") //如果item是td  
 //设置占位符的data-droptarget属性然后添加到容器中  
 $(opts.placeHolderTemplate).attr("data-droptarget", true).appendTo(this.container);  
 else  
 //placeHolderItem移除data-placeholder后克隆然后设置data-droptarget为true然后添加到容器中  
 //list.placeHolderItem.clone().removeAttr("data-placeholder") crashes in IE7 and jquery 1.5.1 (doesn't in jquery 1.4.2 or IE8)  
 $(this.container).append(list.placeHolderItem.removeAttr("data-placeholder").clone().attr("data-droptarget", true));  
 //设置容器的placeHolderItem的data-placeholder属性为true  
 list.placeHolderItem.attr("data-placeholder", true);  
 }  
 });  
 }  
 };  
  
 newList.init(); //调用初始化方法  
 lists.push(newList);  
 });  
  
 return this;  
 };  
  
 $.fn.dragsort.defaults = {  
 itemSelector: "", //item选择器  
 dragSelector: "", //拖拽选择器  
 dragSelectorExclude: "input, textarea", //不能拖拽的元素  
 dragEnd: function() {}, //drag结束时回调  
 dragBetween: false,//容器之间是否可拖拽  
 placeHolderTemplate: "", //占位符模板  
 scrollContainer: window, //滚动容器  
 scrollSpeed: 5 //滚动速度（步长）  
 };  
  
})(jQuery);  

```

