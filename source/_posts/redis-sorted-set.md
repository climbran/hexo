title: redis数据类型-sorted set
date: 2015-09-02 16:13:04
tags: [Redis]
---
1、set 一样,sorted set 也是 string 类型元素的集合,不同的是每个元素都会关联一个 double 类型的 score。
2、sorted set 的实现是 skip list 和 hash table 的混合体。
3、sorted set 最经常的使用方式应该是作为索引来使用。我们可以把要排序的字段作为 score 存储,对象的 id 当元素存储。

下面是 sorted set 相关命令:
{% codeblock %}
	zadd key score member	#添加元素到集合,元素在集合中存在则更新对应 score
	
	zrem key member		#删除指定元素,1 表示成功,如果元素不存在返回 0
	
	zincrby key incr member		#增加对应 member 的 score 值,然后移动元素并保持 skip list 保持有序。返回更新后的 score 值
	
	zrank key member	#返回指定元素在集合中的排名(下标),集合中元素是按 score 从小到大排序的
	
	zrevrank key member		#同上,但是集合中元素是按 score 从大到小排序
	
	zrange key start end	#类似 lrange 操作从集合中去指定区间的元素。返回的是有序结果
	
	zrevrange key start end		#同上,返回结果是按 score 逆序的
	
	zrangebyscore key min max	#返回集合中 score 在给定区间的元素
	
	zcount key min max		#返回集合中 score 在给定区间的数量
	
	zcard key		#返回集合中元素个数
	
	zscore key element		#返回给定元素对应的 score
	
	zremrangebyrank key min max		#删除集合中排名在给定区间的元素
	
	zremrangebyscore key min max	#删除集合中 score 在给定区间的元素
{% endcodeblock %}