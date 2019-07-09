title: overflow-hidden与对齐
date: 2016-05-28
tags: 
 - css
---


遇到一个场景，inline-block元素设置了overflow:hidden后，它的兄弟元素（也是inline-block）出现了下沉，两者没有水平对齐。原来inline-block默认的对齐方式baseline(基线对齐)。正常情况下，line-block元素的基线是其内部最后一个行内元素基线。在内容非空或者overflow不为visible的情况下，它的基线是其下边距。inline-block设为overflow:hidden后，因为要遵循基线对齐，另一个元素就向下偏移了。

如何解决：  
方案1：设置为浮动元素，浮动元素的display默认为block，就不遵循inline-block的baseline对齐规则了。  
方案2：修改vertical-align:top，不是基线对齐自然就不会下沉。  
方案3：也加上overflow:hidden。

