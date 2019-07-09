title: 红米note手机border-bottom设置为1px时不显示问题
date: 2016-06-17
tags: 
 - 移动端
---


首先这个问题并不是我遇到的，隔壁的小伙伴来问，最后他说改成border-top就解决了。

场景，两列列表布局时，item之间用1px的边框分隔。

设置item的border-bottom:1px solid #ddd,border-right:1px solid #ddd;但是发现小米某机型中，列表的border-bottom除第一行外，其他都没显示。

解决办法是把border-bottom改为border-top

写到这里我已方。。。

昨天下班和今天早上都查了一些资料，但是无解。但是我测试了下该手机的devicePixelRatio值为3，猜测是它导致的。就像PSD中画了1px的边框，在缩放时底部的边框也很容易丢失。

