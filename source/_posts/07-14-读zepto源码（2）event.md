title: 读zepto源码（2）event
date: 2016-07-14
tags: 
 - zepto
---


我之前对event这个对象了解的不多，所以zepto的事件机制看的时候还是很懵逼的。也参考了下一些文章：

*   [【zepto学习笔记03】事件机制](http://www.cnblogs.com/yexiaochai/p/3448500.html)
*   [Zepto源码分析-event模块](http://www.cnblogs.com/mominger/p/4384692.html)

废话不说，先看下这个模块的我认为比较重要的一些东西吧：

```
;(function($){  
 var _zid = 1,  
 handlers = {}  
  
 //计时器，返回zid  
 function zid(element) {}  
  
 //查找事件队列  
 function findHandlers(element, event, fn, selector) {}  
  
  
  
 function add(element, events, fn, data, selector, delegator, capture){}  
  
 function remove(element, events, fn, selector, capture){}  
  
  
  
 $.fn.trigger = function(event, args){}  
  
 $.Event = function(type, props) {}  
  
})(Zepto)  

```

我觉得从功能实现上看，它的核心可以分为三块内容：

1、是计数器和事件队列，他们的存在让每一个有事件绑定的元素（对象）都有了唯一的_zid标识，通过标识又把元素和事件队列联系了起来，因为所有的事件队列都是存储在handlers这个对象中的，通过标识_zid可以查找到每一个元素（对象）所对应的事件队列。_zid绑定在元素或者对象上，而不是zepto.Z实例上，因为$每次调用会产生新的实例，而dom元素或某个对象它是唯一的。

2、内部的add和remove方法，是事件绑定和解绑的核心。所有的绑定和解绑（on,off,live,die,delegate,undelegate,one），都是间接或直接调用了这两个方法。对外，add和remove方法通过$.event对象暴露出去，话说，我还没明白这样有什么用。。。findHandlers方法查找事件队列，它是通过元素的_zid,选择器，回调，以及事件类型和事件类型的命名空间来匹配查找结果的，每一个匹配的对象handler中存储了一个下标属性i，这样remove方法在根据_zid找到对应的队列后，能很方便的根据i移除队列中对应的handler对象。

3、就是手动触发事件的实现。$.Event()创建和初始化对象，$.trigger()负责触发。

最后，依旧放上完整的注释，event模块中，其它比较值得注意的就是live的坑点，还有就是哪些事件不支持冒泡。。。（zepto中就提到了focus,blur,mouseenter,mouseleave,此外还有change,submit），再有就是它扩展了event对象，使其拥有了stopPropagation()、preventDefault()和stopImmediatePropagation()。

```
  
//     Zepto.js  
//     (c) 2010-2016 Thomas Fuchs  
//     Zepto.js may be freely distributed under the MIT license.  
  
  
;(function($){  
 var _zid = 1, //用于计数  
 undefined,  
 slice = Array.prototype.slice,  
 isFunction = $.isFunction,  
 isString = function(obj){ return typeof obj == 'string' },  
 handlers = {},//存储事件对象，{1:[{...},{...}]}  
 specialEvents={},//存储特殊事件类型，在手动触发事件上用  
 //关于focus与focusin的区别：  
 //1,.focusin()指的是当一个元素，或者其内部任何一个元素获得焦点的时候会触发这个  
 //事件,因此我们可以在父元素上检测子元素获取焦点的情况。.focus()只在元素本身被触发。  
 //2，focus事件本身是不冒泡的，但是focusin可以，动态添加元素时，  
 //就不需重新绑定焦点事件，通过冒泡就能触发  
 focusinSupported = 'onfocusin' in window,//是否支持focusin事件  
 focus = { focus: 'focusin', blur: 'focusout' },//鼠标焦点相关事件，目的是做焦点修正  
 //鼠标移动相关事件，目的是做冒泡修正，因为mouseenter,mouseleave是不支持冒泡的  
 hover = { mouseenter: 'mouseover', mouseleave: 'mouseout' }  
  
 //click,mousedown,mouseup,mousemove都归为‘MouseEvent’  
 specialEvents.click = specialEvents.mousedown = specialEvents.mouseup = specialEvents.mousemove = 'MouseEvents'  
  
 //计时器，返回zid  
 function zid(element) {  
 //如果元素存在zid，返回zid，如果不存在，则会在element添加zid属性，再返回  
 return element._zid || (element._zid = _zid++)  
 }  
  
 //查找事件队列  
 function findHandlers(element, event, fn, selector) {  
 event = parse(event)//把event分隔成两部分  
 //如果存在命名空间，根据命名空间生成相应的正则  
 if (event.ns) var matcher = matcherFor(event.ns)  
 //zid(element)返回element._zid  
 //handlers[zid(element)]获取相应的事件队列，遍历事件队列，过滤出  
 return (handlers[zid(element)] || []).filter(function(handler) {  
 //handler存在  
 //且event.e不存在或者事件类型相同  
 //且event.ns不存在或者命名空间正则匹配  
 //且fn不存在或者fn的_zid相同  
 //且selector不存在或选择器相同  
 return handler  
 && (!event.e  || handler.e == event.e)  
 && (!event.ns || matcher.test(handler.ns))  
 && (!fn       || zid(handler.fn) === zid(fn))  
 && (!selector || handler.sel == selector)  
 })  
 }  
  
 //将event分割成事件类型和命名空间两部分  
 //click.me=>{e:click,ns:me}  
 function parse(event) {  
 //event用.分割成数组  
 var parts = ('' + event).split('.')  
 //属性e,存储event的类型  
 //属性ns,空间存储命名对剩余部分排序然后后空格分隔  
 return {e: parts[0], ns: parts.slice(1).sort().join(' ')}  
 }  
  
 //用于生成匹配的正则，多个事件类型之间有其他字符或者空格分隔  
 function matcherFor(ns) {  
 //ns中把空格替换为' .* ?'，含义是一个空格后面跟任意个任意字符，再跟0-1个空格。  
 //'(?:^| )'和'(?: |$)'就是以空字符串或者空格开头且结尾  
 return new RegExp('(?:^| )' + ns.replace(' ', ' .* ?') + '(?: |$)')  
 }  
  
 //事件捕获  
 //对focus和blur事件且浏览器不支持focusin focusout，通过设置捕获来模拟冒泡  
 function eventCapture(handler, captureSetting) {  
 //handler.del为真值且不支持focusin且handler.e为focus或blur  
 //或者captureSetting为真  
 //返回值为captureSetting的bool值，要不就是判断handler.e是否为focus或blur  
 return handler.del &&  
 (!focusinSupported && (handler.e in focus)) ||  
 !!captureSetting  
 }  
  
 //返回真实的事件类型，比如mouseenter，实际上，它是mouseover模拟的  
 //hover.mouseenter的值就是mouseover  
 //还有focus->focusIn, blur->focusOut,  mouseleave->mouseout  
 function realEvent(type) {  
 //返回hover[type]  
 //或者如果支持focusin，返回focus[type]  
 //或者返回type  
 return hover[type] || (focusinSupported && focus[type]) || type  
 }  
  
 function add(element, events, fn, data, selector, delegator, capture){  
 //获取zid  
 var id = zid(element),   
 //如果handlers[id]不存在，说明该元素没有注册过事件。  
 //handlers[id]设默认值为[]，就开辟了一个以_zid关联的新的事件队列。  
 //set=handlers[id]，//获取事件队列  
 set = (handlers[id] || (handlers[id] = []))  
 //event转为数组后遍历  
 events.split(/\s/).forEach(function(event){  
 //ready事件  
 if (event == 'ready') return $(document).ready(fn)  
  
 var handler   = parse(event)//分隔成两部分：handler.e，handler.ns  
 handler.fn    = fn//回调  
 handler.sel   = selector//选择器  
  
 // emulate mouseenter, mouseleave  
 //如果事件类型是mouseenter或者mouseleave，使用mouseover,mouseout来模拟  
 //因为在webkit中，对mouseenter和mouseleave的支持并不好  
 //这里又重写了fn  
 if (handler.e in hover) fn = function(e){  
 //对于mouseover和mouseout,e.relatedTarget用于获取事件的相关对象  
 var related = e.relatedTarget//鼠标从A移到B，relatedTarget是A，从B离开A，relatedTarget是B。  
 //related为假值说明不是mouseover,mouseout事件  
 //related不为this，这样可以排除掉从内部元素移动到外部元素的触发的情况  
 //this和related不是包含关系，说明鼠标事件不是在this这个元素或其子元素中发生的  
 if (!related || (related !== this && !$.contains(this, related)))  
 //调用handler.fn，调用域是this  
 return handler.fn.apply(this, arguments)  
 }  
  
 //事件委托  
 handler.del   = delegator  
 var callback  = delegator || fn  
 handler.proxy = function(e){  
 e = compatible(e)//扩展和兼容事件对象  
 //如果阻止事件冒泡且阻止了事件绑定，返回  
 if (e.isImmediatePropagationStopped()) return  
 e.data = data//缓存数据  
 //如果e._args未定义,第二个参数为[e]，否则就是与e._args合并后的[e]  
 var result = callback.apply(element, e._args == undefined ? [e] : [e].concat(e._args))  
 //如果返回值为false,阻止默认事件并阻止冒泡  
 if (result === false) e.preventDefault(), e.stopPropagation()  
 return result  
 }  
 //标记下标  
 handler.i = set.length  
 //添加到队列中  
 set.push(handler)  
 if ('addEventListener' in element)  
 //添加事件监听  
 element.addEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))  
 })  
 }  
  
 function remove(element, events, fn, selector, capture){  
 var id = zid(element)//获取元素element上的_zid的值  
 //event用空格分隔，遍历  
 ;(events || '').split(/\s/).forEach(function(event){  
 //遍历找到的事件队列  
 findHandlers(element, event, fn, selector).forEach(function(handler){  
 //倒序删除事件队列中的对象  
 delete handlers[id][handler.i]  
 if ('removeEventListener' in element)  
 //移除事件监听  
 element.removeEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))  
 })  
 })  
 }  
  
 //event对象，暴露了add和remove方法  
 $.event = { add: add, remove: remove }  
  
 //传入一个函数，返回一个上下文指向context的新函数  
 $.proxy = function(fn, context) {  
 //如果参数个数大于或等于3，将第3个及以后的参数存为args数组  
 var args = (2 in arguments) && slice.call(arguments, 2)  
 //如果fn是函数  
 if (isFunction(fn)) {  
 //如果args存在，将它与proxyFn的arguments合并成一个数组作为fn.apply的第二个参数  
 //proxyFn其实就是更改了上下文，然后参数增加了，内部还是会调用fn  
 var proxyFn = function(){ return fn.apply(context, args ? args.concat(slice.call(arguments)) : arguments) }  
 //函数上标记_zid属性  
 proxyFn._zid = zid(fn)  
 //返回代理函数  
 return proxyFn  
 //如果fn不是函数，而上下文是字符串  
 } else if (isString(context)) {  
 //如果有3个及以上的参数存在  
 if (args) {  
 //args头部增加fn[context], fn这两个元素  
 args.unshift(fn[context], fn)  
 //这里回过头又调用了$.proxy()，前面两个参数分别是fn[context], fn  
 //所以如果第二个参数是字符串，我们认为它的上下文是fn，函数是fn.context  
 return $.proxy.apply(null, args)  
 } else {  
 return $.proxy(fn[context], fn)  
 }  
 } else {  
 throw new TypeError("expected function")  
 }  
 }  
  
 //内部调用on  
 $.fn.bind = function(event, data, callback){  
 return this.on(event, data, callback)  
 }  
  
 //内部调用off  
 $.fn.unbind = function(event, callback){  
 return this.off(event, callback)  
 }  
  
 $.fn.one = function(event, selector, data, callback){  
 return this.on(event, selector, data, callback, 1)  
 }  
  
 var returnTrue = function(){return true},//函数，返回true  
 returnFalse = function(){return false},//函数，返回false  
  
 //以大写字母开头，以returnValue结束或者layerX结束或者layerY结束  
 ignoreProperties = /^([A-Z]|returnValue$|layer[XY]$)/,  
  
 //这三个方法所对应的标记函数，如果被执行，标记函数应返回true  
 //e.stopPropagation()和e.stopImmediatePropagation()都是用来阻止事件冒泡的。  
 //只是这两个方法有个区别，就是后面的方法不仅阻止了一个事件的冒泡，  
 //也把这个元素上的其他绑定事件也阻止了。  
 //而前者只是阻止一个绑定事件的冒泡，而不影响其他绑定事件执行。  
 //e.preventDefault()是用来阻止浏览器的默认行为的。  
 eventMethods = {  
 preventDefault: 'isDefaultPrevented',  
 stopImmediatePropagation: 'isImmediatePropagationStopped',  
 stopPropagation: 'isPropagationStopped'  
 }  
  
 //对event做了扩展和兼容  
 //event是代理对象，source是原始事件对象  
 function compatible(event, source) {  
 //什么情况下source存在，事件代理的时候  
 //如果source存在或者event的默认事件没有被阻止  
 //event.isDefaultPrevented用于判断是否调用了preventDefault方法  
 if (source || !event.isDefaultPrevented) {  
 //source如果不存在，则source = event  
 source || (source = event)  
  
 //遍历eventMethods对象  
 $.each(eventMethods, function(name, predicate) {  
 //缓存原生方法  
 var sourceMethod = source[name]  
 //重写这个方法  
 event[name] = function(){  
 //调用之后，相应的标志函数就会返回true  
 this[predicate] = returnTrue  
 //内部还是调用原生方法  
 return sourceMethod && sourceMethod.apply(source, arguments)  
 }  
 //扩展原生的事件对象event,遍历后它拥有了  
 //preventDefault,stopImmediatePropagation,stopPropagation  
 //这三个方法，默认都返回false  
 event[predicate] = returnFalse  
 })  
  
 //1如果浏览器支持e.defaultPrevented,判断e.defaultPrevented是否为真值  
 //2如果不支持，看e是否存在returnValue，如果存在，判断e.returnValue是否为假  
 //e.returnValue用于ie中判断是否阻止默认事件  
 //3如果e不存在returnValue，看e.getPreventDefault是否支持，如果支持看它的执行结果是否为真值  
 //e.getPreventDefault()是非标准方法，已经被e.defaultPrevented属性替代  
 if (source.defaultPrevented !== undefined ? source.defaultPrevented :  
 'returnValue' in source ? source.returnValue === false :  
 source.getPreventDefault && source.getPreventDefault())  
 //是否取消了默认事件  
 event.isDefaultPrevented = returnTrue  
 }  
 return event  
 }  
  
 //创建代理事件对象  
 function createProxy(event) {  
 //originalEvent,原始事件对象event  
 var key, proxy = { originalEvent: event }  
 for (key in event)  
 //如果event对象中的属性不匹配ignoreProperties正则的，复制给proxy对象  
 if (!ignoreProperties.test(key) && event[key] !== undefined)   
 proxy[key] = event[key]  
  
 return compatible(proxy, event)  
 }  
  
 $.fn.delegate = function(selector, event, callback){  
 return this.on(event, selector, callback)  
 }  
 $.fn.undelegate = function(selector, event, callback){  
 return this.off(event, selector, callback)  
 }  
  
 //zepto中live和die事件是绑定在document.body上的,内部调用delegate和undelegate  
 //而jquery是默认绑定在document上.  
 //把事件绑定在document.body上，如果我们设置了body的height为100%，将导致点击元素可能无法触发事件  
  
 $.fn.live = function(event, callback){  
 $(document.body).delegate(this.selector, event, callback)  
 return this  
 }  
 $.fn.die = function(event, callback){  
 $(document.body).undelegate(this.selector, event, callback)  
 return this  
 }  
  
 $.fn.on = function(event, selector, data, callback, one){  
 var autoRemove, delegator, $this = this  
 //event存在且不是字符串，监听多个事件  
 if (event && !isString(event)) {  
 //遍历event对象，添加事件绑定  
 $.each(event, function(type, fn){  
 $this.on(type, selector, data, fn, one)  
 })  
 return $this  
 }  
  
 //如果选择器不为字符串且回调不是函数且回调不为空  
 //认为选择器是未定义，即直接bind，  
 if (!isString(selector) && !isFunction(callback) && callback !== false)  
 callback = data, data = selector, selector = undefined  
  
 //如果callback不存在或者callback存在但data为false,认为data不存在  
 if (callback === undefined || data === false)  
 callback = data, data = undefined  
  
 if (callback === false) callback = returnFalse  
  
 return $this.each(function(_, element){  
 //如果只调用一次，定义autoRemove方法，用于先移除事件监听，再调用事件  
 if (one) autoRemove = function(e){  
 remove(element, e.type, callback)  
 return callback.apply(this, arguments)  
 }  
  
 //如果有选择器存在，说明是事件委托  
 if (selector) delegator = function(e){  
 //e.target为事件源  
 //从事件源开始查找离element最近的祖先元素selector  
 var evt, match = $(e.target).closest(selector, element).get(0)  
 //如果匹配到且不为本身  
 if (match && match !== element) {  
 //创建代理事件对象，继承currentTarget与liveFired属性后赋给evt  
 evt = $.extend(createProxy(e), {currentTarget: match, liveFired: element})  
 //优先选择调用autoRemove  
 return (autoRemove || callback).apply(match, [evt].concat(slice.call(arguments, 1)))  
 }  
 }  
  
 //内部调用add方法  
 add(element, event, callback, data, selector, delegator || autoRemove)  
 })  
 }  
  
 $.fn.off = function(event, selector, callback){  
 var $this = this  
 //多个事件  
 if (event && !isString(event)) {  
 $.each(event, function(type, fn){  
 $this.off(type, selector, fn)  
 })  
 return $this  
 }  
  
 //selector不存在  
 if (!isString(selector) && !isFunction(callback) && callback !== false)  
 callback = selector, selector = undefined  
  
 if (callback === false) callback = returnFalse  
  
 //遍历调用remove  
 return $this.each(function(){  
 remove(this, event, callback, selector)  
 })  
 }  
  
 //触发事件  
 $.fn.trigger = function(event, args){  
 //如果event为字符串或存对象  
 //event为$.Event(event)，否则event为compatible(event)  
 //这一步创建并初始化了事件对象  
 event = (isString(event) || $.isPlainObject(event)) ? $.Event(event) : compatible(event)  
    
 event._args = args  
 return this.each(function(){  
 // handle focus(), blur() by calling them directly  
 //如果是focus或者blur事件，可以直接调用  
 if (event.type in focus && typeof this[event.type] == "function") this[event.type]()  
 // items in the collection might not be DOM elements  
 //如果当前环境支持dispatchEvent，调用  
 else if ('dispatchEvent' in this) this.dispatchEvent(event)  
 //否则调用triggerHandler方法  
 else $(this).triggerHandler(event, args)  
 })  
 }  
  
 // triggers event handlers on current element just as if an event occurred,  
 // doesn't trigger an actual event, doesn't bubble  
 $.fn.triggerHandler = function(event, args){  
 var e, result  
 this.each(function(i, element){  
 //创建事件代理对象  
 //如果event是字符串，参数为$.Event(event)  
 //这里$.Event创建并初始化了一个事件对象，并扩展了一些阻止默认事件和冒泡的方法  
 e = createProxy(isString(event) ? $.Event(event) : event)  
 e._args = args  
 e.target = element//触发目标  
 //找到目标元素上的事件队列，遍历  
 $.each(findHandlers(element, event.type || event), function(i, handler){  
 //调用其代理的回调函数  
 result = handler.proxy(e)  
 if (e.isImmediatePropagationStopped()) return false  
 })  
 })  
 return result  
 }  
  
 // shortcut methods for `.bind(event, fn)` for each event type  
 //快捷方法，如果有参数，调用bind，无参数调用trigger  
 ;('focusin focusout focus blur load resize scroll unload click dblclick '+  
 'mousedown mouseup mousemove mouseover mouseout mouseenter mouseleave '+  
 'change select keydown keypress keyup error').split(' ').forEach(function(event) {  
 $.fn[event] = function(callback) {  
 return (0 in arguments) ?  
 this.bind(event, callback) :  
 this.trigger(event)  
 }  
 })  
  
 $.Event = function(type, props) {  
 //当type是个对象，props为type,type为props.type  
 if (!isString(type)) props = type, type = props.type  
  
 //如果type为click,mousedown,mouseup,mousemove,传参MouseEvents，否则传参Events  
 //冒泡设为true  
 //1、创建自定义事件document.createEvent  
 var event = document.createEvent(specialEvents[type] || 'Events'), bubbles = true  
    
 //遍历props,如果当前属性为bubbles，bubbles为props.bubbles的bool值，否则赋值当前属性到event对象中  
 if (props) for (var name in props) (name == 'bubbles') ? (bubbles = !!props[name]) : (event[name] = props[name])  
 //调用初始化事件对象  
 //2、初始化自定义事件  
 //initEvent()方法用于初始化通过DocumentEvent接口创建的Event的值  
 //添加一些方法  
 return compatible(event)  
 }  
  
})(Zepto)  

```

