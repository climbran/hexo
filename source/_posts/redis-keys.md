title: redis数据类型-keys
date: 2015-09-02 14:24:59
tags: [Redis]
---
redis本质上一个 key-value数据库，关于key注意以下几点：
1、其中key也是字符串类型,但是key中不能包括边界字符。
2、由于key不是二进制安全(binary safe)的字符串,所以像"my key"和"mykey\n"这样包含空格和换行的 key 是不允许的。
3、redis内部并不限制使用binary字符,这是redis协议限制的。"\r\n"在协议格式中会作为特殊字符。
关于key的一个格式约定介绍下, object-type:id:field。比如 user:1000:password,blog:xxidxx:title。key的长度最好不要太长，占内存,而且查找时候比短key慢。不过也不推荐过短的key,比如 u:1000:pwd,这的，可读性太差。
下面是key相关命令：
	exits key	
测试指定 key 是否存在,返回 1 表示存在,0 不存在	
	del key1 key2...keyN	
删除给定 key,返回删除 key 的数目,0 表示给定 key 都不存在	type key
返回给定 key 的 value 类型。返回 none表示不存在,string字符类型,list链表类型,set无序集合类型...		keys pattern	
返回匹配指定模式的所有 key,下面给个例子	randomkey
返回从当前数据库中随机选择的一个 key,如果当前数据库是空的,返回空串
	
	rename oldkey newkey
原子的重命名一个 key,如果 newkey已存在,将会被覆盖,返回 1 表示成功,0 失败。可能是oldkey不存在或者oldkey和newkey相同。

	renamenx oldkey newkey
同上,但是如果 newkey 存在返回失败(0)
	
	dbsize
返回当前数据库的key的数量

	expire key seconds
为key指定过期时间，单位是秒，返回1成功,0表示key已经设置过过期时间,或者key不存在

	ttl key
返回设置过过期时间的 key 的剩余过期秒数 -1 表示 key 不存在或者没有设置过过期时间

	select db-index
通过索引选择数据库,默认连接的数据库是0,默认数据库数是16个。返回1表示切换连接成功,0表示失败

	move key db-index 
将key从当前数据库移动到指定数据库。返回1成功。0如果key不存在,或者已经在指定数据库中

	flushdb
删除当前数据库中所有 key,此方法不会失败。慎用

	flushall
删除所有数据库中的所有 key,此方法不会失败。更加慎用


总结：
1、关于select db-index,redis默认分割成16个数据库，默认连接到索引为0的数据库，每个数据库都是动态大小的，一开始非常小。分库作用只是提供一个简单的命名空间（namespace），以便将不同用途的数据分开而已，在同一个服务器进行分库不会带来任何效率上的提升。
2、关于move key db-index,只要目标数据库中已存在指定的key，不管value为多少都会返回失败。
3、关于keys pattern,例子:
{% codeblock %}
redis> set test dsf OKredis> set tast dsaf OKredis> set tist adff OKredis> keys t*1. "tist"2. "tast"3. "test"redis> keys t[ia]st
1. "tist"2. "tast"redis> keys t?st1. "tist"2. "tast"3. "test"
{% endcodeblock %}