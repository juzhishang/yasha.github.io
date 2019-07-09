title: 四则运算 calc()
date: 2015-12-09 16:25:42
tags:
 - css
 - css3
---
它的出现还真的蛮令人惊喜的，很适用于百分比宽度。之前我们有box-sizng，而今又多了一个它，并且，calc的实用性更高。我们可以在border、margin、pading、font-size和width等属性上做一些加减乘除，简单拿width举例：
```javascript
.wrap {
background-color: #dee;
height: 200px;
padding: 40px;
display: inline-block;
width: calc(100% - 2 * 40px);
}
```
书写注意点：

width: calc(100%-2*40px);//错误的写法
（这是我在测试的时候踩的坑→ →）

运算符前后加一个空格，不然浏览器可能会无法识别哟~~

 

其它貌似也没有什么了，最后看一下兼容性吧，:)
![](http://ww3.sinaimg.cn/large/66474ec2gw1eythif09c3j20ln0603zl.jpg)