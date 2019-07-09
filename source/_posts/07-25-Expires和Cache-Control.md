title: Expires和Cache-Control
date: 2016-07-25
tags: 
 - http
 - 缓存
---


Expires要求客户端和服务端的时钟严格同步。HTTP1.1引入Cache-Control来克服Expires头的限制。如果max-age和Expires同时出现，则max-age有更高的优先级。

```
Cache-Control: no-cache, private, max-age=0  
ETag: abcde  
Expires: Thu, 15 Apr 2014 20:00:00 GMT  
Pragma: private  
Last-Modified: $now // RFC1123 format  

```

