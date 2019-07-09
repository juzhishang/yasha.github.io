---
title: 关于css权重
date: 2018-08-23 20:53:50
tags:
 - css
---

记录一下今天面试的知识点，一个是css优先级、一个是css权重，以及css权重的越级问题

---

### 优先级

从高到低依次排列如下：

+ !important。ie6不支持该属性。
+ 行内样式
+ id选择器
+ 类选择器、属性选择器、伪类选择器定义
+ 标签选择器、伪元素选择器
+ 通配选择器`*`

---

### css权重

我觉得最好的文章应该是这篇：http://www.w3cplus.com/css/css-specificity-things-you-should-know.html


![截图](https://ysha-01.oss-cn-shanghai.aliyuncs.com/FireShot%20Capture%2010%20-%20%E4%BD%A0%E5%BA%94%E8%AF%A5%E7%9F%A5%E9%81%93%E7%9A%84%E4%B8%80%E4%BA%9B%E4%BA%8B%E6%83%85%E2%80%94%E2%80%94CSS%E6%9D%83%E9%87%8D_CSS%E9%80%89%E6%8B%A9%E5%99%A8%20%E6%95%99%E7%A8%8B_w3cplus_%20-%20http___www.w3cplus.com_css_css-spe.png?x-oss-process=style/origin-compress)

---

### 权重越权问题

>早期是有一些浏览器存在越权问题，比如opera 256个class选择器就可以覆盖掉一个id选择器，而在chrome30之前是65536个不过现在权重的计算方式已经变了，不管多少个class都无法覆盖id。

——感谢我前同事的答案

下面是sf上的回答：

![截图](https://ysha-01.oss-cn-shanghai.aliyuncs.com/QQ20180823-222206%402x.png?x-oss-process=style/origin-compress)


综上总结，当前已经不会存在权重越级问题了～