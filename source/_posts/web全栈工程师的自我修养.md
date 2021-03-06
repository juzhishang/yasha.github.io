title: web全栈工程师的自我修养
date: 2015-11-24 01:40:23
tags:
 - 读书笔记
---
从标题就不难看出，这不是一本技术型的书籍。它涉及的内容很杂，大部分篇幅是关于个人的成长，当然也有一些技术类的文章，比如http与缓存。从网上的评论看，好坏参半，对我而言，算是一本值得买的书吧。

#### 思考

野生程序员：非计算机专业背景的开发者大概会比较容易陷入到这样一种尴尬的境地，各方面都涉及一点，但又都一知半解。其实不管有没有专业背景，最后都需要有自己的技术发展方向，我们应该追求一专多长。如果都只是略懂，不管是在大公司还是小公司，都很有可能陷入难堪。

大公司还是小公司：最早的时候，我在一家10余人的小公司打杂，不久来到杭州，公司从200多人换到300多人，而后是现在的这家，将近1000人。我的感受是，小公司通常会缺乏规范，疲于应付业务，你很少有机会见识到新技术的应用，一方面它的用户量不多，另一方面，是由于资金和人才的比较缺乏，以及学习成本等的限制。而大公司，它的创造力会更强一点（脑洞大开），经常会有一个新的项目出来解放生产力。大公司通常在代码管理、规范、性能优化方面会做的比较好，这也是我们个人成长上非常重要的一点。

声望与技术积累：这个是老生常谈的话题，但是确实，对我们的日后发展有很大的影响，不管未来是走管理路线还是技术路线，不管是升职还是跳槽，都离不了。感觉声望是“专业可信赖”的升级版。技术沉淀和开源项目，这一方面我很欠缺。

#### 笔记：

##### web性能优化（http请求）
IE6/7和firefox2的设计规则是，同时只能对一个域名发起两个并发连接。新版本的各种浏览器普遍把这一连接上限设定为4至8个。

把静态资源放在非主域名下，这种做法除了可以增加浏览器并发，还可以减少http请求中携带的不必要的cookie数据。

对于比较大的文本资源，必须开启gzip压缩，因为gzip对于含有重复“单词”的文本文件，压缩率非常高，能有效提高传输过程。

##### 框架与库的区别
框架vs库：一个库是一系列对象、方法等代码，它起到了重用代码的作用。而一个框架是一个软件系统中可重用的一部分，它可能包含子程序、库、胶水语言、图片等资源。框架是一个更加灵活和宽松的名词。

##### 版本控制最佳实践
版本控制最佳实践：
1、鼓励频繁的提交。2、确定分支流程。3、不要把逻辑的修改和代码格式化的操作混在一起。4、不相干的代码分开提交。5、保持工作代码库的“干净”。

##### 版本号规范
版本号：根据semver规范，版本号用小数点分割为三个数字。如v3.2.1中3是主要版本号，2是次要版本号，1是补丁。主要版本号：有API变更导致不兼容旧版本时使用。次要版本号：新增功能，但是向前兼容的情况下使用。补丁：修复向前兼容的bug时使用。

##### prefork与worker的区别

虽然服务器的多个进程看上去是在同时运行，但是对于单核cpu的架构来说，实际上是计算机系统同一时间内，以进程的形式，将多个程序加载到存储器中，并借由时间共享，以在一个处理器上表现出同时运行的感觉。由于在操作系统中，生成进程、销毁进程、进程间切换都很消耗cpu和内存，因此，当负载高时，性能会明显降低。

在早期系统中（如linux2.4以前），进程是基本运作单位，在支持线程的系统中（如linux2.6），线程才是基本的运作单位，而进程只是线程的容器。Apache有一个模块叫做多处理模块（MPM），在linux上，默认的MPM是prefork，为了优化可以改成worker。

prefork和worker模式的最大区别就是，prefork的一个进程维持一个连接，而worker的一个线程维持一个连接，所以prefork更稳定但更消耗内存也更大，worker没有那么稳定，因为很多连接的线程共享一个进程，当一个线程崩溃的时候，整个进程和所有线程一起死掉。但是worker的内存使用要比prefork低很多，所以很适合用在高http请求的服务器上。

Nginx更轻量级，占用更少的资源和内存，另一方面，nginx处理请求的异步非阻塞的。而Apache是阻塞型的。在高并发下Nginx能保持低资源、低消耗和高性能。经常的搭配是Nginx处理前端并发，Apache处理后台请求。Nodejs也是采用基于事件的异步非阻塞方式处理请求，所以在处理高并发请求上有天然的优势。

##### DdoS
DDoS:分布式拒绝服务攻击。攻击者通过海量的请求，让目标服务器瘫痪，无法响应正常的用户请求，以此达到攻击的目的。

##### 现有数据请求流程的问题与优化方案

现有数据请求流程的问题：
第一，http协议的底层是tcp/ip,而tcp/ip规定3次握手才建立一次连接。每一个新增的请求都要重新建立tcp/ip连接，从而消耗服务器的资源，并且浪费连接时间。
第二，在现有阻塞模型中，服务器计算生成页面需要时间。等服务器完全生成好整个页面，才开始网络传输，网络传输也需要时间。整个页面都完全传输到浏览器中后，在浏览器中的最后渲染还是需要时间。三者是阻塞式的，每一个环节都在等上一个环节100%完成才开始。页面作为一个整体，需要完整地经历3个阶段才能出现在浏览器中，效率很低。

BigPipe非阻塞式模型，http1.1的分块传输编码。允许服务器为动态生成的内容维持http持久链接。如果一个http消息（请求或应答）的transferEncoding消息头值为chunked，那么消息体由数量不确定的块组成，也就是说想发送多少块就发送多少块，并以最后一个大小为0的块为结束。
