title: 读zepto源码（1）zepto
date: 2016-07-06
tags: 
 - zepto
---


四月份的时候，部门开展了一次读源码活动。后来由于需求越来越多且架构调整，原来的小伙伴都被拆分到不同的项目组去了。zepto源码当时大家计划是读完核心代码、event、ajax…结果就是连核心代码的四分之一都没有读完…

好吧，不吐槽了，因为我自己也是断断续续的，一直到上周才读完核心的部分。

* * *

zepto号称是移动端的jquery，所以我是很想认真的把它读一遍，然后比较下它和jquery，当然这个目标实现起来有点困难╮(﹁_﹂)╭

不说废话了。

首先它定义了一个Zepto对象，通过window.Zepto=Zepto将Zepto绑定到全局window下。  
在window.$不存在的时候，才会把Zepto绑定到window.$下，我想，这是为了给jQuery让路吧。

```
var Zepto=(function(){//...})()  
window.Zepto = Zepto  
window.$ === undefined && (window.$ = Zepto)  

```

Zepto是一个匿名自执行函数，它的内部环境中，我觉得最重要的三个对象是：zepto、$、Z。

### Z是一个构造器

```
 function Z(dom, selector) {  
 //这里添加的都是对象自身的属性  
 //dom存在，len为dom的length，否则len为0  
 var i, len = dom ? dom.length : 0  
 //添加数字属性，存DOM元素储  
 for (i = 0; i < len; i++) this[i] = dom[i]  
 //添加length与selector属性  
 this.length = len  
 this.selector = selector || ''  
}  

```

### zepto是核心，看一下我觉得比较关键的几个方法：

*   zepto.Z:返回Z的实例
*   zepto.isZ:判断继承关系
*   zepto.init:初始化，内部调用了zepto.Z

```
 zepto.Z = function(dom, selector) {  
 //返回Z的实例  
 return new Z(dom, selector)  
}  
  
 // `$.zepto.isZ` should return `true` if the given object is a Zepto  
 // collection. This method can be overridden in plugins.  
 //http://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/  
 zepto.isZ = function(object) {  
 //这里用zepto.Z而不是Z，大概是因为zepto.Z可以被重新吧  
 //判断object是否是zepto.Z的实例,看它的原型链，是否是zepto.z.prototype  
 //object.__proto__ === zepto.Z.prototype↘  
 //object.__proto__.__proto__===zepto.Z.prototype  
 return object instanceof zepto.Z  
}  
  
 // `$.zepto.init` is Zepto's counterpart to jQuery's `$.fn.init` and  
 // takes a CSS selector and an optional context (and handles various  
 // special cases).  
 // This method can be overridden in plugins.  
 zepto.init = function(selector, context) {  
 //...  
 //返回Z的实例  
 return zepto.Z(dom, selector)  
}  

```

这里很有意思，命名zepto.Z执行的是new Z()，为什么zepto.isZ判断继承关系是用zepto.Z而不是Z呢，关键在一句`zepto.Z.prototype = Z.prototype = $.fn`，我们先不去看$.fn是什么鬼，这里zepto.Z.prototype = Z.prototype就是原因。因为instanceof判断继承关系实际上看的就是原型链。

### 是这个匿名自执行函数对外暴露的一个接口。

$它自身是一个函数。

```
//返回zepto.init的调用结果  
$ = function(selector, context) {  
 return zepto.init(selector, context)  
}  

```

这样，我们调用$()就能拿到zepto.Z的实例。

$上又有很多属性和方法，这里不一一列举。我们直接通过$.xxx调用，而不需要生成zepto.Z的实例。所以，这可以认为是一些工具方法和全局属性。

```
Object.keys($)  
/*["extend", "contains", "type", "isFunction", "isWindow", "isArray", "isPlainObject", "isEmptyObject", "inArray", "camelCase", "trim", "uuid", "support", "expr", "noop", "map", "each", "grep", "parseJSON", "fn", "zepto", "event", "proxy", "Event", "active", "ajaxJSONP", "ajaxSettings", "ajax", "get", "post", "getJSON", "param", "expando"] */  

```

其中$有一个fn对象，这个fn对象上也挂了很多属性和方法。我们发现，$.fn实际上就是zepto.Z的实例的原型，为什么呢，还有由于这句`zepto.Z.prototype = Z.prototype = $.fn`，它重写了zepto.Z和Z的prototype。

而`$.zepto = zepto`这句，又把核心对象zepto暴露了出来。

### 最后，放下核心代码的注释：

```
//     Zepto.js  
//     (c) 2010-2016 Thomas Fuchs  
//     Zepto.js may be freely distributed under the MIT license.  
//核心，返回Zepto集合  
var Zepto = (function() {  
 /*  
 定义变量  
 */  
 var undefined, //未定义  
 key,  
 $, //DOM操作对象，重要  
 classList,//存储className  
 emptyArray = [], //空数组  
 concat = emptyArray.concat,  
 filter = emptyArray.filter,  
 slice = emptyArray.slice,  
 document = window.document, //缓存document  
 elementDisplay = {}, //存储各个标签的默认display值  
 classCache = {}, //缓存class  
 cssNumber = { //这些css属性赋值时不自动加px  
 'column-count': 1,  
 'columns': 1,  
 'font-weight': 1,  
 'line-height': 1,  
 'opacity': 1,  
 'z-index': 1,  
 'zoom': 1  
 },  
 fragmentRE = /^\s*]*>/, //html片段  
 singleTagRE = /^(?:|)$/, //标签  
 //非闭合标签的闭合写法，虽然说法不太准确  
 //这里[\w:]中冒号的情况比如'svg:svg'  
 //  
 //    
 //  
 tagExpanderRE = /]*)\/>/ig,  
 rootNodeRE = /^(?:body|html)$/i, //根节点  
 capitalRE = /([A-Z])/g, //大写  
  
 // special attributes that should be get/set via method calls  
 //这里是方法名称数组，它们通过遍历调用的方式设置的，调用时取值和  
 methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'],  
  
 //dom插入操作名称  
 adjacencyOperators = ['after', 'prepend', 'before', 'append'],//其实它跟jquery一样也有appendTo,prependTo,insertAfter,insertBefore  
 table = document.createElement('table'), //创建table标签  
 tableRow = document.createElement('tr'), //创建tr标签  
 containers = { //父级标签  
 'tr': document.createElement('tbody'),  
 'tbody': table,  
 'thead': table,  
 'tfoot': table,  
 'td': tableRow,  
 'th': tableRow,  
 '*': document.createElement('div')  
 },  
 readyRE = /complete|loaded|interactive/, //请求ready状态的正则  
 simpleSelectorRE = /^[\w-]*$/, //匹配简单的class  
 class2type = {}, //这里有一个内置对象的内部属性[[class]]的概念，这个对象用来做class值到类型的映射  
 toString = class2type.toString, //Object原型上的toString方法  
 zepto = {},//很重要  
 camelize, //驼峰  
 uniq, //唯一  
 tempParent = document.createElement('div'), //临时父元素，创建一个div标签  
 propMap = { //标签属性映射  
 'tabindex': 'tabIndex',  
 'readonly': 'readOnly',  
 'for': 'htmlFor',  
 'class': 'className',  
 'maxlength': 'maxLength',  
 'cellspacing': 'cellSpacing',  
 'cellpadding': 'cellPadding',  
 'rowspan': 'rowSpan',  
 'colspan': 'colSpan',  
 'usemap': 'useMap',  
 'frameborder': 'frameBorder',  
 'contenteditable': 'contentEditable'  
 },  
 isArray = Array.isArray ||function(object) {  
 return object instanceof Array  
 } //是否是数组  
  
 zepto.matches = function(element, selector) {  
 //如果选择器为假或者元素为假或者元素的节点类型并非element，返回  
 if (!selector || !element || element.nodeType !== 1) return false  
 //matchesSelector用来匹配某个元素是否匹配某个选择器   
 //这里并没有支持IE的兼容？  
 var matchesSelector = element.webkitMatchesSelector || element.mozMatchesSelector ||  
 element.oMatchesSelector || element.matchesSelector  
 //如果支持，返回匹配结果  
 if (matchesSelector) return matchesSelector.call(element, selector)  
 // fall back to performing a selector:  
 var match,  
 parent = element.parentNode, //获取父节点  
 temp = !parent //若无父节点，则判断为临时DOM  
 //若为临时，parent指向临时父元素tempParent,parent添加element子元素  
 if (temp)(parent = tempParent).appendChild(element)  
 //zepto.qsa类似querySellectAll，这里qsa返回一个数组结果，我们查找返回结果中是否存在该元素  
 //以判断是否匹配。位运算~这里的作用注意是针对-1这个值，~-1的结果为0，假值。  
 match = ~zepto.qsa(parent, selector).indexOf(element)  
 temp && tempParent.removeChild(element) //匹配完后移除临时父元素中的子元素  
 return match //返回匹配结果  
 }  
  
 function type(obj) {  
 //obj == null的两种情况：1：null，2：undefined  
 //当String作为函数调用时，它执行的是类型转换，内部调用toString,规则如下：  
 //http://www.ecma-international.org/ecma-262/6.0/index.html#sec-tostring  
 //Object.prototype.toString.call(null) ==> "[object Null]"  
 //Object.prototype.toString.call(undefined) ==> "[object Undefined]"  
 return obj == null ? String(obj) :  
 class2type[toString.call(obj)] || "object" //非内置对象，类型默认为‘object’  
 }  
  
 //是否是数组  
 function isFunction(value) {  
 return type(value) == "function"  
 }   
  
 //是否为window,window.window属性指向自身  
 function isWindow(obj) {  
 return obj != null && obj == obj.window  
 }   
 //是否为document  
 function isDocument(obj) {  
 return obj != null && obj.nodeType == obj.DOCUMENT_NODE  
 }   
 //是否为对象  
 function isObject(obj) {  
 return type(obj) == "object"  
 }   
 //纯粹的对象，通过{}或者new Object创建。  
 function isPlainObject(obj) {   
 return isObject(obj) && !isWindow(obj) && Object.getPrototypeOf(obj) == Object.prototype  
 }  
  
 //类数组，长度属性为数值  
 //bug，未考虑function的情况  
 function likeArray(obj) {  
 return typeof obj.length == 'number'  
 }   
  
 //返回过滤掉null和undefined后的数组  
 function compact(array) {  
 return filter.call(array, function(item) {  
 return item != null  
 })  
 }   
  
 //数组扁平化，如果数组长度大于0，调用$.fn.concat.apply方法，否则返回数组  
 //只能做到二级数组扁平  
 function flatten(array) {  
 return array.length > 0 ? $.fn.concat.apply([], array) : array  
 }  
  
 //a-b转为aB，中划线可以有多个  
 camelize = function(str) {  
 return str.replace(/-+(.)?/g, function(match, chr) {  
 return chr ? chr.toUpperCase() : ''  
 })  
 }  
  
 //abC -> a-bc  AbC -> ab-c  
 //可以认为是camelize的反向  
 function dasherize(str) {  
 return str.replace(/::/g, '/') //::替换成/  
 .replace(/([A-Z]+)([A-Z][a-z])/g, '$1_$2') //连续的大写字母与大写小写之间使用_分隔，AAb->A_Ab  
 .replace(/([a-z\d])([A-Z])/g, '$1_$2') //数字或小写字母与大写字母之间使用_分隔，1A->1_A  
 .replace(/_/g, '-') //_替换为-  
 .toLowerCase() //转为小写  
 }   
  
 //数组去重，赞~通过比较当前下标和array.indexOf(item)下标是否相等  
 uniq = function(array) {  
 return filter.call(array, function(item, idx) {  
 return array.indexOf(item) == idx  
 });  
 }  
  
 //返回classCache中的name  
 function classRE(name) {  
 //返回name的正则，先在classCache中去取，如果不在classCache中，创建正则并存储到classCache中  
 //这里的正则匹配所有name以空字符串或者一个空格开头且以一个空字符串或空格结束  
 return name in classCache ?  
 classCache[name] : (classCache[name] = new RegExp('(^|\\s)' + name + '(\\s|$)'))  
 }  
  
 //为css属性值添加px后缀  
 function maybeAddPx(name, value) {  
 //value为数值类型，并且name不在cssNumber中，返回value+px后缀  
 return (typeof value == "number" && !cssNumber[dasherize(name)]) ? value + "px" : value  
 }  
  
 //http://www.zhangxinxu.com/wordpress/2012/05/getcomputedstyle-js-getpropertyvalue-currentstyle/  
 //获取默认的display值  
 function defaultDisplay(nodeName) {  
 var element, display  
 if (!elementDisplay[nodeName]) { //如果没有在缓存中找到  
 element = document.createElement(nodeName) //创建该标签  
 document.body.appendChild(element) //添加到body中  
 //为什么使用getPropertyValue，getPropertyValue中传的参数是css样式里的书写格式，考虑float与margin-left  
 display = getComputedStyle(element, '').getPropertyValue("display") //获取默认的display值  
 element.parentNode.removeChild(element) //移除该标签  
 //head,meta,link,script,style  
 display == "none" && (display = "block") //如果display为none,设置display为block  
 elementDisplay[nodeName] = display //加入缓存  
 }  
 return elementDisplay[nodeName] //返回display  
 }  
  
 function children(element) {  
 return 'children' in element ? //如果element有children属性  
 //将类数组转为数组，创建多少长度的数组取决于参数的length属性  
 slice.call(element.children) :  
 //如果没有length属性，遍历childNodes  
 //这里$.map返回nodeType为1的节点，即元素节点  
 $.map(element.childNodes, function(node) {  
 if (node.nodeType == 1) return node  
 })  
 }  
  
 // 构造器，返回的是zepto数组集合  
 function Z(dom, selector) {  
 //这里添加的都是对象自身的属性  
 //dom存在，len为dom的length，否则len为0  
 var i, len = dom ? dom.length : 0  
 //添加数字属性，存DOM元素储  
 for (i = 0; i < len; i++) this[i] = dom[i]  
 //添加length与selector属性  
 this.length = len  
 this.selector = selector || ''  
 }  
  
 /*  
 绑定zepto上的静态方法  
 */  
 // `$.zepto.fragment` takes a html string and an optional tag name  
 // to generate DOM nodes from the given html string.  
 // The generated DOM nodes are returned as an array.  
 // This function can be overridden in plugins for example to make  
 // it compatible with browsers that don't support the DOM fully.  
 // 将html片段转为Dom数组  
 zepto.fragment = function(html, name, properties) {  
 var dom, nodes, container  
  
 // A special case optimization for a single tag  
 //如果html片段匹配单标签，创建dom标签  
 if (singleTagRE.test(html)) dom = $(document.createElement(RegExp.$1))  
 if (!dom) {  
 //如果html是字符串,替换非闭合标签的闭合写法''->''  
 if (html.replace) html = html.replace(tagExpanderRE, "")  
 //如果name不存在，取fragmentRE匹配的最外面的标签  
 if (name === undefined) name = fragmentRE.test(html) && RegExp.$1  
 //如果name不在containers中（不是表格相关的标签），设为*，*的父级容器为div  
 if (!(name in containers)) name = '*'  
 container = containers[name] //获取父级容器  
 container.innerHTML = '' + html //填充内容  
  
 //获取引用后清空container  
 dom = $.each(slice.call(container.childNodes), function() {  
 //移除container子元素  
 container.removeChild(this)  
  
 })  
 }  
 //如果properties是纯对象  
 if (isPlainObject(properties)) {  
 nodes = $(dom) //转为zepto集合  
 $.each(properties, function(key, value) { //遍历properties  
 //如果key在methodAttributes数组中  
 if (methodAttributes.indexOf(key) > -1) nodes[key](value) //$(selector).val(value)  
 else nodes.attr(key, value) //使用attr方式设值  
 })  
 }  
  
 return dom  
 }  
  
 // `$.zepto.Z` swaps out the prototype of the given `dom` array  
 // of nodes with `$.fn` and thus supplying all the Zepto functions  
 // to the array. This method can be overridden in plugins.  
 zepto.Z = function(dom, selector) {  
 //返回Z的实例  
 return new Z(dom, selector)  
 }  
  
 // `$.zepto.isZ` should return `true` if the given object is a Zepto  
 // collection. This method can be overridden in plugins.  
 //http://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/  
 zepto.isZ = function(object) {  
 //这里用zepto.Z而不是Z，大概是因为zepto.Z可以被重新吧  
 //判断object是否是zepto.Z的实例,看它的原型链，是否是zepto.z.prototype  
 //object.__proto__ === zepto.Z.prototype↘  
 //object.__proto__.__proto__===zepto.Z.prototype  
 return object instanceof zepto.Z  
 }  
  
 // `$.zepto.init` is Zepto's counterpart to jQuery's `$.fn.init` and  
 // takes a CSS selector and an optional context (and handles various  
 // special cases).  
 // This method can be overridden in plugins.  
 //如果是创建DOM或者数组包裹对象，selector为null  
 zepto.init = function(selector, context) {  
 var dom  
 // If nothing given, return an empty Zepto collection  
 //如果选择器没有传，返回一个空的zepto实例  
 if (!selector) return zepto.Z()  
 // Optimize for string selectors  
 else if (typeof selector == 'string') { //如果选择器是字符串  
 selector = selector.trim() //去两端空格  
 // If it's a html fragment, create nodes from it  
 // Note: In both Chrome 21 and Firefox 15, DOM error 12  
 // is thrown if the fragment doesn't begin with <  
 if (selector[0] == ' && fragmentRE.test(selector)) //如果字符串的第一个字符是  
 dom = zepto.fragment(selector, RegExp.$1, context), selector = null //创建dom标签  
 // If there's a context, create a collection on that context first, and select  
 // nodes from there  
 //如果有上下文，先创建一个zepto集合，在集合中找  
 else if (context !== undefined) return $(context).find(selector)  
 // If it's a CSS selector, use it to select nodes.  
 //如果是选择器，使用zepto.qsa  
 else dom = zepto.qsa(document, selector)  
 }  
 // If a function is given, call it when the DOM is ready  
 //如果选择器是函数，DOM ready后调用  
 else if (isFunction(selector)) return $(document).ready(selector)  
 // If a Zepto collection is given, just return it  
 //如果selector是zepto.Z实例，直接返回selector  
 else if (zepto.isZ(selector)) return selector  
 else {  
 // normalize array if an array of nodes is given  
 //如果selector是数组，过滤掉undefined和null  
 if (isArray(selector)) dom = compact(selector)  
 // Wrap DOM nodes.  
  
 else if (isObject(selector)) //如果是对象  
 dom = [selector], selector = null //数组包裹对象  
 // If it's a html fragment, create nodes from it  
 else if (fragmentRE.test(selector)) //如果是html字符串，创建dom数组  
 dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null  
 // If there's a context, create a collection on that context first, and select  
 // nodes from there  
 //如果上下文不为空，先创建一个集合，在集合中找  
 else if (context !== undefined) return $(context).find(selector)  
 // And last but no least, if it's a CSS selector, use it to select nodes.  
 //否则返回zepto.qsa  
 else dom = zepto.qsa(document, selector)  
 }  
 // create a new Zepto collection from the nodes found  
 //返回Z的实例  
 return zepto.Z(dom, selector)  
 }  
  
 // `  
 // function just call `$.zepto.init, which makes the implementation  
 // details of selecting nodes and creating Zepto collections  
 // patchable in plugins.  
  
 /*  
 继承  
 */  
 //返回zepto.init的调用结果  
 $ = function(selector, context) {  
 return zepto.init(selector, context)  
 }  
  
 function extend(target, source, deep) {  
 for (key in source)  
 if (deep && (isPlainObject(source[key]) || isArray(source[key]))) { //如果是深拷贝且 对象中的属性是一个纯对象或数组（引用类型）  
 if (isPlainObject(source[key]) && !isPlainObject(target[key])) //如果源对象的属性是纯对象且目标对象的属性不是纯对象  
 target[key] = {} //目标对象的属性设为{}  
 if (isArray(source[key]) && !isArray(target[key])) //如果源对象属性是数组且目标对象的属性是数组  
 target[key] = [] //目标对象的属性设为[]  
 extend(target[key], source[key], deep) //调用extend  
 } else if (source[key] !== undefined) target[key] = source[key] //浅拷贝或者属性不是引用类型或者是宿主对象，如果属性值不为undefined  
 }  
  
 // Copy all but undefined properties from one or more  
 // objects to the `target` object.  
 //绑定$的静态方法extend  
 $.extend = function(target) {  
 var deep, args = slice.call(arguments, 1) //转为数组，args从arguments的第二个参数开始取  
 if (typeof target == 'boolean') { //如果target类型为bool  
 deep = target //是否深拷贝  
 target = args.shift() //数组的第一个值，shift删除会影响原数组  
 }  
 //调用extend  
 args.forEach(function(arg) {  
 extend(target, arg, deep)  
 })  
 return target  
 }  
  
 /*  
 选择器  
 */  
  
 // `$.zepto.qsa` is Zepto's CSS selector implementation which  
 // uses `document.querySelectorAll` and optimizes for some special cases, like `#id`.  
 // This method can be overridden in plugins.  
 zepto.qsa = function(element, selector) {  
 var found,  
 maybeID = selector[0] == '#',//selector以#开头认为是id选择器  
 maybeClass = !maybeID && selector[0] == '.',//类选择器  
 //选取选择器的名字  
 nameOnly = maybeID || maybeClass ? selector.slice(1) : selector, // Ensure that a 1 char tag name still gets checked  
 //检测是否是简单选择器  
 isSimple = simpleSelectorRE.test(nameOnly)  
 //如果 1存在getElementById方法，2是简单选择器，3可能是id选择器  
 return (element.getElementById && isSimple && maybeID) ? // Safari DocumentFragment doesn't have getElementById  
 //调用getElementById，结果作为数组返回  
 ((found = element.getElementById(nameOnly)) ? [found] : []) :  
 //如果不满足上面的123，这三个条件  
 //如果满足4不为元素、5不为document，6不是文档碎片这三个条件，返回空数组  
 (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] :  
 //元素或者document或者文档碎片  
 //如果满足7是简单选择器，8不是id选择器，9getElementsByClassName方法存在这三个条件  
 //=>如果是类选择器，调用getElementsByClassName  
 //=>不是类选择器，就调用getElementsByTagName  
 //不满足789的话，调用querySelectorAll  
 //最后调用结果转为数组返回  
 slice.call(  
 isSimple && !maybeID && element.getElementsByClassName ? // DocumentFragment doesn't have getElementsByClassName/TagName  
 maybeClass ? element.getElementsByClassName(nameOnly) : // If it's simple, it could be a class  
 element.getElementsByTagName(selector) : // Or a tag  
 element.querySelectorAll(selector) // Or it's not simple, and we need to query all  
 )  
 }  
  
 //过滤  
 function filtered(nodes, selector) {  
 //如果selector为null或者undefined，返回$(nodes)  
 //否则返回$(nodes).filter(selector)  
 return selector == null ? $(nodes) : $(nodes).filter(selector)  
 }  
  
 //静态方法$.contains  
 //如果document.documentElement.contains方法的存在，  
 $.contains = document.documentElement.contains ?  
 //$.contains方法定义如下  
 function(parent, node) {  
 //contains这个方法判断自身的包含关系时也返回true，所以首先用parent !== node排除掉  
 return parent !== node && parent.contains(node)  
 } ://如果不存在，则  
 function(parent, node) {  
 //如果node存在，node赋值为其父节点  
 while (node && (node = node.parentNode))  
 //判断node是否等于parent，如果是跳出循环，返回true  
 if (node === parent) return true  
 //如果否，继续判断。直到node已经没有父节点了，然后返回false  
 return false  
 }  
  
 function funcArg(context, arg, idx, payload) {  
 //如果arg是方法，调用arg方法，且this指向context,idx和payload作为参数传入，然后返回调用结果  
 //如果不是直接返回arg  
 return isFunction(arg) ? arg.call(context, idx, payload) : arg  
 }  
  
 function setAttribute(node, name, value) {  
 //如果value为null或者undefined，元素移除属性  
 //否则的话，元素设置属性  
 //这里的坑点就是bool属性了  
 value == null ? node.removeAttribute(name) : node.setAttribute(name, value)  
 }  
  
 // access className property while respecting SVGAnimatedString  
 function className(node, value) {  
 var klass = node.className || '',//className  
 svg = klass && klass.baseVal !== undefined//认为是svg  
  
 //如果value为undefined，如果是svg，返回className的baseVal，如果不是svg，直接返回className  
 if (value === undefined) return svg ? klass.baseVal : klass  
 //如果value不为undefined，如果是svg，设置其className的baseVal值，如果不是svg，设置其className值  
 svg ? (klass.baseVal = value) : (node.className = value)  
 }  
  
 // "true"  => true  
 // "false" => false  
 // "null"  => null  
 // "42"    => 42  
 // "42.5"  => 42.5  
 // "08"    => "08"  
 // JSON    => parse if valid  
 // String  => self  
  
 //跟toString方法相反  
 function deserializeValue(value) {  
 //+value + "" == value 貌似适用于数字，布尔和字符串简单类型  
 try {  
 return value ?//如果value为真值  
 value == "true" ||//如果value是字符串true,就返回true  
 //如果value不是字符串true的话...  
 (value == "false" ? false ://value为"false"就返回false  
 value == "null" ? null ://value为"null"就返回null  
 +value + "" == value ? +value ://value为数字或者字符串  
 /^[\[\{]/.test(value) ? $.parseJSON(value) ://如果是数组字符串或者json字符串，调用$.parseJSON方法解析，否则就返回value  
 value) : value//如果value为假值就直接返回  
 } catch (e) {  
 return value  
 }  
 }  
  
 //静态方法，工具方法  
 $.type = type，  
 $.isFunction = isFunction  
 $.isWindow = isWindow  
 $.isArray = isArray  
 $.isPlainObject = isPlainObject  
  
 //是否是空对象  
 $.isEmptyObject = function(obj) {  
 var name  
 for (name in obj) return false//如果对象有属性，返回假  
 return true  
 }  
  
 //判断某一个元素是否在数组中  
 $.inArray = function(elem, array, i) {  
 return emptyArray.indexOf.call(array, elem, i)  
 }  
  
 $.camelCase = camelize//驼峰  
 //去除两头空格  
 $.trim = function(str) {  
 //如果str为null或者undefined，直接返回''  
 //竟然能将数组转为字符串然后去空格  
 return str == null ? "" : String.prototype.trim.call(str)  
 }  
  
 // plugin compatibility  
 $.uuid = 0 //唯一标识码，目前还看不出是什么用处  
 $.support = {} //貌似是为了做兼容  
 $.expr = {} //不知道是什么鬼  
 $.noop = function() {} //空函数  
  
 $.map = function(elements, callback) {  
 var value, values = [],  
 i, key  
 if (likeArray(elements))//如果是类数组for循环调用，不是类数组for in调用  
 for (i = 0; i < elements.length; i++) {  
 value = callback(elements[i], i)//遍历调用callback  
 //如果value不为null或者undefined，将value推入values数组中  
 if (value != null) values.push(value)  
 }   
 else //如果不是类数组  
 for (key in elements) {  
 value = callback(elements[key], key)  
 if (value != null) values.push(value)  
 }  
 return flatten(values)  
 }  
  
 $.each = function(elements, callback) {  
 var i, key  
 if (likeArray(elements)) {//这里也同map一样区分了是否类数组  
 for (i = 0; i < elements.length; i++)  
 //如果调用===false 中断遍历返回，必须是false才会中断  
 if (callback.call(elements[i], i, elements[i]) === false) return elements  
 } else {  
 for (key in elements)  
 if (callback.call(elements[key], key, elements[key]) === false) return elements  
 }  
  
 return elements  
 }  
  
 //过滤数组，返回过滤结果  
 $.grep = function(elements, callback) {  
 return filter.call(elements, callback)  
 }  
  
 if (window.JSON) $.parseJSON = JSON.parse  
  
 //遍历设置class2type对象的属性  
 // Populate the class2type map  
 $.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {  
 class2type["[object " + name + "]"] = name.toLowerCase()  
 })  
  
  
 // Define methods that will be available on all  
 // Zepto collections  
 $.fn = {  
 constructor: zepto.Z,//设置构造器为zepto.Z  
 length: 0,//长度，不知道有什么用  
  
 // Because a collection acts like an array  
 // copy over these useful array functions.  
 forEach: emptyArray.forEach,//数组原生方法  
 reduce: emptyArray.reduce,  
 push: emptyArray.push,  
 sort: emptyArray.sort,  
 splice: emptyArray.splice,  
 indexOf: emptyArray.indexOf,  
 //连接数组，它不会改变原有数组，返回的是一个新的数组  
 concat: function() {  
 var i, value, args = []  
 for (i = 0; i < arguments.length; i++) {  
 value = arguments[i]//遍历arguments  
 //如果当前参数value是zepto.Z的实例，args[i]是转为数组的value,  
 //如果不是zepto.Z的实例,args[i]为value  
 args[i] = zepto.isZ(value) ? value.toArray() : value  
 }  
 //调用concat方法，如果this是zepto.z的实例，this指向转为数组后的this  
 return concat.apply(zepto.isZ(this) ? this.toArray() : this, args)  
 },  
  
 // `map` and `slice` in the jQuery API work differently  
 // from their array counterparts  
 map: function(fn) {  
 //遍历this，调用$.map,返回的结果转为zepto.Z的实例  
 return $($.map(this, function(el, i) {  
 return fn.call(el, i, el)  
 }))  
 },  
 slice: function() {  
 //调用slice，返回的结果转为zepto.Z的实例  
 return $(slice.apply(this, arguments))  
 },  
  
 ready: function(callback) {  
 // need to check if document.body exists for IE as that browser reports  
 // document ready when it hasn't yet created the body element  
 //如果readyState的状态为complete|loaded|interactive，并且document.body存在，调用回调  
 if (readyRE.test(document.readyState) && document.body) callback($)  
 //否则，添加DOMContentLoaded事件绑定  
 else document.addEventListener('DOMContentLoaded', function() {  
 callback($)  
 }, false)  
 return this  
 },  
 get: function(idx) {  
 //如果idx为undefined，返回slice.call(this)，转为数组的  
 //如果不为undefined，这时返回不是数组而是dom元素  
 //=>如果idx大于等于0，返回this[idx]  
 //=>否则返回this[idx + this.length]，即倒数第x个  
 return idx === undefined ? slice.call(this) : this[idx >= 0 ? idx : idx + this.length]  
 },  
 //将this转为数组  
 toArray: function() {  
 return this.get()  
 },  
 //获取length属性  
 size: function() {  
 return this.length  
 },  
 remove: function() {  
  
 return this.each(function() {  
 //先判断是否有父节点，如果有就可移除this  
 if (this.parentNode != null)  
 this.parentNode.removeChild(this)  
 })  
 },  
 each: function(callback) {  
 //调用数组的every方法，坑点，$().each方法回调中使用return false无效  
 //所以还是使用$.each吧  
 emptyArray.every.call(this, function(el, idx) {  
 return callback.call(el, idx, el) !== false  
 })  
 return this  
 },  
 filter: function(selector) {  
 //如果selector是函数  
 if (isFunction(selector)) return this.not(this.not(selector))  
 //调用filter方法，过滤出匹配的元素，返回结果转为zepto.Z的实例  
 return $(filter.call(this, function(element) {  
 //如果元素匹配选择器，返回true  
 return zepto.matches(element, selector)  
 }))  
 },  
 add: function(selector, context) {  
 //首先$(selector, context)生成了一个zepto.Z的实例  
 //调用this.concat方法合并后，转为了数组  
 //经过数组去重，最后又转为zepto.Z的实例  
 return $(uniq(this.concat($(selector, context))))  
 },  
 is: function(selector) {  
 //length>0且判断this的第一个元素是否匹配选择器  
 return this.length > 0 && zepto.matches(this[0], selector)  
 },  
 not: function(selector) {  
 var nodes = []  
 //如果selector是函数，并且selector.call存在  
 if (isFunction(selector) && selector.call !== undefined)  
 //遍历this  
 this.each(function(idx) {  
 //调用selector，修改this指向，如果返回结果为假值，push到nodes数组中  
 if (!selector.call(this, idx)) nodes.push(this)  
 })  
 else {//否则的话，判断selector是否是字符串，如果是字符串，excludes=调用this.filter结果  
 var excludes = typeof selector == 'string' ? this.filter(selector) :  
 //如果不是字符串，判断它是否是类数组，  
 //且是否有item（HTMLCollection、NodeList、NamedNodeMap这三个都有item方法）  
 //如果为真，excludes=转为数组selector  
 //如果为假，selector=selector转为zepto.Z的实例  
 (likeArray(selector) && isFunction(selector.item)) ? slice.call(selector) : $(selector)  
 //如果不在excludes数组后，push进去  
 this.forEach(function(el) {  
 if (excludes.indexOf(el) < 0) nodes.push(el)  
 })  
 }  
 //转为zepto.Z的实例后返回  
 return $(nodes)  
 },  
 has: function(selector) {  
 //调用this.filter，返回过滤后的结果  
 return this.filter(function() {  
 //内部的过滤方案是首先判断selector是否是一个对象  
 //如果是对象，判断是否包含，包含返回true  
 //如果不是对象，将this转为zepto.Z的实例，调用find方法，返回结果集合的length  
 return isObject(selector) ?  
 $.contains(this, selector) :  
 $(this).find(selector).size()  
 })  
 },  
 eq: function(idx) {  
 //如果idx为-1，返回this.slice(idx)  
 //否则返回this.slice(idx, +idx + 1)  
 return idx === -1 ? this.slice(idx) : this.slice(idx, +idx + 1)  
 },  
 first: function() {  
 var el = this[0]  
 //el为真值且el不是对象的话，返回el  
 //否则返回$(el)  
 //=>假值返回的是[]，对象是转为zepto.Z的实例后返回  
 return el && !isObject(el) ? el : $(el)  
 },  
 last: function() {  
 //el是最后一个元素，其他逻辑同first  
 var el = this[this.length - 1]  
 return el && !isObject(el) ? el : $(el)  
 },  
 find: function(selector) {  
 var result, $this = this  
 //选择器不存在，返回空的zepto.Z实例  
 if (!selector) result = $()  
 else if (typeof selector == 'object')//如果typeof结果是对象  
 //filter过滤，将其转为zepto.Z实例  
 result = $(selector).filter(function() {  
 var node = this  
 //some用于检测数组是否满足条件，只要有一个满足则返回true  
 //这里检测是否包含子元素  
 return emptyArray.some.call($this, function(parent) {  
 return $.contains(parent, node)  
 })  
 })  
 //selector不为对象或数组且this.length为1，直接使用zepto.qsa然后转为zepto.Z实例  
 else if (this.length == 1) result = $(zepto.qsa(this[0], selector))  
 //selector不为对象或数组且this.length不为1  
 else result = this.map(function() {  
 return zepto.qsa(this, selector)  
 })  
 return result  
 },  
 //从元素本身开始，逐级向上级元素匹配，并返回最先匹配selector的元素。如果给定context节点参数，那么只匹配该节点的后代元素。  
 closest: function(selector, context) {  
 var node = this[0],//获取this第一个元素  
 collection = false //不是html集合？  
 //如果selector是对象类型，将selector转为zepto.Z的实例后赋给collection  
 if (typeof selector == 'object') collection = $(selector)  
 //this的第一个元素存在  
 //且第二个表达式为假  
 //collection ? collection.indexOf(node) >= 0 : zepto.matches(node, selector)  
 //=>collection为真则判断collection中是否能找到node  
 //=>为假则（selector不是对象，可能是字符串选择器之类的）调用 zepto.matches  
 while (node && !(collection ? collection.indexOf(node) >= 0 : zepto.matches(node, selector)))  
 //1、判断不为本身，比如$('window')[0]===window  
 //2、判断不为document  
 //3、返回其父节点  
 node = node !== context && !isDocument(node) && node.parentNode  
 return $(node)  
 },  
 parents: function(selector) {  
 var ancestors = [],//祖先  
 nodes = this  
 while (nodes.length > 0)  
 //遍历this  
 nodes = $.map(nodes, function(node) {  
 //node.parentNode赋值给node、判断是否存在  
 //判断node是不是document  
 //判断node还没有存入祖先元素数组中  
 if ((node = node.parentNode) && !isDocument(node) && ancestors.indexOf(node) < 0) {  
 //添加到祖先元素数组  
 ancestors.push(node)  
 return node  
 }  
 })  
 return filtered(ancestors, selector)  
 },  
 parent: function(selector) {  
 //遍历获取this的parentNode属性值，去重后返回filtered的调用结果  
 return filtered(uniq(this.pluck('parentNode')), selector)  
 },  
 children: function(selector) {  
 //遍历调用,this.map内部做了数组扁平化  
 return filtered(this.map(function() {  
 return children(this)  
 }), selector)  
 },  
 contents: function() {  
 return this.map(function() {  
 //contentDocument哪里冒出来的。。。  
 //childNodes包含文本节点与注释  
 return this.contentDocument || slice.call(this.childNodes)  
 })  
 },  
 siblings: function(selector) {  
 //父元素的children过滤掉自身  
 return filtered(this.map(function(i, el) {  
 return filter.call(children(el.parentNode), function(child) {  
 return child !== el  
 })  
 }), selector)  
 },  
 empty: function() {  
 //遍历修改自身的innerHTML  
 return this.each(function() {  
 this.innerHTML = ''  
 })  
 },  
 //遍历获取属性值  
 //$('.item').pluck('title')  
 //=>["多件多折", "Top榜", "全球购", "下期预告", "更多分类", "纸尿裤", "奶粉", "非常大牌", "五羊"]  
 // `pluck` is borrowed from Prototype.js  
 pluck: function(property) {  
 return $.map(this, function(el) {  
 return el[property]  
 })  
 },  
 show: function() {  
 return this.each(function() {  
 //如果display为none，display改为''  
 //我的理解是清除行内样式  
 //http://stackoverflow.com/questions/7420109/what-does-style-display-actually-do  
 this.style.display == "none" && (this.style.display = '')  
 //如果this的计算样式为none，赋值为其默认的display值  
 //清除行内样式后，仍有可能样式表中也设了css，所以如果没效，还是要去设置一下默认值  
 if (getComputedStyle(this, '').getPropertyValue("display") == "none")  
 this.style.display = defaultDisplay(this.nodeName)  
 })  
 },  
 replaceWith: function(newContent) {  
 return this.before(newContent).remove()  
 },  
 wrap: function(structure) {  
 var func = isFunction(structure)//判断structure是否是函数  
 if (this[0] && !func)//如果不为函数  
 var dom = $(structure).get(0),  
 clone = dom.parentNode || this.length > 1//structure有父元素或者this集合中有多个元素  
  
 return this.each(function(index) {  
 $(this).wrapAll(  
 func ? structure.call(this, index) :  
 clone ? dom.cloneNode(true) : dom  
 )  
 })  
 },  
 //包裹外层  
 wrapAll: function(structure) {  
 if (this[0]) {  
 $(this[0]).before(structure = $(structure))  
 var children  
 while ((children = structure.children()).length) structure = children.first()  
 $(structure).append(this)  
 }  
 return this  
 },  
 //内容包裹  
 wrapInner: function(structure) {  
 var func = isFunction(structure)//判断是否是函数  
 return this.each(function(index) {  
 var self = $(this),  
 contents = self.contents(),//获取内容  
 //如果是函数，dom=structure的调用结果，否则dom=structure  
 dom = func ? structure.call(this, index) : structure  
 //内容存在，调用contents.wrapAll ，否则调用append  
 contents.length ? contents.wrapAll(dom) : self.append(dom)  
 })  
 },  
 unwrap: function() {  
 this.parent().each(function() {  
 //子元素替换自身  
 $(this).replaceWith($(this).children())  
 })  
 return this  
 },  
 clone: function() {  
 return this.map(function() {  
 return this.cloneNode(true)  
 })  
 },  
 hide: function() {  
 return this.css("display", "none")  
 },  
 toggle: function(setting) {  
 return this.each(function() {  
 var el = $(this);  
 //setting为undefined时，判断元素的display是否为none,否则判断setting真假  
 (setting === undefined ? el.css("display") == "none" : setting) ? el.show() : el.hide()  
 })  
 },  
 prev: function(selector) {  
 return $(this.pluck('previousElementSibling')).filter(selector || '*')  
 },  
 next: function(selector) {  
 return $(this.pluck('nextElementSibling')).filter(selector || '*')  
 },  
 html: function(html) {  
 //html如果存在  
 return 0 in arguments ?  
 this.each(function(idx) {  
 var originHtml = this.innerHTML//暂存innerHTML  
 //funcArg用于处理参数是函数的情况  
 $(this).empty().append(funcArg(this, html, idx, originHtml))  
 }) :  
 //html如不存在，取值  
 (0 in this ? this[0].innerHTML : null)  
 },  
 text: function(text) {  
 return 0 in arguments ?  
 this.each(function(idx) {  
 var newText = funcArg(this, text, idx, this.textContent)  
 //如果为null转为''  
 this.textContent = newText == null ? '' : '' + newText  
 }) :  
 //获取所有子元素的textContent属性值，用空字符串拼接  
 //textContent的兼容性是ie8及以下不支持  
 (0 in this ? this.pluck('textContent').join("") : null)  
 },  
 attr: function(name, value) {  
 var result  
 //如果name是字符串类型 且value不存在，取值  
 //=>this.length不存在或者this不是element元素，返回undefined  
 //=>存在则获取this第一个元素的name属性this[0].getAttribute(name)，赋值给result，判断result是否为真值，再判断this的第一个元素是否有name属性name in this[0]  
 //如果getAttribute方式取不到值，但name in ...为真，则返回this[0][name]  
 //否则返回result  
 //这里可以区分下属性、特性的区别。。。  
 return (typeof name == 'string' && !(1 in arguments)) ?  
 (!this.length || this[0].nodeType !== 1 ? undefined :  
 (!(result = this[0].getAttribute(name)) && name in this[0]) ? this[0][name] : result  
 ) :  
 this.each(function(idx) {  
 //如果不是元素节点，返回  
 if (this.nodeType !== 1) return  
 //如果是对象，遍历设值   
 if (isObject(name))  
 for (key in name) setAttribute(this, key, name[key])  
 //单个设值，要考虑可能value是函数的情景  
 else setAttribute(this, name, funcArg(this, value, idx, this.getAttribute(name)))  
 })  
 },  
 removeAttr: function(name) {  
 return this.each(function() {  
 //节点类型是元素  
 //多个attr可以用空格分隔  
 this.nodeType === 1 && name.split(' ').forEach(function(attribute) {  
 //第三个参数未填则为移除属性  
 setAttribute(this, attribute)  
 }, this)  
 })  
 },  
 prop: function(name, value) {  
 name = propMap[name] || name//首先判断propMap中是否存在  
 return (1 in arguments) ?//设值  
 this.each(function(idx) {  
 this[name] = funcArg(this, value, idx, this[name])  
 }) ://取值  
 (this[0] && this[0][name])  
 },  
 //内部竟然调用的是attr,只不过在最后做了下数据类型转换  
 data: function(name, value) {  
 var attrName = 'data-' + name.replace(capitalRE, '-$1').toLowerCase()  
  
 var data = (1 in arguments) ?  
 this.attr(attrName, value) :  
 this.attr(attrName)  
 //如果不为null，返回deserializeValue(data)，否则返回undefined  
 return data !== null ? deserializeValue(data) : undefined  
 },  
 val: function(value) {  
 //设值  
 return 0 in arguments ?  
 this.each(function(idx) {  
 this.value = funcArg(this, value, idx, this.value)  
 }) :  
 //取值  
 //如果存在multiple属性，获取所有option下面的value值  
 (this[0] && (this[0].multiple ?  
 $(this[0]).find('option').filter(function() {  
 return this.selected  
 }).pluck('value') :  
 this[0].value))//否则取第一个元素的值  
 } ,  
 offset: function(coordinates) {  
 //设值，目标值减去获取最近非静态定位祖先元素的offset值  
 //然后设置css  
 if (coordinates) return this.each(function(index) {  
 var $this = $(this),  
 coords = funcArg(this, coordinates, index, $this.offset()),  
 parentOffset = $this.offsetParent().offset(),  
 props = {  
 top: coords.top - parentOffset.top,  
 left: coords.left - parentOffset.left  
 }  
  
 if ($this.css('position') == 'static') props['position'] = 'relative'  
 $this.css(props)  
 })  
 //如果this为空集合，返回null  
 if (!this.length) return null  
 //如果document.documentElement不包含this，iframe??  
 if (!$.contains(document.documentElement, this[0]))  
 return {  
 top: 0,  
 left: 0  
 }  
 //返回一个矩形对象，包含四个属性：left、top、right和bottom。分别表示元素各边与页面上边和左边的距离。  
 var obj = this[0].getBoundingClientRect()  
 return {  
 left: obj.left + window.pageXOffset,  
 top: obj.top + window.pageYOffset,  
 width: Math.round(obj.width),  
 height: Math.round(obj.height)  
 }  
 },  
 css: function(property, value) {  
 //取值  
 if (arguments.length < 2) {  
 var computedStyle, element = this[0]//第一个元素  
 if (!element) return  
 computedStyle = getComputedStyle(element, '')  
 if (typeof property == 'string')//如果是字符串  
 //首先获取行内样式，如果为假值再去获取计算后的样式  
 return element.style[camelize(property)] || computedStyle.getPropertyValue(property)  
 else if (isArray(property)) {//如果是数组，返回对象  
 var props = {}  
 $.each(property, function(_, prop) {  
 props[prop] = (element.style[camelize(prop)] || computedStyle.getPropertyValue(prop))  
 })  
 return props  
 }  
 }  
  
 var css = ''  
 if (type(property) == 'string') {//如果是字符串  
 if (!value && value !== 0)//value为假值且不为0，认为是移除属性  
 this.each(function() {  
 this.style.removeProperty(dasherize(property))  
 })  
 else//添加css  
 css = dasherize(property) + ":" + maybeAddPx(property, value)  
 } else {  
 for (key in property)  
 if (!property[key] && property[key] !== 0)  
 this.each(function() {  
 this.style.removeProperty(dasherize(key))  
 })  
 else  
 css += dasherize(key) + ':' + maybeAddPx(key, property[key]) + ';'  
 }  
  
 return this.each(function() {  
 this.style.cssText += ';' + css  
 })  
 },  
 index: function(element) {  
 //如果element存在，返回element第一个元素在this中的index  
 //如果不存在，返回this在其父元素中的index  
 return element ? this.indexOf($(element)[0]) : this.parent().children().indexOf(this[0])  
 },  
 hasClass: function(name) {  
 if (!name) return false  
 return emptyArray.some.call(this, function(el) {  
 return this.test(className(el))  
 }, classRE(name))  
 },  
 addClass: function(name) {  
 if (!name) return this  
 return this.each(function(idx) {  
 if (!('className' in this)) return  
 classList = []  
 var cls = className(this),  
 newName = funcArg(this, name, idx, cls)  
 newName.split(/\s+/g).forEach(function(klass) {  
 if (!$(this).hasClass(klass)) classList.push(klass)  
 }, this)  
 classList.length && className(this, cls + (cls ? " " : "") + classList.join(" "))  
 })  
 },  
 removeClass: function(name) {  
 return this.each(function(idx) {  
 if (!('className' in this)) return  
 if (name === undefined) return className(this, '')  
 classList = className(this)  
 funcArg(this, name, idx, classList).split(/\s+/g).forEach(function(klass) {  
 classList = classList.replace(classRE(klass), " ")  
 })  
 className(this, classList.trim())  
 })  
 },  
 toggleClass: function(name, when) {  
 if (!name) return this  
 return this.each(function(idx) {  
 var $this = $(this),  
 names = funcArg(this, name, idx, className(this))  
 names.split(/\s+/g).forEach(function(klass) {  
 (when === undefined ? !$this.hasClass(klass) : when) ?  
 $this.addClass(klass) : $this.removeClass(klass)  
 })  
 })  
 },  
 scrollTop: function(value) {  
 if (!this.length) return  
 //如果第一个元素有scrollTop属性，hasScrollTop为true  
 var hasScrollTop = 'scrollTop' in this[0]  
 //如果value未定义，判断hasScrollTop的值，为真返回scrollTop，否者返回pageYOffset  
 if (value === undefined) return hasScrollTop ? this[0].scrollTop : this[0].pageYOffset  
 return this.each(hasScrollTop ?  
 function() {  
 this.scrollTop = value  
 } :  
 function() {  
 this.scrollTo(this.scrollX, value)  
 })  
 },  
 scrollLeft: function(value) {  
 if (!this.length) return  
 var hasScrollLeft = 'scrollLeft' in this[0]  
 if (value === undefined) return hasScrollLeft ? this[0].scrollLeft : this[0].pageXOffset  
 return this.each(hasScrollLeft ?  
 function() {  
 this.scrollLeft = value  
 } :  
 function() {  
 this.scrollTo(value, this.scrollY)  
 })  
 },  
 //获取对象集合中第一个元素的位置，  
 position: function() {  
 if (!this.length) return  
  
 var elem = this[0],//第一个元素  
 // Get *real* offsetParent  
 offsetParent = this.offsetParent(),//最近非静态定位祖先元素  
 // Get correct offsets  
 offset = this.offset(),//相对于文档  
 //如果祖先元素是跟节点，parentOffset为{top:0,left:0}  
 //如果不是跟节点，parentOffset的值为offsetParent.offset()  
 parentOffset = rootNodeRE.test(offsetParent[0].nodeName) ? {  
 top: 0,  
 left: 0  
 } : offsetParent.offset()  
  
 // Subtract element margins  
 // note: when an element has margin: auto the offsetLeft and marginLeft  
 // are the same in Safari causing offset.left to incorrectly be 0  
 //自身的offset减去margin  
 offset.top -= parseFloat($(elem).css('margin-top')) || 0  
 offset.left -= parseFloat($(elem).css('margin-left')) || 0  
  
 // Add offsetParent borders  
 //父级的offset加上border  
 parentOffset.top += parseFloat($(offsetParent[0]).css('border-top-width')) || 0  
 parentOffset.left += parseFloat($(offsetParent[0]).css('border-left-width')) || 0  
  
 // Subtract the two offsets  
 //两者相减  
 return {  
 top: offset.top - parentOffset.top,  
 left: offset.left - parentOffset.left  
 }  
 },  
 //最近非静态定位的祖先元素  
 offsetParent: function() {  
 return this.map(function() {  
 var parent = this.offsetParent || document.body //最近非静态定位  
 //parent存在且不是根节点且parent的定位不是static  
 while (parent && !rootNodeRE.test(parent.nodeName) && $(parent).css("position") == "static")  
 parent = parent.offsetParent  
 return parent  
 })  
 }  
 }  
  
 // for now  
 //估计是为了新旧版本兼容... 但是在jQ中，这两者是不同的  
 $.fn.detach = $.fn.remove  
  
 // Generate the `width` and `height` functions  
 ;  
 ['width', 'height'].forEach(function(dimension) {  
  
 //首字母大写？  
 var dimensionProperty = dimension.replace(/./, function(m) {  
 return m[0].toUpperCase()  
 })  
  
 //生成$.fn.width,$.fn.height   
 $.fn[dimension] = function(value) {  
 var offset, el = this[0]  
 //取值  
 //如果是window，返回window.innerWidth/innerHeight  
 if (value === undefined) return isWindow(el) ? el['inner' + dimensionProperty] :  
 //如果是document，返回document.documentElement.scrollWidth/scrollHeight  
 isDocument(el) ? el.documentElement['scroll' + dimensionProperty] :  
 //返回获取其offset().width/height  
 (offset = this.offset()) && offset[dimension]  
 //设值，通过css。。。  
 else return this.each(function(idx) {  
 el = $(this)  
 el.css(dimension, funcArg(this, value, idx, el[dimension]()))  
 })  
 }  
 })  
  
 //自身和子元素都会调用下fun  
 function traverseNode(node, fun) {  
 fun(node)  
 for (var i = 0, len = node.childNodes.length; i < len; i++)  
 traverseNode(node.childNodes[i], fun)  
 }  
  
 // Generate the `after`, `prepend`, `before`, `append`,  
 // `insertAfter`, `insertBefore`, `appendTo`, and `prependTo` methods.  
 //遍历adjacencyOperators   
 adjacencyOperators.forEach(function(operator, operatorIndex) {  
 //话说这么写不太科学啊，数组换个位置不就bug了  
 var inside = operatorIndex % 2 //=> prepend, append  
  
 //生成$.fn.xxx  
 $.fn[operator] = function() {  
 // arguments can be nodes, arrays of nodes, Zepto objects and HTML strings  
 var argType, nodes = $.map(arguments, function(arg) {//参数类型  
 argType = type(arg)//获取参数类型  
 //如果参数是对象、数组或者null或者undefined，nodes=arg，否则nodes=生成的html片段  
 return argType == "object" || argType == "array" || arg == null ?  
 arg : zepto.fragment(arg)  
 }),  
 parent, copyByClone = this.length > 1//是否需要深度克隆  
 if (nodes.length < 1) return this//如果nodes为null，undefined或者空数组或者空的zepto.Z的实例，直接返回this  
  
 //遍历this  
 return this.each(function(_, target) {  
 //如果是prepend, append，parent取值target，否则取值target的父节点  
 //因为后面都要转成before操作，所以这里要转一下target  
 parent = inside ? target : target.parentNode  
  
 // convert all methods to a "before" operation  
 // 都转为before操作  
 target = operatorIndex == 0 ? target.nextSibling ://operatorIndex为0，target为target的下一个兄弟元素  
 operatorIndex == 1 ? target.firstChild ://operatorIndex为1，target为target的第一个子元素  
 operatorIndex == 2 ? target ://operatorIndex为2，target为target  
 null//operatorIndex为3，target为null  
  
 var parentInDocument = $.contains(document.documentElement, parent)  
  
 nodes.forEach(function(node) {  
 if (copyByClone) node = node.cloneNode(true)//如果为true，克隆后代节点  
 else if (!parent) return $(node).remove()//如果parent不存在，移除$(node)，返回值也是$(node)  
  
 parent.insertBefore(node, target)//插入节点  
 //如果parent在document中  
 if (parentInDocument) traverseNode(node, function(el) {  
 //nodeName不为null且是script且type属性不存在或者为text/javascript  
 if (el.nodeName != null && el.nodeName.toUpperCase() === 'SCRIPT' && (!el.type || el.type === 'text/javascript') && !el.src)  
 //eval全局环境执行script中的代码  
 //这里可以拓展下关注eval的直接调用和间接调用  
 window['eval'].call(window, el.innerHTML)  
 })  
 })  
 })  
 }  
  
 // after    => insertAfter  
 // prepend  => prependTo  
 // before   => insertBefore  
 // append   => appendTo  
 $.fn[inside ? operator + 'To' : 'insert' + (operatorIndex ? 'Before' : 'After')] = function(html) {  
 $(html)[operator](this)  
 return this  
 }  
 })  
  
 //这一句很关键  
 //zepto.isZ能正确判断继承关系就是因为它  
 zepto.Z.prototype = Z.prototype = $.fn  
  
 // Export internal API functions in the `$.zepto` namespace  
 zepto.uniq = uniq //去重  
 zepto.deserializeValue = deserializeValue //与toString相反  
 $.zepto = zepto //核心对象  
  
 return $  
  
})()  
  
  
//绑定Zepto变量，如果$未定义，指向Zepto  
window.Zepto = Zepto  
window.$ === undefined && (window.$ = Zepto)  

```

