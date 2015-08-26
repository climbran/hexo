title: 如何在mac os下根据端口号查杀进程
date: 2015-04-01 00:17:09
tags: [mac os,tomcat]
---
在使用mac os时，tomcat异常中断重启时提示端口被占用，可通过lsof -i :8080查看pid,然后kill <pid>来停止进程