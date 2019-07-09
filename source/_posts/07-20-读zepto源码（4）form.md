title: 读zepto源码 (4) form
date: 2016-07-20
tags: 
 - zepto
---


form模块比较简单，原型上增加了serializeArray，serialize，form三个方法。

值得说的就是form.submit事件，不知道小伙伴们有没有注意到，原生监听form事件，直接调用form元素的.submit()方法，并不能触发监听事件，只能触发表单提交事件。所以，zepto中的$.fn.submit()是手动去触发自定义事件，然后再调用form原生的.submit()方法的。

今天意外看到一篇文章，关于enter按钮提交表单背后的故事。引用下总结：

```
按下 Enter 时，如果  要触发 submit 事件，要符合两种情况之一：只有一个输入框（可以没有提交按钮），或者至少有一个提交按钮。"submit"> 和 <button> 都可以看成是提交按钮。jQuery 的 submit() API 也对此有所提及。  
  
达到第一条触发 submit 的条件后，不管是按提交按钮，还是按下 Enter ， 都会执行一样的流程：先触发按钮的 click 事件（如果有按钮），然后触发 submit 事件。这算是提交的标准流程。  
  
但如果提交按钮有多个， 会把第一个出现的按钮作为默认提交按钮，按下 Enter 触发 click 事件时，也只会触发这一个按钮的事件。想触发其他按钮的 click 事件，只能去手动点击。  

```

来源：[http://david-chen-blog.logdown.com/posts/177766-how-forms-submit-when-pressing-enter](http://david-chen-blog.logdown.com/posts/177766-how-forms-submit-when-pressing-enter)

```
//     Zepto.js  
//     (c) 2010-2016 Thomas Fuchs  
//     Zepto.js may be freely distributed under the MIT license.  
  
;(function($){  
  
 //调用此方法需满足，表单元素用form包含，每一个需要提交字段的表单都添加name属性  
 $.fn.serializeArray = function() {  
 var name, type, result = [],  
  
 add = function(value) {  
 //如果value为数组，遍历数组，调用add方法，添加到result中   
 //也就是说多层嵌套数组，会遍历内部数组的子元素，依次添加到result中：  
 //add([1,2,[3,4,[5,6]]])  => [{name:xxx,value:1},{name:xxx,value:2},{name:xxx,value:3}...]  
  
 //[{name:'xxx',value:'yyy'}]  
 if (value.forEach) return value.forEach(add)  
 //[{name:'xxx',value:{o:3}}]  
 result.push({ name: name, value: value })  
 }  
  
 //获取表单form，$.each遍历表单所有元素  
 if (this[0]) $.each(this[0].elements, function(_, field){  
 //获取其type值与name值  
 type = field.type, name = field.name  
  
 //如果name存在且当前表单不是fieldset且表单未被禁用且表单类型不为submit/reset/button/file  
 //且满足表单类型不为radio和checkbox或者满足表单的checked为true  
 //就是排除了field，submit,trset,button,file,和未选中的radio/checkbox，在提交表单时，这些表单的value不需要传  
 if (name && field.nodeName.toLowerCase() != 'fieldset' &&  
 !field.disabled && type != 'submit' && type != 'reset' && type != 'button' && type != 'file' &&  
 ((type != 'radio' && type != 'checkbox') || field.checked))  
 //获取元素的value值，调用add方法  
 add($(field).val())  
 })  
 return result  
 }  
  
 $.fn.serialize = function(){  
 var result = []  
 //遍历表单数据对象，对name和value进行encodeURIComponent后用=拼接转为字符串，  
 //push到数组result中，最后这些字符串用&符号转为字符串  
 this.serializeArray().forEach(function(elm){  
 result.push(encodeURIComponent(elm.name) + '=' + encodeURIComponent(elm.value))  
 })  
 return result.join('&')  
 }  
  
 $.fn.submit = function(callback) {  
 //如果参数大于1，元素绑定submit事件  
 if (0 in arguments) this.bind('submit', callback)  
 //如果参数未传，但当前集合的长度大于0  
 else if (this.length) {  
 //创建和初始化事件对象  
 var event = $.Event('submit')  
 //第一个元素触发submit事件，这里回调函数被执行了  
 this.eq(0).trigger(event)  
 //如果未阻止默认事件，第一个元素触发原生的submit事件，这里才是真正的提交表单  
 if (!event.isDefaultPrevented()) this.get(0).submit()  
 }  
 return this  
 }  
  
})(Zepto)  

```

