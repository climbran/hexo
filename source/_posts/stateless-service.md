title: 服务器无状态
date: 2015-09-07 12:21:03
tags: [web]
---
之前做听说可以不维持服务器的状态来减轻服务器压力，以为是指整个服务端都没有状态，一直没弄明白除非用cookie或者一次性的token，怎么维持客户端的状态。

今天看到<a href="http://kyfxbl.iteye.com/blog/1831869">无状态服务（stateless service）</a>才明白无状态是指对服务器集群中每台服务器(web server)来说是没有状态的。

一般可以认为session就是有状态的，session主要用于两种情况：
1、一个事务的多次请求间暂存数据
2、维持登陆状态
对于第一种情况，可以通过将数据回传到客户端，放在表单中隐藏或用sessionStorage、cookie来实现。
对于第二种情况，可以将session从web server中剥离，存放在共享的地方(如redis)。

附：
<a href="http://book.51cto.com/art/201403/432495.htm">理解服务的无状态性</a>
<a href="http://blog.csdn.net/romandion/article/details/1800025">状态和无状态－－2种服务器架构之间的比较 </a>