title: redis数据类型-hash
date: 2015-09-02 16:28:34
tags: [Redis]
---
1、redishash是一个string类型的field和value的映射表。
2、hash特别适合用于存储对象。相较于将对象的每个字段存成单个 string 类型。将一个对象存储在hash类 型中会占用更少的内存,并且可以更方便的存取整个对象。
3、新建一个 hash 对象时开始是用 zipmap(又称为 small hash)来存储的。
4、如果 field 或者 value 的大小超出一定限制后,redis会在内部自动将zipmap替换成正常的 hash 实现. 这个限制可以在配置文件中指定：
	
	hash-max-zipmap-entries 64 #配置字段最多 64 个
	hash-max-zipmap-value 512 #配置 value 最大为 512 字节
	
下面介绍 hash 相关命令：
	
	hset key field value	#设置 hash field 为指定值,如果 key 不存在,则先创建
	
	hget key field		#获取指定的 hash field
	
	hmget key filed1....fieldN		#获取全部指定的 hash filed
	
	hmset key filed1 value1 ... filedN valueN		#同时设置 hash 的多个 field
	
	hincrby key field integer		#将指定的 hash filed 加上给定值
	
	hexists key field		#测试指定 field 是否存在
	
	hdel key field		#删除指定的 hash field
	
	hlen key	#返回指定 hash 的 field 数量
	
	hkeys key	#返回 hash 的所有 field
	
	hvals key	#返回 hash 的所有 value
	
	hgetall key		#返回 hash 的所有 filed 和 value