title: java的引用传递
date: 2016-03-16 21:05:34
tags: [java]
---
java是否有引用传递，这个理解好几次了，但是每次要用的时候都有点忘记，在此记录下。

java可以说有引用传递，因为传入一个对象时，有将对象地址传入，对于对象的操作可以保留。

java也可以说没有引用传递，因为传入的地址只是对象的一个拷贝，将拷贝的地址修改后对于对象的操作不会影响到原对象。

例如：
{% codeblock lang:java %}
public void changeName(Person a){
	a = new Person();
	a.name = "xixi";
}

public static void main(String[] args){
	Person a = new Person();
	a.name = "haha";
	changeName(a);
	System.out.println(a.name);
	//输出结果：haha
}
{% endcodeblock %}