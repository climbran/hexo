title: Java RTTI学习
date: 2015-08-31 23:46:53
tags: [java]
---
一、什么是RTTI
RTTI就是Run-Time Type Information的缩写，Java使用Class对象来实现RTTI。

二、如何实现RTTI
每当编写并且编译一个新类时，就会产生一个Class对象（保存在同名的.class文件中），获取Class对象有以下几种方式：

1、Class.forName("className");
2、类字面常量：eg: User.class;
不光可用于普通类，也可用于接口、数字、基本数据类型。
特殊的，基本数据类型的类字面常亮等价于对应包装类的TYPE字段（eg:int.class和Integer.TYPE等价）；
3、如果已经有了一个对象的引用（非Class对象），可以通过调用该对象的.getClass()方法获取；

使用类的准备工作实际包括3个步骤：
1、加载：这是由类加载器执行的，该步骤将查找字节码（通常在classpath所指定的路径中查找，但这并非必须的），并从这些字节码中创建一个Class对象；
2、链接：在链接阶段将验证类中的字节码，为静态域分配存储空间，并且如果必需的话，将创建这个类对其他类的所有引用；
3、初始化：如果该类具有超类，则对其初始化，执行静态初始化器和静态初始化块；

通过Class.forName("className")方式创建Class对象的引用时会初始化Class对象，而.class方式不会初始化Class对象，具体什么时候初始化可参考：<a href="http://climbran.github.io/2015/05/29/static-block/">satic块</a>

三、Class对象的常用方法
getName	获取全限定名（即带包名的类名）
getSimpleName()	获取不含包名的类名
getCononicalName()	获取全限定名
getInterfaces() 返回Class对象，他们表示表示调用该方法的Class对象中的全部接口
isInterface() 告诉你这个Class对象是否表示某个接口
newInstance() 实例化一个对象

调用newInstance时，如果使用泛型时，用？通配、super通配或不使用泛型将返回Object类型的值，如果用extends通配或指定具体类型能返回具体类型的值，例如:
{% codeblock lang:java %}
	.....
   	Class c1 = User.class;
   	Class<?> c2 = User.class;
   	Class<? super User> c3 = User.class; //super通配父类，但不知道是具体哪个父类，所以向上转型到Object
   	Class<User> c4 = User.class;
   	Class<? extends User> c5 = User.class;
    try {
        User nu1 = (User)c1.newInstance();	//返回值为Object类型，需要强制类型转换
        User nu2 = (User)c2.newInstance();
        User nu3 = (User)c3.newInstance(); //直接返回User类型，不需要强制类型转换
        User nu4 = c4.newInstance();
        User nu5 = c5.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
{% endcodeblock %}

四、instanceof
两种方式直接上示例：
{% codeblock lang:java %}
public class User{}
	
public class Administrator exnteds User{}
	
public class Test{
	public static void main(String[] args){
		Administrator admin = new Administrator();
		
		//方式一
		if(admn instanceof User)
			System.out.println("admin is instance of User");		
		//方式二
		Class c = User.class;
		if(c.isInstanceof(admin))
			System.out.println("admin is also instance of User");
	}	
}	
{% endcodeblock %}

注意instanceof不仅可用于类，还可以用于接口，当对象为null时返回false