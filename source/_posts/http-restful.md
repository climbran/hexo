title: restful中http请求的语义
date: 2015-07-12 09:24:50
tags: [restful]
---
RESTful中使用HTTP协议作为应用层协议。本文主要介绍http协议的 GET、POST、PUT、PATCH、DELETE请求的语义。

HTTP GET: 主要用于获取URI所代表的资源的表述，但不应该改变资源，是安全且幂等的。

HTTP POST: 用于创建资源，POST对应的URI是一个表述的类型，比如创建一篇文章应该是 
	
	POST http://www.climbran.com/article
多次POST请求会创建多篇文章。

HTTP PUT: 用于创建或更新资源，创建资源时与POST相比，区别在于PUT是幂等的，例如:
	
	PUT http://www.climbran.com/article/1
表示创建或更新id为1的文章。

HTTP PATCH: 用于更新资源，与PUT相比，区别在于PUT应带有URI对应表述的全部信息，PATCH只带有部分信息，可用于表述信息量非常大，但只希望修改其中一小部分的时候。PATCH同样是幂等的。

HTTP DELETE: 用于删除资源，应该是幂等的。

例子：

        GET /zoos：列出所有动物园
        POST /zoos：新建一个动物园
        GET /zoos/ID：获取某个指定动物园的信息
        PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
        PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
        DELETE /zoos/ID：删除某个动物园
        GET /zoos/ID/animals：列出某个指定动物园的所有动物
        DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
        
各种请求的返回结果一般是：

        GET /collection：返回资源对象的列表（数组）
        GET /collection/resource：返回单个资源对象
        POST /collection：返回新生成的资源对象
        PUT /collection/resource：返回完整的资源对象
        PATCH /collection/resource：返回完整的资源对象
        DELETE /collection/resource：返回一个空文档



