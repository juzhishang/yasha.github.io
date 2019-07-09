title: 读zepto源码（3) ajax
date: 2016-07-19
tags: 
 - zepto
---


ajax模块我觉得它的实现可以分为以下三个部分：

### 一、ajax自定义全局方法，包括：

*   ajaxStart
*   ajaxStop
*   ajaxBeforeSend
*   ajaxSuccess
*   ajaxError
*   ajaxComplete

从方法名称就可以看出，是与处理ajax回调相关的。不过这些方法不能直接调用。zepto中它们只是作为内部方法，并没有把这些暴露出来。它们都调用了triggerGlobal方法，而triggerGlobal则调用了triggerAndReturn。

ajaxError的type值有：”timeout”, “error”, “abort”, “parsererror”，  
ajaxComplate的type值有：”success”, “notmodified”, “error”, “timeout”, “abort”, “parsererror”

```
  
function triggerAndReturn(context, eventName, data) {  
 var event = $.Event(eventName)  
 $(context).trigger(event, data)  
 return !event.isDefaultPrevented()  
}  
  
// trigger an Ajax "global" event  
//settings.global是否触发ajax全局事件  
//上下文如果不存在，就设为document  
//如果settings.global为真，就触发上下文的事件，它内部调用了triggerAndReturn  
function triggerGlobal(settings, context, eventName, data) {  
 if (settings.global) return triggerAndReturn(context || document, eventName, data)  
}  

```

### 二、-ajax与-ajaxJSONP，及一些ajax快捷方法

ajax与ajaxJSONP是ajax模块的核心。源码就不贴了，太长。。。

ajax是用XMLHttpRequest实现的，而$.ajaxJSONP是用于实现jsonp。因为jsonp的实现原理其实是动态创建script标签。当zepto判定是jsonp方式时，它会内部调用$.ajaxJSONP。

注意：当我们不传jsonCallback时，zepto会自己生成一个临时的callback名称，用jsonpID控制。  
zepto会默认不会对dataType为jsonp和script的get请求做缓存。

快捷方法：这几个都是内部调用$.ajax。load方法被挂在原型上，因为这个方法是用于将请求返回的hml插入到匹配元素中。

*   $.get
*   $.post
*   $.getJSON

*   $.fn.load

我觉得$.ajax实现而延伸出去的一些知识点很值得关注。比如下面这两篇文章：

[理解DOMString、Document、FormData、Blob、File、ArrayBuffer数据类型](http://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/)

[你真的会使用XMLHttpRequest吗？](https://segmentfault.com/a/1190000004322487)

### 三、工具方法-param，它的作用是将对象转为序列化字符串

```
$.param = function(obj, traditional){  
 var params = []  
 //add方法用于将key和value经过encodeURIComponent编码后以key=value的形式添加到params数组中  
 params.add = function(key, value) {  
 //如果value是函数，value重新赋值为其自身的调用结果  
 if ($.isFunction(value)) value = value()  
 //如果是null，value为空  
 if (value == null) value = ""  
 //对key和value encodeURIComponent编码  
 this.push(escape(key) + '=' + escape(value))  
 }  
 serialize(params, obj, traditional)  
 //将数组转为字符串，用&拼接，%20表示空格，这里替换成+  
 return params.join('&').replace(/%20/g, '+')  
}  

```

$.param内部调用了方法serialize，serialize它将对象以key=value的形式存储在参数params中，可惜zepto也没有将这个方法暴露出来。

ajax请求各个方法调用顺序：

ajaxStart->settings.beforeSend->ajaxBeforeSend(不一定会调用)->ajaxSend->settings.success->ajaxSuccess->settings.complete->ajaxComplete

### 源码注释

```
//     Zepto.js  
//     (c) 2010-2016 Thomas Fuchs  
//     Zepto.js may be freely distributed under the MIT license.  
  
;(function($){  
 var jsonpID = 0,//在jsonCallback未传时，会被用来生成一个临时的jsonCallback  
 document = window.document,  
 key,  
 name,  
 //$.fn.load中会用到，有选择器时会过滤掉响应中的script  
 rscript = /)/gi,//匹配script标签  
 //application/javascript是服务器端处理js文件的mime类型  
 //匹配script的type，type/javascript或者application/javascript  
 scriptTypeRE = /^(?:text|application)\/javascript/i,  
 xmlTypeRE = /^(?:text|application)\/xml/i,  
 jsonType = 'application/json',  
 htmlType = 'text/html',  
 blankRE = /^\s*$/,//匹配任意个空格  
 originAnchor = document.createElement('a')  
  
 //默认a链接的href为当前页面href  
 //originAnchor用于检测是否跨域  
 originAnchor.href = window.location.href  
  
 // trigger a custom event and return false if it was cancelled  
 //触发事件，并返回事件是否被阻止默认事件，如果被阻止默认事件，返回false  
 function triggerAndReturn(context, eventName, data) {  
 var event = $.Event(eventName)  
 $(context).trigger(event, data)  
 return !event.isDefaultPrevented()  
 }  
  
 // trigger an Ajax "global" event  
 //settings.global是否触发ajax全局事件  
 //上下文如果不存在，就设为document  
 //如果settings.global为真，就触发上下文的事件，它内部调用了triggerAndReturn  
 function triggerGlobal(settings, context, eventName, data) {  
 if (settings.global) return triggerAndReturn(context || document, eventName, data)  
 }  
  
 // Number of active Ajax requests  
 //用于标识ajax请求的数量  
 //√  
 $.active = 0  
  
 //触发全局ajaxStart事件  
 function ajaxStart(settings) {  
 //判断settings.global是否为真，且$.active的值是否为0，判断完毕$.active+1  
 //第二个参数context为null，说明事件触发在document对象上，自定义事件名称为'ajaxStart'  
 if (settings.global && $.active++ === 0) triggerGlobal(settings, null, 'ajaxStart')  
 }  
 //触发全局ajaxStop事件  
 function ajaxStop(settings) {  
 //$.active-1  
 if (settings.global && !(--$.active)) triggerGlobal(settings, null, 'ajaxStop')  
 }  
  
 // triggers an extra global event "ajaxBeforeSend" that's like "ajaxSend" but cancelable  
 //ajax发送请求之前触发事件  
 function ajaxBeforeSend(xhr, settings) {  
 var context = settings.context  
 //如果调用settings.beforeSend事件返回false或者触发全局ajaxBeforeSend方法(默认事件被阻止)返回false，返回。  
 if (settings.beforeSend.call(context, xhr, settings) === false ||  
 triggerGlobal(settings, context, 'ajaxBeforeSend', [xhr, settings]) === false)  
 return false  
 //否则的话触发全局ajaxSend方法  
 triggerGlobal(settings, context, 'ajaxSend', [xhr, settings])  
 }  
  
 function ajaxSuccess(data, xhr, settings, deferred) {  
 var context = settings.context, status = 'success'  
 //调用settings.success方法  
 settings.success.call(context, data, status, xhr)  
 //如果deferred对象存在，调用deferred.resolveWith方法  
 if (deferred) deferred.resolveWith(context, [data, status, xhr])  
 //触发全局事件ajaxSuccess  
 triggerGlobal(settings, context, 'ajaxSuccess', [xhr, settings, data])  
 //调用ajaxComplete方法  
 ajaxComplete(status, xhr, settings)  
 }  
 // type: "timeout", "error", "abort", "parsererror"  
 function ajaxError(error, type, xhr, settings, deferred) {  
 var context = settings.context  
 //调用settings.error方法  
 settings.error.call(context, xhr, type, error)  
 //如果deferred对象存在，调用deferred.rejectWith方法  
 if (deferred) deferred.rejectWith(context, [xhr, type, error])  
 //触发全局事件ajaxError  
 triggerGlobal(settings, context, 'ajaxError', [xhr, settings, error || type])  
 //调用ajaxComplete方法  
 ajaxComplete(type, xhr, settings)  
 }  
 // status: "success", "notmodified", "error", "timeout", "abort", "parsererror"  
 //不管成功还是失败最后都会触发全局事件ajaxComplete  
 function ajaxComplete(status, xhr, settings) {  
 var context = settings.context  
 settings.complete.call(context, xhr, status)  
 triggerGlobal(settings, context, 'ajaxComplete', [xhr, settings])  
 ajaxStop(settings)  
 }  
  
 //以上与ajax自定义全局事件有关，deferred对象还是一个疑问点  
  
  
 // Empty function, used as default callback  
 //空函数，用作默认回调  
 function empty() {}  
  
 //下面是ajax的核心，ajax和jsonp的实现  
  
 //动态创建script，实现jsonp  
 $.ajaxJSONP = function(options, deferred){  
 //如果options中无type属性，返回$.ajax(options)的调用结果  
 if (!('type' in options)) return $.ajax(options)  
  
 //callbackName,回调函数名  
 //这里jsonpCallback也可以是一个函数，如果是函数，就取其调用结果  
 //如果jsonpCallback不存在，取'jsonp'+一个累加的jsonpID  
 var _callbackName = options.jsonpCallback,  
 callbackName = ($.isFunction(_callbackName) ?  
 _callbackName() : _callbackName) || ('jsonp' + (++jsonpID)),  
 script = document.createElement('script'),//创建script标签  
 //callbackName是window下的变量，因此，如果两个jsonp请求，  
 //jsonpCallback名称一样，在第一个请求回调还没处理完的时候，调用第二个jsonp，会报错。  
 //window[callbackName]赋值给originalCallback（回调函数），但这里初始化的时候，window[callbackName]其实还是未定义吧  
 originalCallback = window[callbackName],  
 responseData,//响应数据  
 //定义中止请求方法abort  
 abort = function(errorType) {  
 //其实就是script标签触发error事件  
 $(script).triggerHandler('error', errorType || 'abort')  
 },  
 //定义xhr对象，初始只有abort这一个方法，  
 //定义几秒后中止  
 xhr = { abort: abort }, abortTimeout  
  
 //如果deferred对象存在，执行deferred.promise方法  
 if (deferred) deferred.promise(xhr)  
  
 //script添加load和error事件监听  
 $(script).on('load error', function(e, errorType){  
 //清除定时器  
 clearTimeout(abortTimeout)  
 //移除事件，移除script标签，因为已经加载完成  
 $(script).off().remove()  
  
 //如果事件类型为error或者响应数据不存在，调用ajaxError方法  
 if (e.type == 'error' || !responseData) {  
 ajaxError(null, errorType || 'error', xhr, options, deferred)  
 } else {  
 //否则调用ajaxSuccess方法  
 ajaxSuccess(responseData[0], xhr, options, deferred)  
 }  
 //undefined  
 window[callbackName] = originalCallback  
  
 //如果响应数据存在且originalCallback为函数，调用originalCallback执行回调  
 //话说这里if语句不会走进去啊，这到底干嘛  
 if (responseData && $.isFunction(originalCallback))  
 originalCallback(responseData[0])  
  
 //清空  
 originalCallback = responseData = undefined  
 })  
  
 //调用ajaxBeforeSend，如果返回false,调用abort方法来触发error事件  
 if (ajaxBeforeSend(xhr, options) === false) {  
 abort('abort')  
 return xhr  
 }  
  
 //定义window[callbackName]，即回调方法  
 //把所有的参数都赋值给responseData  
 window[callbackName] = function(){  
 responseData = arguments  
 }  
  
 //这个正则匹配?x=?这种格式的字符串，x表示1到多个任意字符  
 //把回调函数名称添加到url中  
 script.src = options.url.replace(/\?(.+)=\?/, '?$1=' + callbackName)  
 //发送请求  
 document.head.appendChild(script)  
  
 //如果选项设了超时事件，添加定时器，超时调用abort  
 if (options.timeout > 0) abortTimeout = setTimeout(function(){  
 abort('timeout')  
 }, options.timeout)  
  
 return xhr  
 }  
  
 //默认ajax配置项  
 $.ajaxSettings = {  
 // Default type of request  
 //默认是get请求  
 type: 'GET',  
 // Callback that is executed before request  
 beforeSend: empty,  
 // Callback that is executed if the request succeeds  
 success: empty,  
 // Callback that is executed the the server drops error  
 error: empty,  
 // Callback that is executed on request complete (both: error and success)  
 complete: empty,  
 // The context for the callbacks  
 //回调上下文默认null的话，其实就是document  
 context: null,  
 // Whether to trigger "global" Ajax events  
 //默认触发全局ajax事件  
 global: true,  
 // Transport  
 xhr: function () {  
 return new window.XMLHttpRequest()  
 },  
 // MIME types mapping  
 // IIS returns Javascript as "application/x-javascript"  
 //datatype所对应content-type  
 accepts: {  
 script: 'text/javascript, application/javascript, application/x-javascript',  
 json:   jsonType,  
 xml:    'application/xml, text/xml',  
 html:   htmlType,  
 text:   'text/plain'  
 },  
 // Whether the request is to another domain  
 crossDomain: false,  
 // Default timeout  
 //超时时间，0表示不超时  
 timeout: 0,  
 // Whether data should be serialized to string  
 //是否需要将数据转为序列化的字符串  
 processData: true,  
 //缓存get请求的相应数据  
 // Whether the browser should be allowed to cache GET responses  
 cache: true  
 }  
  
 //根据mime值获取其dataType  
 function mimeToDataType(mime) {  
 //split的第二个参数limit，这个挺少见，表示截取数组的前几个item  
 //这里很奇怪，截取到2项但是取的时候又只取第一项  
 if (mime) mime = mime.split(';', 2)[0]  
 //如果mime为'text/html'，返回'html'  
 //如果mime为'application/json'，返回'json'  
 //如果mime匹配script类型的正则，返回'script'  
 //如果mime匹配xml类型的正则，返回'xml'  
 //否则返回'test'  
 return mime && ( mime == htmlType ? 'html' :  
 mime == jsonType ? 'json' :  
 scriptTypeRE.test(mime) ? 'script' :  
 xmlTypeRE.test(mime) && 'xml' ) || 'text'  
 }  
  
 //添加查询参数  
 function appendQuery(url, query) {  
 //如果query为空直接返回url  
 if (query == '') return url  
 //否则字符串为url+'&'+query  
 //正则匹配1-2个的&或者?，将其替换为?,即??,&&,?&,?&  
 return (url + '&' + query).replace(/[&?]{1,2}/, '?')  
 }  
  
 // serialize payload and append it to the URL for GET requests  
 function serializeData(options) {  
 //序列化为true且data存在且data的类型不为字符串时  
 //调用$.params将options.data转为序列化字符串  
 if (options.processData && options.data && $.type(options.data) != "string")  
 options.data = $.param(options.data, options.traditional)  
 //如果data存在且 type的类型为GET或dataType为jsonp  
 if (options.data && (!options.type || options.type.toUpperCase() == 'GET' || 'jsonp' == options.dataType))  
 //将data的参数添加到url上，options.data设为undefined  
 options.url = appendQuery(options.url, options.data), options.data = undefined  
 }  
  
 $.ajax = function(options){  
 var settings = $.extend({}, options || {}),//用户ajax配置  
 deferred = $.Deferred && $.Deferred(),//deferred对象  
 urlAnchor, hashIndex  
  
 //如果用户配置值不存在，取默认配置赋值给用户配置   
 for (key in $.ajaxSettings) if (settings[key] === undefined) settings[key] = $.ajaxSettings[key]  
  
 //调用全局ajax自定义事件ajaxStart  
 ajaxStart(settings)  
  
 //如果跨域为假  
 if (!settings.crossDomain) {  
 urlAnchor = document.createElement('a')//创建a元素  
 urlAnchor.href = settings.url//设置其href  
  
 // cleans up URL for .href (IE only), see https://github.com/madrobby/zepto/pull/1049  
 //ie会对window.location.host默认添加端口号，但是对于链接a.host默认不添加  
 //这句话它的作用是为a.host属性添加端口号  
 urlAnchor.href = urlAnchor.href  
  
 //判断当前页面的host与ajax请求的host是否相等，如果不相等，说明跨域  
 //协议，域名，端口  
 settings.crossDomain = (originAnchor.protocol + '//' + originAnchor.host) !== (urlAnchor.protocol + '//' + urlAnchor.host)  
 }  
  
 //如果url为假值，设置其默认值为当前页面  
 if (!settings.url) settings.url = window.location.toString()  
 //查找url中的hash，如果存在，url截取hash前面的部分  
 if ((hashIndex = settings.url.indexOf('#')) > -1) settings.url = settings.url.slice(0, hashIndex)  
    
 //把ajax配置项转为序列化字符串  
 serializeData(settings)  
  
 var dataType = settings.dataType,   
 //如果url匹配?一个以上任意字符=?  
 hasPlaceholder = /\?.+=\?/.test(settings.url)//有占位符  
  
 if (hasPlaceholder) dataType = 'jsonp'//如果有占位符，数据类型改为jsonp  
  
 //满足表达式1，settings.cache === false，不缓存  
 //或者满足表达式2，(!options || options.cache !== true) &&('script' == dataType || 'jsonp' == dataType)  
 //=>用户配置不存在或用户缓存配置不为真且 数据类型为script或jsonp时  
 //说明如果用户不设cache为true的话，jsonp是默认不缓存的  
 if (settings.cache === false || (  
 (!options || options.cache !== true) &&  
 ('script' == dataType || 'jsonp' == dataType)  
 ))  
 //添加时间戳后缀  
 settings.url = appendQuery(settings.url, '_=' + Date.now())  
  
 //如果是jsonp，内部调用$.ajaxJSONP  
 if ('jsonp' == dataType) {  
 if (!hasPlaceholder)//如果没有占位符  
 //settings.url若存在，第二个参数查询参数的值为settings.jsonp加'=?'占位符  
 //settings.url若不存在，若settings.jsonp === false，第二个参数为'',若不为false，第二个参数为'callback=?'  
 //appendQuery用户添加查询参数  
 settings.url = appendQuery(settings.url,  
 settings.jsonp ? (settings.jsonp + '=?') : settings.jsonp === false ? '' : 'callback=?')  
 return $.ajaxJSONP(settings, deferred)  
 }  
  
 var mime = settings.accepts[dataType],//获取mime类型  
 headers = { },//存储请求头信息  
 //此方法用于将header信息添加到headers对象中  
 setHeader = function(name, value) { headers[name.toLowerCase()] = [name, value] },  
  
 //正则匹配以一个或多个“字母、数字、-”开头的字符，且后面为“://”的字符串  
 //中划线的情况比如view-source协议  
 //获取protocol，优先从url中获取，如果不存在取location.protocol  
 protocol = /^([\w-]+:)\/\//.test(settings.url) ? RegExp.$1 : window.location.protocol,  
 xhr = settings.xhr(),//XMLHttpRequest实例  
 nativeSetHeader = xhr.setRequestHeader,//原生的设置请求头方法  
 abortTimeout  
  
 //如果deferred对象存在，调用promise(xhr)  
 if (deferred) deferred.promise(xhr)  
  
 //如果没有跨域，设置请求x-requested-with内容XMLHttpRequest,即异步请求  
 //如果设置其值为null，则是同步请求  
 if (!settings.crossDomain) setHeader('X-Requested-With', 'XMLHttpRequest')  
 setHeader('Accept', mime || '*/*')//客户端能接收的资源类型  
 //如果mime存在，逗号分割成数组取其第一项赋给mime  
 if (mime = settings.mimeType || mime) {  
 if (mime.indexOf(',') > -1) mime = mime.split(',', 2)[0]  
 //overrideMimeType是xhr level 1就有的方法，所以浏览器兼容性良好。  
 //这个方法的作用就是用来重写response的content-type，这样做有什么意义呢？  
 //比如：server 端给客户端返回了一份document或者是 xml文档，  
 //我们希望最终通过xhr.response拿到的就是一个DOM对象，那么就可以用  
 //xhr.overrideMimeType('text/xml; charset = utf-8')来实现。  
 //这篇文章很值得一看：https://segmentfault.com/a/1190000004322487  
 xhr.overrideMimeType && xhr.overrideMimeType(mime)  
 }  
 //如果settings.contentType为真值  
 //或者settings.contentType不为false，且settings.data存在，且settings.type不为get时  
  
 //Content-Type: 内容类型  指定响应的 HTTP内容类型。决定浏览器将以什么形式、什么编码读取这个文件.  如果未指定 ContentType，默认为TEXT/HTML。  
 //application/x-www-form-urlencoded：是一种编码格式，  
 //窗体数据被编码为名称/值对，是标准的编码格式。  
  
 //当action为get时候，浏览器用x-www-form-urlencoded的编码方式把  
 //form数据转换成一个字串（name1=value1&name2=value2...），  
 //然后把这个字串append到url后面，用?分割，加载这个新的url。   
 //当action为post时候，浏览器把form数据封装到http body中，然后发送到server  
  
 //如果有 type=file的话，需要设为multipart/form-data了。  
 //浏览器会把整个表单以控件为单位分割，并为每个部分加上   
 //Content-Disposition(form-data或者file),  
 //Content-Type(默认为text/plain),name(控件 name)等信息，  
 //并加上分割符(boundary)。  
  
 if (settings.contentType || (settings.contentType !== false && settings.data && settings.type.toUpperCase() != 'GET'))  
 setHeader('Content-Type',settings.contentType || 'application/x-www-form-urlencoded')  
  
 //遍历设置请求头  
 if (settings.headers) for (name in settings.headers) setHeader(name, settings.headers[name])  
 xhr.setRequestHeader = setHeader  
  
 // readyState的值  
 // 0：请求未初始化（还没有调用 open()）。  
 // 1：请求已经建立，但是还没有发送（还没有调用 send()）。  
 // 2：请求已发送，正在处理中（通常现在可以从响应中获取内容头）。  
 // 3：请求在处理中；通常响应中已有部分数据可用了，但是服务器还没有完成响应的生成。  
 // 4：响应已完成；可以获取并使用服务器的响应了。  
 xhr.onreadystatechange = function(){  
 if (xhr.readyState == 4) {  
 //防止再次调用  
 xhr.onreadystatechange = empty  
 //清除超时定时器  
 clearTimeout(abortTimeout)  
 var result, error = false  
 //200-300之间表示成功  
 //304重定向  
 //xhr.status == 0 && protocol == 'file:'  表示打开的本地文件  
 if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304 || (xhr.status == 0 && protocol == 'file:')) {  
 //获取mime类型  
 dataType = dataType || mimeToDataType(settings.mimeType || xhr.getResponseHeader('content-type'))  
  
 //arraybuffer,blob都是二进制对象  
 if (xhr.responseType == 'arraybuffer' || xhr.responseType == 'blob')  
 result = xhr.response  
 else {  
 result = xhr.responseText  
  
 try {  
 // http://perfectionkills.com/global-eval-what-are-the-options/  
 //如果响应数据类型为script，全局调用  
 if (dataType == 'script')    (1,eval)(result)  
 else if (dataType == 'xml')  result = xhr.responseXML  
  
 else if (dataType == 'json') result = blankRE.test(result) ? null : $.parseJSON(result)  
 } catch (e) { error = e }  
  
 //以上都不满足，调用ajaxError  
 if (error) return ajaxError(error, 'parsererror', xhr, settings, deferred)  
 }  
  
 ajaxSuccess(result, xhr, settings, deferred)  
 } else {  
 ajaxError(xhr.statusText || null, xhr.status ? 'error' : 'abort', xhr, settings, deferred)  
 }  
 }  
 }  
  
 //如果请求前回调返回false  
 if (ajaxBeforeSend(xhr, settings) === false) {  
 //中断ajax  
 xhr.abort()  
 ajaxError(null, 'abort', xhr, settings, deferred)  
 return xhr  
 }  
 //xhrFields: {withCredentials: true }  
 //支持跨域发送cookie  
 if (settings.xhrFields) for (name in settings.xhrFields) xhr[name] = settings.xhrFields[name]  
  
 //如果没有配置，默认异步  
 var async = 'async' in settings ? settings.async : true  
  
 //创建请求  
 xhr.open(settings.type, settings.url, async, settings.username, settings.password)  
 //设置请求头  
 for (name in headers) nativeSetHeader.apply(xhr, headers[name])  
 //超时处理  
 if (settings.timeout > 0) abortTimeout = setTimeout(function(){  
 xhr.onreadystatechange = empty  
 xhr.abort()  
 ajaxError(null, 'timeout', xhr, settings, deferred)  
 }, settings.timeout)  
  
 // avoid sending empty string (#319)  
 //发送请求  
 xhr.send(settings.data ? settings.data : null)  
 return xhr  
 }  
  
 // handle optional data/success arguments  
 function parseArguments(url, data, success, dataType) {  
 //如果第二个参数是函数，认为它实际表示的是回调success，  
 //第三个参数表示的是datatype,而data认为它没有传  
 if ($.isFunction(data)) dataType = success, success = data, data = undefined  
  
 //如果第三参数不是函数，认为它实际表示的是dataType，而success认为没有传  
 if (!$.isFunction(success)) dataType = success, success = undefined  
  
 //最后转成对象返回  
 return {  
 url: url  
 , data: data  
 , success: success  
 , dataType: dataType  
 }  
 }  
  
 //内部调用parseArguments获取ajax用户配置项，然后调用$.ajax  
 $.get = function(/* url, data, success, dataType */){  
 return $.ajax(parseArguments.apply(null, arguments))  
 }  
  
 //设置type为'post',其他同$.get  
 $.post = function(/* url, data, success, dataType */){  
 var options = parseArguments.apply(null, arguments)  
 options.type = 'POST'  
 return $.ajax(options)  
 }  
  
 //同$.get,只不过dataType改为'json'  
 $.getJSON = function(/* url, data, success */){  
 var options = parseArguments.apply(null, arguments)  
 options.dataType = 'json'  
 return $.ajax(options)  
 }  
  
 //ajax获取html内容,作为this的html内容  
 //如果url字符串中带了选择器，比如：'http://www.a.com #test'  
 //那么响应返回的html内容的script会被删除，最后this的html内容是该html片段中匹配了选择器的部分  
 $.fn.load = function(url, data, success){  
 //如果为空集合，返回  
 if (!this.length) return this  
  
 var self = this,   
 parts = url.split(/\s/), //将url用空格分隔，转为数组  
 selector,  
 options = parseArguments(url, data, success),//获取ajax用户配置  
 callback = options.success//存储原success  
  
 //如果url数量大于1，options.url取parts第一个元素，selector取parts第二个元素  
 if (parts.length > 1) options.url = parts[0], selector = parts[1]  
  
 //重写success  
 options.success = function(response){  
 //如果selector 存在，创建div元素，  
 //response的内容过滤掉script标签，而后添加到div元素中  
 //之后div元素中查找选择器selector,找到的内容作为this的html  
 //如果selector不存在，this的html即为response  
 self.html(selector ?  
 $('').html(response.replace(rscript, "")).find(selector)  
 : response)  
 //最后调用回调  
 callback && callback.apply(self, arguments)  
 }  
 $.ajax(options)  
 return this  
 }  
  
 //下面是序列化相关  
  
 var escape = encodeURIComponent  
 //serializedData内部调用$.param，$.param内部调用serialize  
  
 //将每一个对象中的属性转为编码后的‘xxx=yyy’或者‘xxx[yyy]=zzz’  
 //这种形式然后存入到params数组中  
 //这个方法是被$.param调用的，调用时会传一个params空数组  
 //params是带add方法的空数组  
 //traditional表示是否使用传统浅层方式序列化  
 //scope,是否递归  
 //√  
 function serialize(params, obj, traditional, scope){  
 var type,   
 array = $.isArray(obj), //obj为数组，则array为true  
 hash = $.isPlainObject(obj)//obj为对象，则hash为true  
  
 $.each(obj, function(key, value) {  
 type = $.type(value)//获取value类型  
  
 //参数scope传递的是上一次递归调用serialize时的key变量，  
 //如果当前是第一次调用，scope是未定义的  
 //如果scope为真  
 //如果传统浅层序列化真，则key=scope  
 //如果深层序列化（运算符优先级||高于三目）,要加[]  
 //=>如果hash为真或者类型为对象或者类型为数组，则key=scope+[key]  
 //=>否则key=scope+['']  
 //scope不太好理解，举个例子：举个例子obj为{type:{yong:16}}，且是深层序列化，  
 //遍历时key为type,value为{yong:16}  
 //递归调用serialize时，形参scope的值为'type'。obj传的是{yong:16},traditional的值沿用以前，依旧是深层系列化  
 //第二次执行serialize方法，遍历时新的key为yong,value为16，  
 //此时scope有值了，存的是'type'，且依旧是深层系列化，  
 //所以新的key=scope+'['+key+']',即'type[young]'  
 //最后调用$.params.add添加的一条记录是'type%5Byoung%5D=16'  
 if (scope) key = traditional ? scope :  
 scope + '[' + (hash || type == 'object' || type == 'array' ? key : '') + ']'  
  
 // handle data in serializeArray() format  
 //如果不递归且obj类型是array,  
 //调用params.add方法，value.name下标, value.value值  
 //这里有name属性和value属性，可以推断obj为表单数据  
 //这样就把它转换为['xxx=yyy']  
 if (!scope && array) params.add(value.name, value.value)  
  
 // recurse into nested objects  
 //如果不递归且array不为真  
 //=>且遍历时当前value的类型是数组  
 //=>或者需要深层序列化且当前value类型为对象  
 //还需要调用serialize,这时候需要传第四个参数scope，  
 //它的值就是遍历时obj的当前属性名key,是个字符串~~  
 else if (type == "array" || (!traditional && type == "object"))  
 serialize(params, value, traditional, key)  
  
 //如果不递归且array不为真  
 //且满足条件：value类型是对象且是浅层序列化  
 //或者满足条件：value类型不是对象也不是数组  
 //调用params.add  
 else params.add(key, value)  
 })  
 }  
  
 //将对象转为序列化字符串后返回  
 //第二个参数表示深层序列化  
 $.param = function(obj, traditional){  
 var params = []  
 //add方法用于将key和value经过encodeURIComponent编码后以key=value的形式添加到params数组中  
 params.add = function(key, value) {  
 //如果value是函数，value重新赋值为其自身的调用结果  
 if ($.isFunction(value)) value = value()  
 //如果是null，value为空  
 if (value == null) value = ""  
 //对key和value encodeURIComponent编码  
 this.push(escape(key) + '=' + escape(value))  
 }  
 serialize(params, obj, traditional)  
 //将数组转为字符串，用&拼接，%20表示空格，这里替换成+  
 return params.join('&').replace(/%20/g, '+')  
 }  
})(Zepto)  

```

