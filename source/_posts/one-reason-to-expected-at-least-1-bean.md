title: Autowired时找不到bean的一个原因
date: 2015-08-29 15:20:30
tags: [Spring MVC]
---
今天整合spring和mybatis时，在通过@Autowired注入Dao时一直报
{% codeblock %}
expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
{% endcodeblock %}
查看了很久applicationContext.xml中的配置没有找出问题，最后对照以前项目的web.xml配置时发现少了一个：
{% codeblock lang:xml %}
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
{% endcodeblock %}
加上后问题解决。<br>

ContextLoaderListener通过读取contextConfigLocation参数来读取配置参数，一般来说它配置的是Spring项目的中间层,服务层组件的注入，装配，AOP。