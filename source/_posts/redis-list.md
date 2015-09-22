title: redis数据类型-list
date: 2015-09-02 15:31:25
tags: [Redis]
---
1、redis 的 list 类型其实就是一个每个子元素都是 string 类型的双向链表，所以[lr]push 和[lr]pop 命 令的算法时间复杂度都是 O(1)。
2、list 会记录链表的长度，所以 llen 操作也是 O(1)。
3、链表的最大长度是(2的32次方-1)
4、 list 的 pop 操作还有阻塞版本的。当我们[lr]pop 一个 list对象是,如果list是空,或者不存在,会立即返回nil。但是阻塞版本的b[lr]pop可以则可以阻塞, 当然可以加超时时间,超时后也会返回 nil。阻塞版本的pop主要是为了避免轮询。

list相关命令:

	lpush key string
key 对应 list 的头部添加字符串元素,返回 1 表示成功,0 表示 key 存在且不是 list 类型

	
	rpush key string 
同上，在尾部添加

	llen key
返回 key 对应 list 的长度,key 不存在返回 0,如果 key 对应类型不是 list 返回错误

	lrange key start end 
返回指定区间内的元素,下标从 0 开始,负值表示从后面计算,-1 表示倒数第一个元素 ,key 不存在返回空列表

	ltrim key start end￼截取 list,保留指定区间内元素,成功返回 1,key 不存在返回错误
	lset key index value￼设置 list 中指定下标的元素值,成功返回 1,key 或者下标不存在返回错误
	lrem key count value
从 key 对应 list 中删除count个和value相同的元素。count 为 0 时候删除全部。返回被移除元素的数量	lpop key
从 list 的头部删除元素,并返回删除元素。如果 key 对应 list 不存在或者是空返回 nil,如果 key 对应值不是 list 返回错误
	
	rpop key
同上,但是从尾部删除

	blpop key1...keyN timeout
从左到右扫描返回对第一个非空 list 进行 lpop 操作并返回,比如 blpop list1 list2 list3 0 ,如果 list 不存在,list2,list3 都是非空则对 list2 做 lpop 并返回从 list2 中删除的元素。如果所有的 list 都是空或不存在,则会阻塞 timeout 秒,timeout 为 0 表示一直阻塞。当阻塞时,如果有client 对 key1...keyN 中的任意 key 进行 push 操作,则第一在这个 key 上被阻塞的 client 会立即返回。如果超时发生,则返回 nil。有点像 unix 的 select 或者 poll。

	brpop key1...keyN timeout
同 blpop,一个是从头部删除一个是从尾部删除。

	rpoplpush srckey destkey从 srckey 对应 list 的尾部移除元素并添加到 destkey 对应 list 的头部,最后返回被移除的元素值,整个操作是原子的.如果 srckey 是空或者不存在返回 nil。	
	
总结：
1、关于lpush和rpush
当key不存在时则创建key并添加
2、关于lrem
	count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
	count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
	count = 0 : 移除表中所有与 value 相等的值。
	因为不存在的 key 被视作空表(empty list)，所以当 key 不存在时， LREM 命令总是返回 0 
