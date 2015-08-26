title: java中回调函数的用法
date: 2015-04-28 09:27:42
tags: [java]
---
C语言中回调函数由指针实现，java 中没有指针，可以通过以下方法实现：

<pre>
public interface MyCallInterface{
	public void method();
}
</pre>
<pre>
public Client implements MyCallInterface{
	@Override
	public void method(){
		//TODO: 回调内容
	}
}
</pre>

调用时：
<pre>
public class Caller{
	private MyCallInterface callInterface;

	public Caller(){
	}

	public void setCallFunc(MyCallInterface callInterface){
		this.callInterface = callInterface;
	}

	public void call(){
		callInterface.method();
	}
}
</pre>
<pre>
public class Test{
	public static void main(String args){
		Caller caller = new Caller();
		caller.setCallFunc(new Client());
		client.call();
	}
}
</pre>
或者
<pre>
public class Caller{
	public void call(Client client){
		client.method();
	}
}
</pre>
<pre>
public class Test{
	public static void main(String args){
		Caller caller = new Caller();
		caller.call(new Client());
	}
}
</pre>
