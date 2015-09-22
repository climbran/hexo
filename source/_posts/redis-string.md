title: redis数据类型-string
date: 2015-09-02 15:05:35
tags: [Redis]
---
1、string 是 redis 最基本的类型
2、string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如 jpg 图片或者序列化的对象
3、内部实现来看其实 string 可以看作 byte 数组,最大 上限是 1G 字节
4、sting可以被部分命令当做int处理，如incr

string类型的定义：
{% codeblock lang:c %}
struct sdshdr {long len; 		//数组中剩余可用字节数long free; 		//buf数组的长度char buf[]		//用于存贮实际的字符串内容的数组;};
{% endcodeblock %}

string相关命令如下
	
	set key value 
设置 key 对应的值为 string 类型的 value,返回 1 表示成功,0 失败

	setnx key value
同上,但如果 key 已经存在,返回 0 。nx 是 not exist 的意思

	get key
获取 key 对应的 string 值,如果 key 不存在返回 nil

	getset key value
原子的设置 key 的值,并返回 key 的旧值。如果 key 不存在返回 nil

	mget key1 key2 ... keyN
一次获取多个 key 的值,如果对应 key 不存在,则对应返回 nil

	mset key1 value1 ...keyN value N
一次设置多个 key 的值,成功返回1,表示所有的值都设置了;失败返回0,表示没有任何值被设置

	msetnx key1 value1 ...keyN valueN
同上,但是不会覆盖已经存在的 key

	incr key
对 key 的值做加加操作,并返回新的值。
注意:incr 一个不是 int 的 value 会返回错误;incr一个不存在的 key,则设置 value 为 1

	decr
同上,但是做的是减减操作,decr 一个不存在 key,则设置 key 为-1
	
	incrby key integer
同 incr,加指定值 ,key 不存在时候会设置 key,并认为原来的 value 是 0

	decrby key integer￼同 decr,减指定值。
decrby完全是为了可读性,我们完全可以通过 incrby 一个负值来实现同样效果,反之一样。
	
	append key value
给指定 key 的字符串值追加 value,返回新字符串值的长度。

	substr key start end
返回截取过的 key 的字符串值。
注意该命令并不修改 key 的值,下标是从 0 开始的
	
	
	
总结：
1、关于mget，如果某个key不存在,只有不存在的key返回nil，其他key任正常返回，例子：
{% codeblock %}
redis> flushdb OKredis> dbsize (integer) 0
redis> set k1 a OKredis> set k2 bOKredis> mget k1 k2 k3 
1. "a"2. "b" 3. (nil)
{% endcodeblock %}
2、关于mset和msetnx
mset会覆盖已存在的key
msetnx只要有一个key已存在就返回0，表示没有任何值被设置
3、关于incrby、decrby
当value不是一个int类型时会返回失败