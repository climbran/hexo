title: Java异常开销问题
date: 2015-07-29 14:08:10
tags: [java]
---
感觉在项目中使用异常进行逻辑控制能使代码逻辑清晰很多，但公司同事应为抛出异常开销的问题一直反对使用异常，最后代码中充斥着大量if else的逻辑判断。

但是在使用一些第三方框架时发现里面很多逻辑就在用异常进行控制，所以一直对异常开销这个问题存在疑问，抽空度娘，得出结论如下：

普通异常开销确实很大，但开销不在于异常的抛出，而是在异常对象创建时开销很大。
创建异常开销大的原因在于异常基类Throwable.java的public synchronized native Throwable fillInStackTrace()方法。

方法介绍:
Fills in the execution stack trace. This method records within this Throwable object information about the current state of the stack frames for the current thread.
性能开销在于:
1. 是一个synchronized方法(主因)
2. 需要填充线程运行堆栈信息

解决方法：
自定义异常继承Exception，并重写fillInStackTrace()
{% codeblock lang:java %}
	@Override
	public Throwable fillInStackTrace() {
		return this;
	}
{% endcodeblock %}
以该异常作为逻辑控制的异常的基类，那么异常性能将大幅度提高，和if else开销相近。

参考：<a href="http://www.blogjava.net/stone2083/archive/2010/07/09/325649.html">java异常性能问题</a>