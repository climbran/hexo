title: 单例模式及懒汉式的同步优化
date: 2015-09-10 17:37:16
tags: [设计模式]
---
单例模式分为饿汉式和懒汉式，饿汉式写起来简单，且不用考虑初始化时的同步问题；懒汉式的优点在于一开始不用初始化，可以节约内存，但需要为初始化操作进行同步。

一直不理解为了这一点内存给一个方法加锁值得吗？影响并发，可能是做的项目还不够多，没有遇到需要大量内存的情况？

不过昨天发现了一个比较好的方法可以在节省内存的同时减少阻塞，同样的逻辑也可以用在其他需要同步的情况下，特此记录下

饿汉式标准版：
{% codeblock lang:java %}
public class Singleton{
	private static final Singleton instance = new Singleton();
	
	private	Singleton(){
	}
	
	public static Singleton getInstance(){
		return instance;
	}
	
	//other method

}
{% endcodeblock %}

懒汉标准版：
{% codeblock lang:java %}
public class Singleton{
	private static Singleton instance = null;
	
	private	Singleton(){
	}
	
	public static Singleton getInstance(){
		synchronized(Singleton.class){
			return instance==null?new Singleton:instance;
		}
	}
	
	//other method

}
{% endcodeblock %}

懒汉优化版：
{% codeblock lang:java %}
public class Singleton{
	private static Singleton instance = null;
	
	private	Singleton(){
	}
	
	public static Singleton getInstance(){
		if(instance==null){
			synchronized(Singleton.class){
				if(instance==null)
					return new Singleton();
			}
		}
		return instance
	}
	
	//other method

}
{% endcodeblock %}

优化后，只有未初始化时才会加锁，初始化后instance不为null，直接返回单例对象，不需要再次进入同步块。
