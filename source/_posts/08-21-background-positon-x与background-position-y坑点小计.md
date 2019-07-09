title: background-positon-x与background-position-y坑点小计
date: 2016-08-21
tags: 
 - css
---


有句话说的好啊，“不要以为你以为的就是你以为的”，非常适合background-position这个坑，因为它太常用。

```
background-position: 10px 20px;  

```

```
background-position-x: 10px;  
background-position-y: 20px;  

```

很少会有人认为上面这两种写法有什么区别，不就是下面一种分开来写嘛。

然而[can i use](http://caniuse.com/#search=background-position-x%20%26%20background-position-y)告诉你，firefox就是这么傲娇，当然还有我们国内不怎么关注的opera。

### 纳尼？？？

我们来看下兼容性表格

![兼容性表格](http://ysha-01.oss-cn-shanghai.aliyuncs.com/QQ20160821233903.png)

这里显示firefox48及以下，firefox for Android 47，Opera mini 目前所有版本，都不支持`background-position-x`/`background-position-x`,而Opera mobile直到37才开始支持。

