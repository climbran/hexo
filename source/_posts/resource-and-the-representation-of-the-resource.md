title: 关于资源和表述
date: 2015-07-11 21:44:26
tags: [restful]
---
最早通过在网上资料学习restful时，，都是说每个url代表一个资源(resource)，在项目设计时，会发现有时候为一个资源设计url时会发生冲突。<br><br>
比如一个教育系统中有老师和学生两种类型账号，一个班级的url为/class，而在老师账号中还有一个班级管理模块，同样使用的班级这一资源，但/class这一url已被使用。<br><br>
后来突然想到我们是不是能把浏览的班级和老师的班级管理模块当成两种不同资源？于是设计成了
<pre>
/class		// 班级资源
/classmgr	 //班级管理资源
</pre>
最近看《restful web APIs》时看到表述(representation of the resource)一词，突然顿悟到到其实上面两个url对应的是同一个资源的两个不同表述。每个url代表一个资源的说法也并没错，每个资源可以有多个表述，url和资源是多对一关系。在客户端看来，不同url获取的是不同表述，但最终定位的可能是同一资源。

