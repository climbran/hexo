title: JAVA容器复习
date: 2015-09-09 20:51:01
tags: [java]
---
一、Collection接口
首先是Collection接口，继承Collection接口的有List、Set和Queue接口，Collection接口又继承自Iterable接口,所以List、Set、Queue的实现类都可以使用迭代器。

Interable中使用interator()方法返回一个Iterator接口类型的对象，Iterator<E>有三个方法：
{% codeblock lang:java %}
	boolean hasNext(); //是否有下一个元素
	E next();	//返回的下一个元素
	remove()  //移除迭代器返回的最后一个元素
{% endcodeblock %}
Collection增加以下方法：
{% codeblock lang:java %}	
	int size();
	boolean isEmpty();
	boolean contains(Object o);
	Object[] toArray();
	<T> T[] toArray(T[] a); 
	boolean add(E e);
	boolean remove(Object o);
	boolean containsAll(Collection<?> c);
	boolean addAll(Collection<? extends E> c);
	boolean removeAll(Collection<?> c);
	boolean retainAll(Collection<?> c);
	/*retainAll于从列表中移除未包含在指定collection中的所有元素
	如果List集合对象由于调用retainAll方法而发生更改，则返回 true*/
	void clear();
	boolean equals(Object o);
	int hashCode();
{% endcodeblock %}	
set对Collection基本没有新增方法

List比Collection增加了以下方法：
{% codeblock lang:java %}
get(int index);
E set(int index, E element);
void add(int index, E element);
E remove(int index);
int indexOf(Object o);
int lastIndexOf(Object o);
ListIterator<E> listIterator();	  //比Interator多了hasPrevious、previous、nextIndex、previousIndex、set、add;
List<E> subList(int fromIndex, int toIndex);	//返回一个子list，注意对子list的操作会影响到原来的list
{% endcodeblock %}

Queue比Collection增加以下方法：
{% codeblock lang:java %}
boolean offer(E e);		//添加一个元素并返回true       如果队列已满，则返回false
E remove();	// 移除并返问队列头部的元素,如果队列为空，则抛出一个NoSuchElementException
E poll();	//移除并返问队列头部的元素,如果队列为空，则返回null
E element();	//返回队列头部的元素,如果队列为空，则抛出一个NoSuchElementException异常
E peek();		//返回队列头部的元素,如果队列为空，则返回null
{% endcodeblock %}
二、Map接口
然后是Map接口，Map接口没有继承自任何接口。
{% codeblock lang:java %}
int size();
boolean isEmpty();
boolean containsKey(Object key);
boolean containsValue(Object value);
V get(Object key);
V put(K key, V value);
V remove(Object key);
void putAll(Map<? extends K, ? extends V> m);
void clear();
Set<K> keySet();	//返回key组成的一个set,多次调用返回同一个引用,该Set支持删除操作，不支持add、addAll，能影响到Map;
Collection<V> values();
Set<Map.Entry<K, V>> entrySet();
default boolean replace(K key, V oldValue, V newValue);
default V replace(K key, V value)	//如果map中已有key则覆盖，没有则返回null
{% endcodeblock %}

三、常用容器
(1)List
	ArrayList	非线程安全，能快速进行随机访问，顺序添加效率也高于LinkedList
	LinkedList	非线程安全，底层是双向链表，增删比ArrayList快，能更好的支持栈，同时实现了Queue接口，能支持队列操作,适合变动较为频繁的存储。
	Vector	相当于线程安全版ArrayList,如果不考虑并发，ArrayList效率较高，当数组长度不够时，Vector增加100%，ArrayList增加50%

其中LinkedList有些独有的方法：
{% codeblock lang:java %}
E getFirst();
E getLast();
E removeFirst();
E removeLast();
void addFirst(E e);
void addLast(E e);
{% endcodeblock %}
(2)Map
	HashMap 非线程安全，基于散列表实现，时间开销为O(1)，一般效率比TreeMap高
	TreeMap 非线程安全，基于红黑树实现，时间开销为O(logN),效率比HashMap低，但内部是有序的，TreeMap是唯一带有subMap()方法的Map，可以返回一个子树
	ConcurrentHashMap 线程安全，基于散列表实现ConcurrentMap接口，根据hash值来桶加锁。
	WeakHashMap 非线程安全，基于散列表是实现，其中的键是弱键，当map外没有引用时可能被GC回收。
四、优化
1、ArrayList默认生成一个长度为10的数组，当超过会生成一个更长的数组对象，然后将数据复制过去，如果估计使用ArrayList时将是很大一个数组可以预先设置数组大小来提高效率， eg: List list = new ArrayList(1000);
2、（备注：后来查看ArrayList和LinkedList的代码后发现size()方法只直接return size字段的值，所以两种方法应该效率没差别）
	通过循环遍历List时，如果没有add、remove操作改变数组大小，不用每次都读取数组大小,eg:
	
	for (int i = 0; i < vector.size (); i++) 
	改为 
	for (int i = 0,n=list.size (); i < n; i++) 
3、ArrayList用get遍历，用iterator遍历也可以
	   LinkedList应避免用get遍历(随机存取效率低)，尽量用iterator