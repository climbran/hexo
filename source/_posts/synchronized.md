title: synchronized的对象锁和类锁
date: 2015-09-10 14:52:33
tags: [java]
---
很多人都知道synchronized可以用来加锁，但对于synchronized(Object o)里的参数不太了解。

java的每个对象隐含一个内部锁，这个内部锁对不同线程是互斥的。synchronized(object){}的本质就是对{}区域中的代码加上object对象的对象锁，只有获取了object的对象锁才能执行该代码块。

如果有：
{% codeblock lang:java %}
	Object o1 = new Object();
	Object o2 = new Object();
	
	synchronized(o1){
		//block1
	}
	
	synchronized(o1){
		//block2
	}
	
	synchronized(o2){
		//block3
	}
{% endcodeblock %}
当有一个线程1在执行block1时，线程2想要执行block2同样会被阻塞，应为o1的对象锁已经被线程1获取；而此时线程3想执行block3则不受影响。

对于类锁，其实就是Class实例的对象锁。因为一个类只有一个Class对象，所以同一个类的所有对象共享同一个类锁。

还有对于方法锁：
{% codeblock lang:java %}
class SynchronizedDemo{

	public void synchronized method1()	{//TODO} 
			
	public void method2()	{synchronized1(this){//TODO} }
	//method1效果等价于method2
	
	public static void synchronized method3(){//TODO} 
		
	public static void method4(){synchronized(SynchronizedDemo.class){//TODO} }
	//method3效果等价于method4
	
}
{% endcodeblock %}

总结：
大多数情况下同步代码块比同步方法要好，因为一个同步方法被执行完之前，该对象的所有同步方法都会被阻塞，而同步代码块可以通过加不同的对象锁来减少阻塞。