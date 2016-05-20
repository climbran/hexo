title: java单例模式中的陷阱(反射、序列化、clone)
date: 2015-12-04 11:32:23
tags: [java,设计模式]
---
Java的单例模式希望始终获得的是一个实力，有以下几种情况可能破坏单例模式

一、反射
	即使将构造器设为私有仍可以用反射机制访问构造器，因此应在构造器中判断是否instance == null,不为null时直接返回。
	eg:
{% codeblock %}
public class Singleton{
	private static final Singleton instance = new Singleton();
		
	private	Singleton(){
		if(instance != null)
			return instance;
	}
	
	public static Singleton getInstance(){
		return instance;		
	}	
	//other method

}
{% endcodeblock %}

二、序列化
	序列化过程中静态变量都是transient的，不会被保存，在反序列化的过程中会重新生成实例，从而破坏单例，可以通过readResolve方法解决。
	eg:
{% codeblock %}
public class Singleton implements Serializable{
	private static final Singleton instance = new Singleton();
		
	private	Singleton(){
		if(instance != null)
			return instance;
	}
	
	public static Singleton getInstance(){
		return instance;		
	}	
	
	private Object readResolve() {
        return instance;
    }
	//other method
} 
{% endcodeblock %}

三、clone
	所有类都继承自Object类，所以都有clone方法，为了防止clone方法破坏单例，可以override它的clone方法，让它抛出异常。
	
	