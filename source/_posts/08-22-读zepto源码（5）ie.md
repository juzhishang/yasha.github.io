title: 读zepto源码（5）ie
date: 2016-08-22
tags: 
 - zepto
---


看到代码这么短好开心，因为今天实在是有点困。前段时间换工作，然后忙着读项目代码，于是读zepto源码的安排就各种搁置。

ie这个模块，现在看来是历史遗留问题。在zepto的早期版本中，zepto.Z直接返回dom，于是，dom的原型就成了问题，如果不修改的话，没有办法与$.fn建立关联啊。为了让$.fn与zepto.Z的实例的原型指向相同，用的是**proto**这个私有属性，dom的原型强制指向了$.fn。

我们来看下旧版的实现(话说好像跑题跑到核心模块去了，不管，正好做个补充。。。)

```
 zepto.Z = function(dom, selector) {  
 dom = dom || [] //对象数组  
 dom.__proto__ = $.fn //将dom的原型指向$.fn  
 dom.selector = selector || ''  
 return dom  
 }  
  
zepto.init = function(selector, context) {  
 return zepto.Z(dom, selector)  
}  
  
zepto.Z.prototype = $.fn  

```

而新版是这样，它使用的是Z.prototype。zepto.Z这里返回的是new Z()构造出来的实例。通过原型继承很容易就把实例的原型**proto**与$.fn关联起来了。

```
function Z(dom, selector) {  
 var i, len = dom ? dom.length : 0  
 for (i = 0; i < len; i++) this[i] = dom[i]  
 this.length = len  
 this.selector = selector || ''  
}  
  
zepto.Z = function(dom, selector) {  
 return new Z(dom, selector)  
}  
  
zepto.Z.prototype = Z.prototype = $.fn  

```

好吧，其实上面说了一堆，只是想说明，旧版的zepto使用了**proto**，而新版找到了更好的方式，放弃了**proto**方案。

因为**proto**存在一些兼容性问题，在ie11-是不支持的。所以旧版中ie模块重写了zepto.Z和zepto.isZ:

```
if (!('__proto__' in {})) {  
 $.extend($.zepto, {  
 Z: function(dom, selector){  
 dom = dom || []  
 $.extend(dom, $.fn)  
 dom.selector = selector || ''  
 dom.__Z = true  
 return dom  
 },  
 // this is a kludge but works  
 isZ: function(object){  
 return $.type(object) === 'array' && '__Z' in object  
 }  
 })  
}  

```

而到了新版，这段兼容代码自然被删除了。只剩下对getComputedStyle的处理。而我测试getComputedStyle这个方法发现，参数不填并不是只有ie报错，因此，我才觉得，这个模块，它是存在是因为历史遗留问题，或许哪一天就没了呢。

firefox：TypeError: Not enough arguments to Window.getComputedStyle.

chrome: Uncaught TypeError: Failed to execute ‘getComputedStyle’ on ‘Window’: 1 argument required, but only 0 present.

ie: 缺少对象

最后，我们来看下新版中ie.js这个模块（其实只有getComputedStyle方法）做了什么。

```
 ;(function(){  
 // getComputedStyle shouldn't freak out when called  
 // without a valid element as argument  
 try {  
 getComputedStyle(undefined)  
 } catch(e) {  
 var nativeGetComputedStyle = getComputedStyle;  
 window.getComputedStyle = function(element){  
 try {  
 return nativeGetComputedStyle(element)  
 } catch(e) {  
 return null  
 }  
 }  
 }  
})()  

```

尝试调用了getComputedStyle(undefined)，因为没有参数，这一段很有可能是要报错的（我只测了ie8和Chrome某版本），zepto捕获异常后，重写了window.getComputedStyle，这里对无参数的情况做了处理。简单的说，它的功能就是在用户调用前先把异常给处理掉。

