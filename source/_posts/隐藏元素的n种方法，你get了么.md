---
title: 隐藏元素的n种方法，你get了么
date: 2017-06-18 15:55:10
tags:
 - css
---

这是一篇拾遗的文章，隐藏元素的方法很多，但是随着css的发展，我们错过了一些黑科技，（微笑）

先来看css2以前的时代，我们怎么处理隐藏

---

+ 1、设置高宽，不怎么灵巧的方法，打个补丁吧

```
width: 0;
height: 0;
```

或

```
max-width: 0;
height: 0;
```

如果这个元素没有子元素，通常上面的代码就可以了，如果有文字，不妨加上一句font-size:0;当然如果子元素上设置了字体大小，或者子元素有图片，（这里都指正常定位的子元素），还要再加一个overflow: hidden;。
但如果子元素应用了position:absolute/fixed，这种方法隐藏就失效了。
+ 2&3、我是透明的，看不见看不见

```
opacity: 0;
```
或者

```
  font-size: 0;
  background-color:transparent;
```
虽然是透明，看不见但是摸得着哦。因此，如果要隐藏的元素上有事件绑定，用这个方法就不合适了。

+ 4、绝对定位把元素移出可视区

```
position: absolute;
left: -9999px;
```
这种方法吧，用的还是很多的，比如视差滚动页面。

+ 5、把文字移出可视区

```
text-indent: -9999px;
```
场景比如站点logo,你以为你看到的只是logo,实际上为了对搜索引擎友好，logo的旁边是有文字的。


+ 6、z-index不常用的黑科技

```
<div class="box">
  <div class="d1">1</div>
  2
</div>
```

```
.box {
  position: relative;
  background: #000;
  color: #fff;
}

.d1 {
  position:absolute;
  left: 0;
  top: 0;
  z-index: -1;
}
```
在这个例子中父元素相对定位，除了d1绝对定位外，父元素还有非定位的元素，即文本2。现在想要d1隐藏起来，只需设置d1的层级在父元素之下，因为默认父元素的层级是0，所以这里设置为－1。

+ 7&8、display: none 与 visibility:hidden，分清楚了么

它们之间有几点不同。

display: 不占据空间，会重新渲染，设置父元素为display: none，则子元素也无法显示
visibility: 占据空间，不会重新渲染，设置父元素为visibility: hidden;子元素可以通过修改visibility: visible;重新显示。

+ 9、zoom缩到无限小

```
zoom: 0.0001;
```
这个属性其实挺奇怪的，设为0又恢复成了1倍大小，(笑cry)

---

下面来看css3的方法

+ 10&11&12 、变形记，这很容易

```
transform: scale(0)
```

```
transform: trans
```
