title: 如何创建width与height比例固定的元素
date: 2015-12-09 16:29:04
tags:
 - css
---
面试题，刚在github上看到的，说说这里面的知识点吧~~

padding-bottom的值，其百分比是根据元素自身的width来算的。

padding，在标准盒模型中，width+padding+border=内盒

到这里，其实已经做到了宽高自适应且固定比例，题目算是解答完成了。不过，我们可以扩展下这个问题：

好了，新的问题来了，wrap的position为relative，而高度又无法指定，怎么让子元素的高度占满父元素呢？我们知道直接为box指定height：100%是没有效果的。

这时就又用到了神奇的position:absolute;结合left:0;top:0;right:0bottom:0;可以使box完全填充满父元素wrap，当然你不能额外的为box指定width与height，那样会功归一篑。
```html
    <style>
        .wrap{width: 100%;
            position: relative;
            padding-bottom: 20%;
        }
        .wrap .box{
            position: absolute;
            left: 0;
            top:0;
            right: 0;
            bottom: 0;
            background-color: #edd;
        }
    </style>
<div class="wrap">
    <div class="box">
        高宽5：1
    </div>
</div>
```
 还可以继续给box加padding,border，margin什么的，都ok的哟~~
 
 ```html
         .wrap .box{
            position: absolute;
            left: 0;
            top:0;
            right: 0;
            bottom: 0;
            padding: 10px;
            margin: 15px;
            background-color: #edd;
            border: 5px solid #fff;
        }
 ```
 ![](http://ww2.sinaimg.cn/large/66474ec2gw1eythm26ubgj207605kgly.jpg)