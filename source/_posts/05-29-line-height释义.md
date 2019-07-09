title: line-height释义
date: 2016-05-29
tags: 
 - css
---


引自`https://www.w3.org/TR/CSS21/visudet.html#propdef-vertical-align`

‘line-height’

值: normal | `<number>` | `<length>` | `<percentage>` | inherit

初始值: normal

适用: 所有元素

继承性: 是

百分比值: 与元素本身的font-size有关

媒体: 可见

计算后的值:对于 `<length>` 和 `<percentage>` 而言为绝对值;  
其它则由行内元素所组成的块容器元素指定， ‘line-height’ 指定了元素内line box的最小高度。最小高度由基线以上的最小高度和基线以下的最小深度组成，就像每一个line box以一个零宽的有font和line-height属性的行内盒子开始，我们把这种虚构的盒子叫做strut（支杆）。

字体基线以上的高度及基线以下的深度被认为是字体内的标准。（更多详情，查看CSS level 3）。

在非嵌入行内元素中，’line-height’指定的高度被用于计算line box的高度。

该属性值释义如下：

*   normal

告诉用户代理根据元素的字体设置合理的值。该值与`<number>`同义。我们认为’normal’的使用值在1.0到1.2之间。计算值为’normal’。

*   `<length>`

指定长度被用来计算line box的高度，负值是非法的。

*   `<number>`

该属性它的用法是数值*元素字体大小。负值是非法的。计算值与指定值相同。

*   `<percentage>`

属性的计算值是百分比值*元素的计算字体大小。负值是非法的。

下面例子中的三个规则行高结果一致：

```
div { line-height: 1.2; font-size: 10pt }     /* number */  
div { line-height: 1.2em; font-size: 10pt }   /* length */  
div { line-height: 120%; font-size: 10pt }    /* percentage */  

```

当元素所包含的文本渲染时字体多于一种，用户代理会根据最大的字体大小来决定’line-height’的’normal’值。

注意：当块容器元素中所有的行内元素line-height值相同，并且它们的字体相同（并且其中没有嵌入元素，也没有行内块元素），由此可以确定连续文本行的基线之间的距离就是’line-height’。当不同字体的文本行需要对齐的时候，这点很重要，比如在表格中。

</percentage></number></length></number></percentage></length></percentage></length></number>