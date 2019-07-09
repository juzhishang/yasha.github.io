title: background简写属性
date: 2016-06-19
tags: 
 - css3
 - css
---


在我刚开始接触到h5的时候，曾经有同事告诉我，background-size比较特殊，不能放在background中简写。知道我看到css揭秘这本书，2333333

原来在background简写属性中指定background-size时，需要同时提供一个background-position的值（即便就是初始值也要加），然后它们之间用/分隔。  
比如：`background:url(bg.png) no-repeat top center / 2rem 2rem;`

这里引入/的原因是为了消除歧义，因为如果background-positon也是数值的话，解析器会无法区分到底是设置background-position还是设置background-size

