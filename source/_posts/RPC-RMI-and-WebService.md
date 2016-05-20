title: RPC定义(RMI、WebService)
date: 2015-12-05 14:51:51
tags: [RPC,RMI,WebService]
---
RPC(Remote Process Call)--即远程调用，使用C/S的方式，发送请求到服务器，等待返回，已有很多成熟的实现，RMI和Web Service都是其是实现。

Web Service
	Web Service是基于web容器(如Tomcat、IIS等)提供服务的，是RPC在http协议上的一种是实现，能灵活的跨平台应用。
		
RMI(Remote Method Invocation)
	RMI是在tcp协议上传递可序列化的java对象，绑定语言，client与service紧耦合。

Web Service 与 RMI 对比
	Web Service和RMI方式的优劣主要基于其依赖的网络协议。
	RMI通过TCP协议实现，TCP协议在协议栈中处于http协议的底层，能通过较少的字节数传输同样的内容，降低了网络开销，提高性能，增强吞吐量和并发能力，但实现代价更高，而且RMI只能基于java语言，较难跨平台使用。
	Web Service基于http协议，可以使用json和xml等格式传输数据，有很多成熟的解析工具可以使用，屏蔽了很多底层实现细节，但传输效率要比RMI方式低。