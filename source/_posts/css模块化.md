---
title: css模块化
date: 2018-09-06 19:20:10
tags: 
 - vue
---

这是今天发现的一个新东西，提到vue中的css模块化，首先我想到的是style标签中的`scoped`属性,但它只是为了避免当前组件内的样式修改其他地方的样式。

也许外部的同名样式还是可以修改当前组件内的样式，比如添加`!important`。

scoped是在标签上添加一个唯一的data属性，同时在css选择器上加上相应的data标签选择器，其实是增加了选择器的权重的。

但[css module](https://github.com/css-modules/css-modules)，它是把标签和css选择器的class名称直接替换成一串唯一的字符串。外部样式就很难去影响组件内的样式，本身样式的权重也不会受到影响。

下面来看一下它的简单用法。

首先是webpack配置中，开启style-loader的module选项。
```javascript
...
loader: 'css-loader',
options: {
  // 开启 CSS Modules
  modules: true,
  // 自定义生成的类名
  localIdentName: '[local]_[hash:base64:8]'
}
...
```

然后在你的`<style>`标签中添加`module`属性
```html
<style lang="stylus" module>
.test1 {
  border: 1px solid #f00;
}
</style>
```
标签中不再使用`class=“test1”`，而是通过`:class="$style.test1"`来设置。
```html
<div :class="$style.test1">...</div>
```

来看一下生成的html标签和css样式
```html
<div class="_3X7g1lIceKkA-VJUIZdf_t_0">333</div>
```
```css
._3X7g1lIceKkA-VJUIZdf_t_0 {
    border: 1px solid #eee;
}
```
注意，为了调试方便，最好开启css souceMap。


如果想要在模块中设置全局样式，可以用`:global()`
```css
:global(.test2) {
  border: 1px solid #f23232;
}
```
使用时就直接设置class属性：
```html
<div class="test2">...</div>
```

以上，最基本的用法就介绍完了，更详细的可以参考 [vue-loader的这篇文档](https://vue-loader.vuejs.org/zh/guide/css-modules.html#%E7%94%A8%E6%B3%95) 和 [在vue中使用css modules替代scoped](https://www.cnblogs.com/xiaohuochai/p/8537959.html) 这两篇文章。


