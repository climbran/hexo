title: static块
date: 2015-05-29 17:03:15
tags: [java]
---
看springside的showcase发现static块的直接使用:
{% codeblock lang:java %}
package org.springside.examples.showcase.web;

import java.util.List;
.....

@Controller
@RequestMapping(value = "/account/user")
public class UserController {

	private static Map<String, String> allStatus = Maps.newHashMap();

	static {
		allStatus.put("enabled", "有效");
		allStatus.put("disabled", "无效");
	}
	
	......

}
{% endcodeblock %}
<br>
static{}，会在类被加载的时候执行且仅会被执行一次，一般用来初始化静态变量和调用静态方法。<br><br>
虚拟机的生命周期中一个类只被加载一次；又因为static{}是伴随类加载执行的，所以，不管你new多少次对象实例，static{}都只执行一次。<br><br>
类会在以下情况加载:<br>
1、用Class.forName()显示加载的时候，如上面的示例一;<br>
2、实例化一个类的时候，如将main()函数的内容改为:Test t=new Test();//这种形式其实和1相比，原理是相同的，都是显示的加载这个类，读者可以验证Test t=new Test();和Test t=(Test)Class.forName().newInstance();这两条语句效果相同。<br>
3、调用类的静态方法的时候，如将main()函数的内容改为:Test.display();<br>
4、调用类的静态变量的时候，如将main()函数的内容改为:System.out.println(Test.X);<br>
<br><br>
以下两种情况类不会加载:<br>
1、调用类的静态常量的时候，是不会加载类的，即不会执行static{}语句块，读者可以自己验证一下(将main()函数的内容改为System.out.println(Test.Y);)，你会发现程序只输出了一个200；(这是java虚拟机的规定，当访问类的静态常量时，如果编译器可以计算出常量的值，则不会加载类，否则会加载类)<br>
2、用Class.forName()形式的时候，我们也可以自己设定要不要加载类，如将Class.forName("Test")改为 Class.forName("Test",false,StaticBlockTest.class.getClassLoader())，你会发现程序什么都没有输出，即Test没有被加载，static{}没有被执行。<br><br>
当带satic的静态初始化块与不带staic的非静态初始化块同时需要执行时，按照以下顺序执行：<br>
所有类的静态初始化块(从父类自顶向下) --> 所有类的普通初始化块(从父类自顶向下) --> 类的构选器(从父类自顶向下)