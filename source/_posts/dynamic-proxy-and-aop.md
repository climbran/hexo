title: 动态代理及简单的AOP框架的实现
date: 2015-09-22 14:43:37
tags: [设计模式,AOP]
---
面向切面编程（AOP = Aspect Oriented Programming）核心就是动态代理机制。动态代理在实现阶段不需要关系代理哪一个类，不用实现具体某个类的代理类。本文通过AOP框架的实现来学习动态代理。

{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;

/**
 * Created by swe on 15/9/22.
 * 业务接口
 */
public interface Subject {

    public void doSomething(String str);
}
{% endcodeblock%}
{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;

/**
 * Created by swe on 15/9/22.
 * 被代理类
 */
public class RealSubject implements Subject {

    //实际业务
    public void doSomething(String str){
        System.out.println("do something: " +str);
    }
}

{% endcodeblock%}
{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * Created by swe on 15/9/22.
 * 业务处理类，对被代理对象操作进行处理
 */
public class DynamicProxyHandle implements InvocationHandler {
    private Object targer = null;

    public DynamicProxyHandle(Object obj){
        targer = obj;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{

        if(method.getName().equalsIgnoreCase("doSomething")) //设置point cut
            new BeforeAdvice().exec();      //执行前置通知(advie)

        return method.invoke(targer, args);
    }

}
{% endcodeblock%}
{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

/**
 * Created by ywq on 15/9/22.
 * 代理类
 */
public class DynamicProxy<T> {
    public static <T> T newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler handler){
        //这里也可以设置一个aspect(aspect = point cut + advice)

        //根据传入的loader和interfaces生成代理对象的方法,并且生成的方法通过handler的invoke函数处理,返回代理对象
        return (T) Proxy.newProxyInstance(loader, interfaces, handler);
    }
}
{% endcodeblock%}
{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;

/**
 * Created by ywq on 15/9/22.
 */
public interface IAdvice {
    public void exec();
}
{% endcodeblock%}
{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;


/**
 * Created by ywq on 15/9/22.
 */
public class BeforeAdvice implements IAdvice {
    public void exec(){
        System.out.println("do before advice");
    }
}
{% endcodeblock%}

最后客户端进行调用：
{% codeblock lang:java%}
package com.climbran.designPatterns.proxy;

import java.lang.reflect.InvocationHandler;

/**
 * Created by swe on 15/9/22.
 */
public class Client {

    public static void main(String[] args){
        Subject subject = new RealSubject();

        InvocationHandler handler = new DynamicProxyHandle(subject);

        Class<?> cla = subject.getClass();
        Subject proxy = DynamicProxy.newProxyInstance(cla.getClassLoader(), cla.getInterfaces(),handler);

        proxy.doSomething("test");
    }

}
{% endcodeblock%}