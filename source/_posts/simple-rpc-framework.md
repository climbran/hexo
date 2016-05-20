title: 手把手教你实现SCF框架
date: 2016-03-30 21:37:43
tags: [RPC]
---
先标题党一下，要实现一个完善的RPC框架需要大量的工作，本文只是模拟公司的scf框架实现一个简单的RPC框架。

其基本原理很简单，就是客户端通过动态代理将本地方法包装成网络请求，在服务端通过注解进行服务配置，收到请求后将执行结果返回给客户端。这样在客户端可以像调用本地方法一样获取远程服务。
<img width="500" src="/img/simple-rpc-framework/1.jpg"><br><br>

首先是客户端代理的实现,为了便于理解，网络io部分简单的使用Socket,为提高服务端性能，可通过nio的方式来实现非阻塞的网络io：
{% codeblock lang:java %}
package com.swe.scf.client;

import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.net.InetSocketAddress;
import java.net.Socket;

public class ProxyFactory {

    public static <T> T create(final Class service, final String hostName,final int port) {
        return (T) Proxy.newProxyInstance(service.getClassLoader(),
                new Class[]{service},
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Socket socket = null;
                        ObjectInputStream input = null;
                        ObjectOutputStream output = null;
                        try {
                            socket = new Socket();
                            socket.connect(new InetSocketAddress(hostName, port));
                            output = new ObjectOutputStream(socket.getOutputStream());
                            output.writeUTF(service.getName());
                            output.writeUTF(method.getName());
                            output.writeObject(method.getParameterTypes());
                            output.writeObject(args);
                            input = new ObjectInputStream(socket.getInputStream());
                            return input.readObject();
                        } finally {
                            if (socket != null)
                                socket.close();
                            if (input != null)
                                input.close();
                            if (output != null)
                                output.close();
                        }

                    }
                });
    }
}
{% endcodeblock %}

接下来是服务端的实现，在服务端，我们通过LookUp注解对服务进行配置：
{% codeblock lang:java %}
package com.swe.scf.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface LookUp {
    String value();
}
{% endcodeblock %}

LookUp注解在程序员的好朋友：hello world示例中使用方式如下：
{% codeblock lang:java %}
package com.swe.scf.service;

public interface HelloWorldService {
    String say(String name);
}
{% endcodeblock %}
{% codeblock lang:java %}
package com.swe.scf.service.impl;

import com.swe.scf.annotation.LookUp;
import com.swe.scf.service.HelloWorldService;

@LookUp("HelloWorldService")
public class HelloWorldServiceImpl implements HelloWorldService {

    @Override
    public String say(String name) {
        return "helow " + name;
    }
}
{% endcodeblock %}

当服务端收到请求后通过ServiceExecutor响应请求：
{% codeblock lang:java %}
package com.swe.scf.server;

import com.swe.scf.support.BeanFactory;

import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Method;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;


public class ServiceExecutor {
    private static Executor excutor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

    public static void start(String hostName, int port)throws Exception{
        ServerSocket server = new ServerSocket();
        server.bind(new InetSocketAddress(hostName, port));
        try {
            while (true) {
                excutor.execute(new ProxyTask(server.accept()));
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        finally {
            server.close();
        }
    }

    private static class ProxyTask implements Runnable{
        Socket socket = null;
        ProxyTask(Socket socket){
            this.socket = socket;
        }

        @Override
        public void run() {
            ObjectInputStream input = null;
            ObjectOutputStream output = null;
            try{
                input = new ObjectInputStream(socket.getInputStream());
                String serviceName = input.readUTF();
                String methodName = input.readUTF();
                Class<?> cls = Class.forName(serviceName);

                Object instance = BeanFactory.getBean(cls.getSimpleName());
                Class<?>[] parameterTypes = (Class<?> [])input.readObject();
                Method method =  instance.getClass().getMethod(methodName, parameterTypes);

                Object[] values = (Object[])input.readObject();

                Object result = method.invoke(instance,values);
                output = new ObjectOutputStream(socket.getOutputStream());
                output.writeObject(result);
            }
            catch (Exception e){
                e.printStackTrace();
            }
            finally {
                try {
                    if (input != null) {
                        input.close();
                    }
                    if (output != null) {
                        output.close();
                    }
                    if (socket != null) {
                        socket.close();
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }
}
{% endcodeblock %}
在服务端需要加载和获取service对象实例，要用到ClassUtil、ClassLoaer和BeanFactory三个辅助类：

ClassLoader中通过getServiceImplSet方法获取指定包名下service实现类的Class对象
{% codeblock lang:java %}
package com.swe.scf.support;

import com.swe.scf.annotation.LookUp;
import com.swe.scf.util.ClassUtil;

import java.util.HashSet;
import java.util.Set;


public class ClassLoader {
    private static final Set<Class<?>> CLASS_SET;
    private static String basePackage = "com.swe.scf"; //service实现类所在包路径

    static {
        CLASS_SET = ClassUtil.getClassSet(basePackage);
    }

    public static Set<Class<?>> getClassSet(){
        return CLASS_SET;
    }

    public static Set<Class<?>> getServiceImplSet(){
        Set<Class<?>> classSet = new HashSet<Class<?>>();
        for(Class<?> cls : CLASS_SET){
            if(cls.isAnnotationPresent(LookUp.class))
                classSet.add(cls);
        }
        return classSet;
    }

}
{% endcodeblock %}
ClassLoader中用到的ClassUtil代码如下：
{% codeblock lang:java %}
package com.swe.scf.util;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.FileFilter;
import java.io.IOException;
import java.net.JarURLConnection;
import java.net.URL;
import java.util.Enumeration;
import java.util.HashSet;
import java.util.Set;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

public class ClassUtil {
    private static final Logger logger = LoggerFactory.getLogger(ClassUtil.class);

    /**
     * 获取类加载器
     *
     * @return
     */
    public static ClassLoader getClassLoader() {
        return Thread.currentThread().getContextClassLoader();
    }

    /**
     * 加载类
     *
     * @param className
     * @param isInitialized
     * @return
     */
    public static Class<?> loadClass(String className, boolean isInitialized) {
        Class<?> cls;
        try {
            cls = Class.forName(className, isInitialized, getClassLoader());
        } catch (ClassNotFoundException e) {
            logger.error("load class failure", e);
            throw new RuntimeException(e);
        }
        return cls;
    }

    /**
     * 获取指定包名下的所有类
     *
     * @param packageName
     * @return
     */
    public static Set<Class<?>> getClassSet(String packageName) {
        Set<Class<?>> classSet = new HashSet<Class<?>>();
        try {
            Enumeration<URL> urls = getClassLoader().getResources(packageName.replace(".", "/")); //获取目录下所有资源
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                if (url != null) {
                    String protocol = url.getProtocol(); //获取资源类型
                    if (protocol.equals("file")) {
                        String packagePath = url.getPath().replaceAll("%20", " ");
                        addClass(classSet, packagePath, packageName);
                    } else if (protocol.equals("jar")) {
                        JarURLConnection jarURLConnection = (JarURLConnection) url.openConnection();
                        if (jarURLConnection != null) {
                            JarFile jarFile = jarURLConnection.getJarFile();
                            if (jarFile != null) {
                                Enumeration<JarEntry> jarEntries = jarFile.entries();
                                while (jarEntries.hasMoreElements()) {
                                    JarEntry jarEntry = jarEntries.nextElement();
                                    String jarEntryName = jarEntry.getName();
                                    if (jarEntryName.endsWith(".class")) {
                                        String className = jarEntryName.substring(0, jarEntryName.lastIndexOf(".")).replaceAll("/", ".");
                                        doAddClass(classSet, className);
                                    }
                                }
                            }
                        }
                    }
                }
            }
        } catch (IOException e) {
            logger.error("get class set failure", e);
            throw new RuntimeException(e);
        }
        return classSet;
    }

    private static void addClass(Set<Class<?>> classSet, String packagePath, String packageName) {
        File[] files = new File(packagePath).listFiles(new FileFilter() {
            public boolean accept(File file) {
                return (file.isFile() && file.getName().endsWith(".class")) || file.isDirectory();
            }
        });
        for (File file : files) {
            String fileName = file.getName();
            if (file.isFile()) {
                String className = fileName.substring(0, fileName.lastIndexOf("."));
                if (packageName != null)
                    packageName = packageName.trim();
                if (!StringUtils.isEmpty(packageName))
                    className = packageName + "." + className;
                doAddClass(classSet, className);
            } else {
                String subPackagePath = fileName;
                if (packagePath != null)
                    packageName = packageName.trim();
                if (!StringUtils.isEmpty(packagePath))
                    subPackagePath = packagePath + "/" + subPackagePath;
                String subPackageName = fileName;
                if (packageName != null)
                    packageName = packageName.trim();
                if (!StringUtils.isEmpty(packageName))
                    subPackageName = packageName + "." + subPackageName;
                addClass(classSet, subPackagePath, subPackageName);
            }

        }
    }

    private static void doAddClass(Set<Class<?>> classSet, String className) {
        Class<?> cls = loadClass(className, false);
        classSet.add(cls);
    }
}
{% endcodeblock %}

BeanFactory通过ClassLoader获取Class对象，用来加载和获取service的bean对象：
{% codeblock lang:java %}
package com.swe.scf.support;


import com.swe.scf.annotation.LookUp;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class BeanFactory {
    private static final Logger logger = LoggerFactory.getLogger(BeanFactory.class);
    private static final Map<String,Object> BEAN_MAP = new HashMap<String, Object>() ;

    static {
        Set<Class<?>> beanClassSet = ClassLoader.getServiceImplSet();
        try {
            for (Class<?> beanClass : beanClassSet) {
                LookUp lookUp = beanClass.getAnnotation(LookUp.class);
                Object obj = beanClass.newInstance();
                BEAN_MAP.put(lookUp.value(), obj);
            }
        } catch (Exception e) {
            logger.error("new instance failure", e);
            throw new RuntimeException(e);
        }

    }

    public static Map<String, Object> getBeanMap(){
        return BEAN_MAP;
    }

    public static <T> T getBean(String service){
        int i =1;
        if(!BEAN_MAP.containsKey(service))
            throw new RuntimeException("cant't get bean by service name:" +service);
        return (T)BEAN_MAP.get(service);
    }

}
{% endcodeblock %}

测试代码：
{% codeblock lang:java %}
package com.swe.scf;

import com.swe.scf.client.ProxyFactory;
import com.swe.scf.service.HelloWorldService;
import com.swe.scf.server.ServiceExecutor;

public class Main {

    public static void main(String[] args)throws Exception{
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    ServiceExecutor.start("localhost", 8089);
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }).start();

        HelloWorldService service = ProxyFactory.create(HelloWorldService.class,"localhost",8089);
        System.out.println(service.say("world"));

    }
}
{% endcodeblock %}
客户端通过ProxyFactory.create获取service实例，当执行service中的函数时，通过动态代理发起请求，服务端接受到请求后利用反射执行函数，并将结果序列化后返回。

这样就得到了一个原始的RPC框架，后续可以在io、序列化、服务注册、监控等方面优化形成一个完善的远程调用框架。

	
	
