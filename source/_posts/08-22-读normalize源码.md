title: 读normalize源码
date: 2016-08-22
tags: 
 - css
---


公司有一个项目用了normailize v3.0，于是看了下源码，顺便与4.2.0版本做了对比。

_注：这里只是对比4.2.0与3.0的不同点，并不是说这些变动都是在4.2.0中发生的。_

### 防止横屏时重置字体大小

```
html {  
 font-family: sans-serif;  
 -ms-text-size-adjust: 100%;  
 -webkit-text-size-adjust: 100%;  
}  

```

v4.2.0增加了line-height的统一设置：line-height: 1.15;

### 移除border默认外边距

```
body {  
 margin: 0;  
}  

```

### ie8-9中一些h5标签的display未正确设置为block

```
article,  
aside,  
details,  
figcaption,  
figure,  
footer,  
header,  
hgroup,  
main,  
nav,  
section,  
summary {  
 display: block;  
}  

```

4.2.0做了下拆分，然后summary的display改成了list-item：

```
article,  
aside,  
footer,  
header,  
nav,  
section {  
 display: block;  
}  
  
figcaption,  
figure,  
main { /* 1 */  
 display: block;  
}  
  
details, /* 1 */  
menu {  
 display: block;  
}  
  
summary {  
 display: list-item;  
}  

```

### 这几个标签在IE8-9中的display未定义，progress在Chrome-firefox-opera中不是基线对齐。

```
audio,  
canvas,  
progress,  
video {  
 display: inline-block;  
 vertical-align: baseline;  
}  

```

4.2.0也做了拆分，因为只有progress需要修正vertical-align,它被单独拎出来。

```
audio,  
video {  
 display: inline-block;  
}  
  
canvas {  
 display: inline-block;  
}  
  
progress {  
 display: inline-block; /* 1 */  
 vertical-align: baseline; /* 2 */  
}  

```

### audio不支持controls属性的默认不显示

```
audio:not[controls] {  
 display: none;  
 height: 0;  
}  

```

### 修复IE8-9-hidden属性不起作用的问题以及在IE-safari，firefox22-中隐藏template元素

```
[hidden],  
template {  
 display: none;  
}  

```

4.2.0也拆分了,template是针对所有ie,[hidden]是针对ie10-

```
template {  
 display: none;  
}  
  
[hidden] {  
 display: none;  
}  

```

### ie10鼠标点击时a链接会有灰色背景

```
a {  
background: transparent;  
}  

```

4.2.0新增了对装饰线的设置

```
a {  
 background-color: transparent; /* 1 */  
 -webkit-text-decoration-skip: objects; /* 2 */  
}  

```

关于装饰线查看张鑫旭的这篇文章：[http://www.zhangxinxu.com/wordpress/2015/06/know-css-text-decoration-style-color-ship/](http://www.zhangxinxu.com/wordpress/2015/06/know-css-text-decoration-style-color-ship/)

### 去掉a链接点击时或者鼠标划过时的outline

```
a:hover,  
a:active {  
 outline: 0;  
}  

```

4.2.0优化成`outline-width: 0;`

### abbr支持title属性的说明是完整版。但abbr在ie8-9、safari、chrome中没有下划线，而firefox有。

```
abbr[title] {  
 border-bottom: 1px dotted;  
}  

```

4.2.0规则改了,移除firefox39-的下划线，所有浏览器加`text-decoration`修饰

```
abbr[title] {  
 border-bottom: none; /* 1 */  
 text-decoration: underline; /* 2 */  
 text-decoration: underline dotted; /* 2 */  
}  

```

### firefox4-safari-5-chrome中。默认的粗体不一致

```
b,  
strong {  
 font-weight: bold;  
}  

```

4.2.0都改成了bolder?然后我不明白下面第一条设为inherit是在针对Safari 6做什么？

```
/**  
 * Prevent the duplicate application of `bolder` by the next rule in Safari 6.  
 */  
b,  
strong {  
 font-weight: inherit;  
}  
  
/**  
 * Add the correct font weight in Chrome, Edge, and Safari.  
 */  
  
b,  
strong {  
 font-weight: bolder;  
}  

```

### dfn标签在safari-5和chrome中未被设置为斜体，这里做了修正

```
dfn {  
 font-style: italic;  
}  

```

### 对h1字体大小和上下间距的修正。h1字体大小，session与article标签中的h1的上下间距，它们在firefox4-。safari-5，chrome中不一致。

```
h1 {  
 font-size: 2em;  
 margin: 0.67em 0;  
}  

```

### ie8-9中mark样式设置，黑色字体黄色背景

```
mark {  
 color: #000;  
 background: #ff0;  
}  

```

### small字体在各个浏览器中大小不一致，这里统一为80

```
small {  
 font-size: 80%;  
}  

```

### 防止所有浏览器中的sub和sup影响行高

sup和sub两个标签是用来表示上标和下标，据HTML标准中对small，sub和sup的大小要求都是smaller，但是如上所示normalize.css给small设的是80%，而sub和sup却是75%，所以为了保持一致，且不影响其他元素的行高，把两者的line-height设为0，然后设置它的垂直以baseline开始，设置top和bottom手动设置两者偏移量

```
sub,  
sup {  
 font-size: 75%;  
 line-height: 0;  
 position: relative;  
 vertical-align: baseline;  
}  
  
sup {  
 top: -0.5em;  
}  
  
sub {  
 bottom: -0.25em;  
}  

```

### 移除ie8-9中a标签内img的边框

```
img {  
 border: 0;  
}  

```

4.2.0优化成：

```
img {  
 border-style: none;  
}  

```

### 修复-IE9-中的overflow的怪异表现

```
svg:not(:root) {  
 overflow: hidden;  
}  

```

### 修复-IE-8-9、Safari中-figure元素的默认margin失效

```
figure {  
 margin: 1em 40px;  
}  

```

### firefox的hr标签与其他浏览器不一致。firefox-hr默认高度2px-box-sizing为border-box

```
hr {  
 -moz-box-sizing: content-box;  
 box-sizing: content-box;  
 height: 0;  
}  

```

4.2中增加了ie与edge的溢出显示,然后把moz私有前缀去掉了。

```
hr {  
 box-sizing: content-box;   
 height: 0;  
 overflow: visible;  
}  

```

### 默认设置overflow为溢出滚动，不设置的话是溢出显示

```
pre {  
 overflow: auto;  
}  

```

4.2.0中这条重置被移除

### 修复-Safari-5-和-Chrome-中-一些冷门h5标签的奇怪的字体设置，统一字体样式

```
code,  
kbd,  
pre,  
samp {  
 font-family: monospace, monospace;  
 font-size: 1em;  
}  

```

4.2.0把pre单独拎出来不知道为么。

### 修正一些表单元素

1.修正所有浏览器中颜色不继承的问题  
2.修正所有浏览器中字体不继承的问题  
3.修正 Firefox 3+， Safari5 和 Chrome 中外边距不同的问题

有一些浏览器会把form表单中的一些元素 textarea，text，button，select 中的字体和字体颜色默认会设置成用户的字体或者是浏览器的字体，并不会从父元素继承，所以这里重置了这些元素的默认样式。

```
button,  
input,  
optgroup,  
select,  
textarea {  
 color: inherit; /* 1 */  
 font: inherit; /* 2 */  
 margin: 0; /* 3 */  
}  

```

4.2.0改动如下：把color的继承干掉了，color和font的inherit值改成显示指定，是否inherit有bug?

```
button,  
input,  
optgroup,  
select,  
textarea {  
 font-family: sans-serif; /* 1 */  
 font-size: 100%; /* 1 */  
 line-height: 1.15; /* 1 */  
 margin: 0; /* 2 */  
}  

```

### 修正ie8-9-10中，按钮溢出隐藏的问题，改成溢出显示

```
button {  
 overflow: visible;  
}  

```

4.2.0新增了input也为溢出隐藏

```
button,  
input { /* 1 */  
 overflow: visible;  
}  

```

### 统一各浏览器text-transform不会继承的问题

```
button,  
select {  
 text-transform: none;  
}  

```

### 按钮bug修复，统一设置光标样式

1.避免 Android 4.0.* 中的 WebKit bug ，该bug会破坏原生的audio和video的控制器  
2.更正 iOS 中无法设置可点击的input的问题  
3.统一其他类型的input的光标样式

```
button,  
html input[type="button"], /* 1 */  
input[type="reset"],  
input[type="submit"] {  
 -webkit-appearance: button; /* 2 */  
 cursor: pointer; /* 3 */  
}  

```

4.2.0把光标重置移除了：

```
button,  
html [type="button"], /* 1 */  
[type="reset"],  
[type="submit"] {  
 -webkit-appearance: button; /* 2 */  
}  

```

### 重置disabled按钮的光标样式

```
button[disabled],  
html input[disabled] {  
 cursor: default;  
}  

```

4.2.0中这条重置被移除

### 移除firefox4-input与button元素的padding与border

```
button::-moz-focus-inner,  
input::-moz-focus-inner {  
 border: 0;  
 padding: 0;  
}  

```

4.2.0做了优化调整

```
button::-moz-focus-inner,  
[type="button"]::-moz-focus-inner,  
[type="reset"]::-moz-focus-inner,  
[type="submit"]::-moz-focus-inner {  
 border-style: none;  
 padding: 0;  
}  

```

然后它又把最先前去掉的outline加回去了。。

```
button:-moz-focusring,  
[type="button"]:-moz-focusring,  
[type="reset"]:-moz-focusring,  
[type="submit"]:-moz-focusring {  
 outline: 1px dotted ButtonText;  
}  

```

### 修复firefox4-input元素line-height设置了-important的问题-normal-important

```
input {  
 line-height: normal;  
}  

```

4.2.0中这条重置被移除

### 对checkbox和radio设置统一的box-sizng和padding

firefox不支持对这两个元素设置box-sizng,padding和width。而ie8/9/10的这两个元素boz-sizng为content-box（ie5.5盒模型），这里统一改成border-box。  
移除ie8/9/10中多余的padding

```
input[type="checkbox"],  
input[type="radio"] {  
 box-sizing: border-box; /* 1 */  
 padding: 0; /* 2 */  
}  

```

### 修复chrome下number类型的input，上下两个箭头处于焦点时鼠标不是箭头的问题。

```
input[type="number"]::-webkit-inner-spin-button,  
input[type="number"]::-webkit-outer-spin-button {  
 height: auto;  
}  

```

### 修正Safari-5-和-Chrome中appearance被设置为searchfield-box-sizing被设置为border-box的问题

appearance用于控制外观，可选的值查看MDN:[https://developer.mozilla.org/en-US/docs/Web/CSS/-moz-appearance](https://developer.mozilla.org/en-US/docs/Web/CSS/-moz-appearance)

```
input[type="search"] {  
 -webkit-appearance: textfield; /* 1 */  
 -moz-box-sizing: content-box;  
 -webkit-box-sizing: content-box; /* 2 */  
 box-sizing: content-box;  
}  

```

4.2.0：修正了safari的`outline`偏移，去除了`box-sizing`的重置

```
[type="search"] {  
 -webkit-appearance: textfield; /* 1 */  
 outline-offset: -2px; /* 2 */  
}  

```

然后移除了OS X系统中chrome与safari的内边距与取消按钮

```
/**  
 * Remove the inner padding and cancel buttons in Chrome and Safari on OS X.  
 */  
  
[type="search"]::-webkit-search-cancel-button,  
[type="search"]::-webkit-search-decoration {  
 -webkit-appearance: none;  
}  

```

### 设置fildset统一样式

```
fieldset {  
 border: 1px solid #c0c0c0;  
 margin: 0 2px;  
 padding: 0.35em 0.625em 0.75em;  
}  

```

### 修复ie8-9中legend标签不继承color属性的问题，清除默认padding

color属性无法继承是用设置border为0解决。这个我一开始还以为是作者注释写错→ →。

```
legend {  
 border: 0; /* 1 */  
 padding: 0; /* 2 */  
}  

```

4.2.0 中，颜色继承的解决方案改成了color: inherit，也修正了IE与Edeg中的文本包含bug

```
/**  
 * 1\. Correct the text wrapping in Edge and IE.  
 * 2\. Correct the color inheritance from `fieldset` elements in IE.  
 * 3\. Remove the padding so developers are not caught out when they zero out  
 *    `fieldset` elements in all browsers.  
 */  
  
legend {  
 box-sizing: border-box; /* 1 */  
 color: inherit; /* 2 */  
 display: table; /* 1 */  
 max-width: 100%; /* 1 */  
 padding: 0; /* 3 */  
 white-space: normal; /* 1 */  
}  

```

### 移除ie8-9中-textarea默认的垂直滚动条

```
textarea {  
 overflow: auto;  
}  

```

### 统一设置optgroup为bold

```
optgroup {  
 font-weight: bold;  
}  

```

4.2.0中这条重置被移除

### 表格边框、padding重置

```
table {  
 border-collapse: collapse;  
 border-spacing: 0;  
}  
  
td,  
th {  
 padding: 0;  
}  

```

4.2.0中这条重置被移除

* * *

###总结

reset已经过时，对比4.2与3.0也可以发现，4.2移除了很多样式重置的代码，normalize越来越趋向于bug修复，正如其名，让浏览器正常显示。

在性能上，4.2比3.0也更加注意，避免了使用不必要的缩写属性。比如`border: 0`改成`border-style: none;`

