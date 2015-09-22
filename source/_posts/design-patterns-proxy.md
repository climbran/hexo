title: 代理模式
date: 2015-09-21 21:00:49
tags: [设计模式]
---

一、代理模式

代理模式是通过一个代理类来控制对象的操作，下面是代理模式的一个简单实现：
{% codeblock Subject.java %}
public interface Subject{
	public void doSomething();
}
{% endcodeblock %}
{% codeblock RealSubject.java %}
public class RealSubject implements Subject{
	private String topic = "";
	
	public RealSubject(String topic){
		this.topic = topic;
	}
	public void doSomething(){
		System.out.println("do " + topic);
	}
}
{% endcodeblock %}
{% codeblock Proxy.java %}
public class Proxy implements Subject{
	private Subject subject = null;
	
	public Proxy(Subject subject){
		this.subject = subject;
	}
	
	public void doSomething(){
		this.before();
		this.subject.doSomething();
		this.after();
	}
	
	private void before(){
		//TODO: do something
	}
	
	private void after(){
		//TODO: do something
	}

}
{% endcodeblock %}
{% codeblock Client.java %}
public class Client{
	public void main(String[] args){
		Subject subject = new RealSubject("test");
		Subject proxy = new Proxy(subject);
		proxy.doSomething();
	}
}
{% endcodeblock %}

代理模式中真实角色只负责实际的业务逻辑，不用关心其他非本职的事务（如权限、日志等），其他事务都有代理类去实现(这就是一个简单的aop，更完善的aop实现可以使用动态代理)而且由于实现了接口，可以随时替换真实角色

二、普通代理

普通代理是代理模式的一个扩展，普通代理只要关注代理类，调用者不用关注被代理对象，因此被代理类只需要实现接口就可以随便修改，具有很高的扩展性，其实现对RealSubject和Proxy类做出如下修改：
{% codeblock RealSubject.java %}
public class RealSubject implements Subject{
	private String topic = "";
	
	public RealSubject(Subject _subject,String topic){
		if(_subject == null){
			throw new Exception("不能创建真实角色");//传入参数用来约束不能直接new一个实力，也可以通过团队内约定来实现
		}
	}
	public void doSomething(){
		System.out.println("do " + topic);
	}
}
{% endcodeblock %}
{% codeblock Proxy.java %}
public class Proxy implements Subject{
	private Subject subject = null;
	
	public Proxy(String topic){
		try{
			subject = new RealSubject(this,topic);
		}catch(Exception e){
			//TODO: 异常处理 
		}
	}
	
	public void doSomething(){
		this.before();
		this.subject.doSomething();
		this.after();
	}
	
	private void before(){
		//TODO: do something
	}
	
	private void after(){
		//TODO: do something
	}

}
{% endcodeblock %}
{% codeblock Client.java %}
public class Client{
	public void main(String[] args){
		Subject proxy = new Proxy("test");
		proxy.doSomething();
	}
}
{% endcodeblock %}

三、强制代理

强制代理也是代理模式的一个扩展，是通过被代理对象获取对应的代理，并且必须通过代理来访问方法。另外代理类可以实现多个接口，完成不同模块整合，在下面强制代理的示例中一起展示多接口的实现：
{% codeblock Subject.java %}
public interface Subject{
	public void doSomething();
	public Subject getProxy();
}
{% endcodeblock %}
{% codeblock  Accountant.java %}
public interface Accountant{
	public void count(); //计算花费
}
{% endcodeblock %}
{% codeblock RealSubject.java %}
public class RealSubject implements Subject{
	private String topic = "";
	private Subject proxy = null;
	
	public RealSubject(String topic){
		this.topic = topic;
	}
	
	public Subject getProxy(){
		this.proxy = new Proxy(this);
		return this.proxy;
	}
	
	public void doSomething(){
		if(isProxy())
			System.out.println("do " + topic);
		else
			System.out.println("error: please use proxy");
	}
	
	//判断是否是通过代理调用，其实调用getProxy方法后不通过代理也可以直接调用其他方法，和普通代理不能直接new一个被代理对象的约束一样，可通过团队内约定来约束
	private boolean isProxy(){
		return proxy==null?false:true;
	}
}
{% endcodeblock %}
{% codeblock Proxy.java %}
public class Proxy implements Subject,Accountant{
	private Subject subject = null;
	
	public Proxy(Subject subject){
		this.subject = subject;
	}
	
	public doSomething(){
		subject.doSomething();
		this.count();
	}
	publiv Subject getProxy(){
		return this;
	}
	
	public void count(){
		//TODO: do something
	}
}
{% endcodeblock %}
{% codeblock Client.java %}
public class Client{
	public void main(String[] args){
		Subject subject = new RealSubject("test");
		Subject proxy = subject.getProxy();
		proxy.doSomething();
	}
}
{% endcodeblock %}


四、动态代理

 和AOP编程密切相关，见<a href="/2015/09/22/dynamic-proxy-and-aop/">《动态代理及简单的AOP框架的实现》</a>