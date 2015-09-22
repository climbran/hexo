title: 匿名内部类学习
date: 2015-08-30 22:04:41
tags: [java]
---
晚上看Spring-Data-Redis,别人的demo里通过对匿名内部类的回调实现set、get方法，仔细想以下，发现自己对匿名内部类掌握不是很好，就又翻了下Thinking in Java。

匿名内部类的格式：
{% codeblock lang:java %}
new Contents(){
	...
}
{% endcodeblock %}

我对匿名内部类主要有以下几个疑问：<br>
一、匿名内部类实现的时什么类型的对象，可以实现接口吗？<br>
Contents()可以是调用的一个类的构造器，此时括号中可以根据构造器传入参数，将创建一个继承自Contents的匿名类对象；<br>
Contents也可以是一个接口，将创建一个实现Contents接口的匿名类对象，此时匿名类中只有一个隐式的无参构造器，所以括号中不能传入参数；<br>
二、如何引用匿名内部类？<br>
回答了问题一其实就能解决这个问题了，new表达式返回的引用会被自动向上转型为对Contents的引用<br>
三、如何实现类似构造器的效果？<br>
通过实例初始化，就像这样：
{% codeblock lang:java %}
new Contents(){
	{print("instance init");}
	.....
}
{% endcodeblock %}
<br>
使用匿名内部类还需要注意以下问题：<br>
匿名内部类中可以直接调用外部定义的对象，但编译器会要求该对象的引用必须是final的，例如：
{% codeblock lang:java %}
public Contents getContents(final String param){
	return new Contents(){
		String label = param;
		.....
	}
}
{% endcodeblock %}