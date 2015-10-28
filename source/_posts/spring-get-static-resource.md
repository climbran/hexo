title: Spring访问静态资源(js、css、图片等)
date: 2015-10-28 11:23:52
tags: [Spring MVC]
---

在一个新搭建的spring测试框架中，执行ajax时没有返回，发现是jquery的js文件加载时返回404错误，猜测是spring配置文件中缺少静态文件路径的相关配置，查看后果然是如此。查找后了解到静态文件有3种配置方式：

####方法一：在web.xml中增加静态资源的url
该方法是通过web容器来访问静态资源，需要写再DispatcherServlet前面，让web容器的servlet在spring之前先拦截，这种那个方式不能访问WEB-INF目录，只能访问web根目录下的路径（webapp、webContent等）：
{% codeblock lang:xml %}
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.jpg</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.js</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>*.css</url-pattern>
</servlet-mapping> 
{% endcodeblock %}
>各种常见web容器的servlet名称如下：<br>
Tomcat、Jetty、JBoss、GlassFish	: "default"<br>
Google App Engine	:	"_ah_default"<br>
Resin	:	"resin-file"<br>
WebLogic	:	"FileServlet"<br>
WebSphere	:	"SimpleFileServlet"

####方法二：Spring MVC中配置<mvc:resources>
spring3.0.4后提供的方式
{% codeblock lang:xml %}
<mvc:resources mapping="/images/**" location="/images/" />
{% endcodeblock %}
将mapping的url映射到ResourceHttpRequestHandler进行处理，location指定静态资源位置，可以使webapp根目录下，也可以使WEB-INF目录和jar包里面，这样可以把静态资源压缩到jar包种

####方法三：Spring MVC中配置使用<mvc:default-servlet-handler/>
{% codeblock lang:xml %}
<mvc:default-servlet-handler/>
{% endcodeblock %}
这种方式只需要一行代码，但也不能访问WEB-INF目录下

方法二和方法三还需要同时配置

	<mvc:annotation-driven/>