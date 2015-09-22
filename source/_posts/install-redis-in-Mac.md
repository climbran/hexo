title: Mac下安装Redis
date: 2015-09-1 19:34:08
tags: [Redis]
---
<a href="http://redis.io/download">http://redis.io/download</a>
下载最新的Redis版本，我下载的是3.0.3
下载后将文件解压，通过进入文件夹
	
	cd redis-3.0.3
执行

	make install
Redis就安装好了，下面是启动指令：

	#启动 加上‘&’使redis能在后台运行
	src/redis-service &
	#关闭
	src/redis-cli shutdown
测试

	localhost:redis-3.0.3 ywq$ src/redis-cli 
	127.0.0.1:6379> set foo test
	OK
	127.0.0.1:6379> get foo
	"test"