title: h5 meta
date: 2015-12-15 11:23:55
tags:
 - 移动端
---
#### apple-mobile-web-app-capable

`<meta content="yes" name="apple-mobile-web-app-capable" />`
设置Web应用是否以全屏模式运行。

+ 语法：
<meta name="apple-mobile-web-app-capable" content="yes">
+ 说明：
如果content设置为yes，Web应用会以全屏模式运行，反之，则不会。content的默认值是no，表示正常显示。你可以通过只读属性window.navigator.standalone来确定网页是否以全屏模式显示。
+ 兼容性：
iOS 2.1 +

#### Meta标签中的format-detection属性及含义
```	
<meta content="telephone=no" name="format-detection" />
<meta content="email=no" name="format-detection" />```
format-detection翻译成中文的意思是“格式检测”，顾名思义，它是用来检测html里的一些格式的，那关于meta的format-detection属性主要是有以下几个设置：
+ meta name="format-detection" content="telephone=no"
+ meta name="format-detection" content="email=no"
+ meta name="format-detection" content="adress=no" 

也可以连写：meta name="format-detection" content="telephone=no,email=no,adress=no"
下面具体说下每个设置的作用：
##### 一、telephone
你明明写的一串数字没加链接样式，而iPhone会自动把你这个文字加链接样式、并且点击这个数字还会自动拨号！想去掉这个拨号链接该如何操作呢？这时我们的meta又该大显神通了，代码如下：
+ telephone=no就禁止了把数字转化为拨号链接！
+ telephone=yes就开启了把数字转化为拨号链接，要开启转化功能，这个meta就不用写了,在默认是情况下就是开启！
##### 二、email
告诉设备不识别邮箱，点击之后不自动发送

+ email=no禁止作为邮箱地址！
+ email=yes就开启了把文字默认为邮箱地址，这个meta就不用写了,在默认是情况下就是开启！
##### 三、adress
+ adress=no禁止跳转至地图！
+ adress=yes就开启了点击地址直接跳转至地图的功能,在默认是情况下就是开启！

####  apple-mobile-web-app-status-bar-style
`<meta content="black" name="apple-mobile-web-app-status-bar-style">`
改变顶部状态条的颜色
+ 说明：

在 web app 应用下状态条（屏幕顶部条）的颜色；
默认值为 default（白色），可以定为 black（黑色）和 black-translucent（灰色半透明）；
+ 注意：若值为“black-translucent”将会占据页面位置，浮在页面上方（会覆盖页面 20px 高度 iphone4 和 itouch4 的 Retina 屏幕为 40px ）。

#### wap-font-scale

uc浏览器判断到页面上文字居多时，会自动放大字体优化移动用户体验。添加以下头部可以禁用掉该优化
`<meta name="wap-font-scale" content="no">`