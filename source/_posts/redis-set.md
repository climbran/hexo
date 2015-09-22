title: redis数据类型-set
date: 2015-09-02 15:58:06
tags: [Redis]
---

1、redis 的 set 是 string 类型的无序集合
2、set 元素最大可以包含(2的32次方-1)个元素
3、set 的是通过 hash table 实现的,所以添加,删除,查找的复杂度都是 O(1)。

相关命令：
{% codeblock %}
	sadd key member		#添加一个 string 元素到,key 对应的 set 集合中,成功返回 1,如果元素已经在集合中,返回 0,key 对应的 set 不存在返回错误

	srem key member		#从 key 对应 set 中移除给定元素,成功返回 1,如果 member 在集合中不存在或者 key 不存在返回 0,如果 key 对应的不是 set 类型的值返回错误
	
	spop key	#删除并返回 key 对应 set 中随机的一个元素,如果 set 是空或者 key 不存在返回 nil
	
	srandmember		#同 spop,随机取 set 中的一个元素,但是不删除元素
	
	smove srckey dstkey member		#从 srckey 对应 set 中移除 member 并添加到 dstkey 对应 set 中,整个操作是原子的。成功返回 1,如果 member 在 srckey 中不存在返回 0,如果 key 不是 set 类型返回错误
	
	scard key		#返回 set 的元素个数,如果 set 是空或者 key 不存在返回 0
	
	sismember key member	#判断 member 是否在 set 中,存在返回 1,0 表示不存在或者 key 不存在
	
	sinter key1 key2...keyN		#返回所有给定 key 的交集
	
	sinterstore dstkey key1...keyN		#同 sinter,但是会同时将交集存到 dstkey 下
	
	sunion key1 key2...keyN		#返回所有给定 key 的并集
	
	sunionstore dstkey key1...keyN		#同 sunion,并同时保存并集到 dstkey 下
	
	sdiff key1 key2...keyN		#返回所有给定 key 的差集
	
	sdiffstore dstkey key1...keyN	#同 sdiff,并同时保存差集到 dstkey 下
	
	smembers key	#返回 key 对应 set 的所有元素,结果是无序的￼{% endcodeblock %}