title: 'Nodejs 连接 mysql时报错 Error: Cannot enqueue Query after fatal error'
date: 2015-12-09 11:41:03
tags: 
 - node
---
报错如下：
```
Error during request (500): Error: Cannot enqueue Query after fatal error.
at Protocol._validateEnqueue (MY_CODE\node_modules\express-mysql-session\node_modules\mysql-connection-manager\node_modules\mysql\lib\protocol\Protocol.js:193:16)
at Protocol._enqueue (MY_CODE\node_modules\express-mysql-session\node_modules\mysql-connection-manager\node_modules\mysql\lib\protocol\Protocol.js:129:13)
at Connection.query (MY_CODE\node_modules\express-mysql-session\node_modules\mysql-connection-manager\node_modules\mysql\lib\Connection.js:185:25)
at SessionStore.get (MY_CODE\node_modules\express-mysql-session\lib\index.js:127:18)
at Layer.session [as handle] (MY_CODE\node_modules\express-session\index.js:404:11)
```

解决办法，：[参考](https://github.com/chill117/express-mysql-session/issues/18)

我们只需在实例化SessionStore的时候，配置useConnectionPooling: true。比如：
```javascript
var sessionStore = new SessionStore({
    host: 'localhost',
    port: 3306,
    user: 'root',
    password: 'root',
    database: 'session',
    useConnectionPooling: true
});

```