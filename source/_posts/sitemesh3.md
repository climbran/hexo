title: Sitemesh配置和使用
date: 2015-11-04 20:46:56
tags: [web]
---
一、Sitemesh介绍
Sitemesh是一个网页布局框架，基于servlet中的Filter拦截器，在servlet(如spring)的拦截器外层拦截，对html文件进行装饰后再进入servlet，本文使用最新版本sitemesh3

二、Sitemesh配置
maven配置：
{% codeblock pom.xml %}
<dependency>
    <groupId>org.sitemesh</groupId>
    <artifactId>sitemesh</artifactId>
    <version>3.0.0</version>
</dependency>
{% endcodeblock %}
web.xml配置，注意sitemesh的filter必须配置在servlet的filter之前：
{% codeblock web.xml %}
<web-app>
    ...

    <filter>
        <filter-name>sitemesh</filter-name>
        <filter-class>org.sitemesh.config.ConfigurableSiteMeshFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>sitemesh</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    ...
</web-app>
{% endcodeblock %}
三、sitemsh3.xml配置文件详解
使用sitemesh3需要新建/WEB-INF/sitemesh3.xml
{% codeblock sitemesh3.xml %}
<?xml version="1.0" encoding="UTF-8"?>
<sitemesh>
	<!--默认情况下，sitemesh 只对 HTTP 响应头中 Content-Type 为 text/html 的类型进行拦截和装饰，我们可以添加更多的 mime 类型-->
    <mime-type>text/html</mime-type>
    <mime-type>application/xhtml+xml</mime-type>
    
    <!-- 默认装饰器，当下面的路径都不匹配时，启用该装饰器进行装饰 -->
    <mapping decorator="/default-decorator.html"/>
    
    <!-- 指明满足path的页面，将被decorator所装饰,可配置多个 -->
    <mapping path="/*" decorator="/WEB-INF/views/decorators/decorator.html" />

    <!-- 指明满足path的页面，将被排除，不被装饰,可配置多个 -->
    <mapping path="/exclude.jsp*" exclue="true" />
</sitemesh>
{% endcodeblock %}
四、使用Sitemesh
根据上面的配置，首先新建在/WEB-INF/views下新建decorator.html
{% codeblock /WEB-INF/views/decorator.html %}
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>
    <sitemesh:write property='title' /> - hello world
</title>
<sitemesh:write property='head' />
</head>
<body>
    <header>header</header>
    <hr />
    <sitemesh:write property='title' /><br />
    <sitemesh:write property='body' />
    <hr />
    <footer>footer</footer>
</body>
</html>
{% endcodeblock %}

再新建被装饰的html页面demo.html：
{% codeblock demo.html %}
<!DOCTYPE html>
<html>
<head>
    <title>demo title</title>
    demo head 
</head>
<body>
    demo content
</body>
</html>
{% endcodeblock %}

sitemesh会通过sitemesh:write标签将property在被装饰页面demo.html中相应的标签抽取出来插入sitemesh:write所在位置，上面两个html页面最终通过sitemesh输出的结果如下：
{% codeblock lang:html%}
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>demo title - hello world</title>
demo head 
</head>
<body>
    <header>header</header>
    <hr />
    demo title<br />
    demo content
    <hr />
    <footer>footer</footer>
</body>
</html>
{% endcodeblock %}
五、自定义<sitemesh:write>标签的property属性
Sitemesh3<sitemesh:write>标签默认只提供了body，title，head等property属性：
{% codeblock lang:java %}
import org.sitemesh.SiteMeshContext;
import org.sitemesh.content.ContentProperty;
import org.sitemesh.content.tagrules.TagRuleBundle;
import org.sitemesh.content.tagrules.html.ExportTagToContentRule;
import org.sitemesh.tagprocessor.State;

public class MyTagRuleBundle implements TagRuleBundle {
    @Override
    public void install(State defaultState, ContentProperty contentProperty,
            SiteMeshContext siteMeshContext) {
        defaultState.addRule("sidebar", new ExportTagToContentRule(siteMeshContext,contentProperty.getChild("sidebar"), false));
        
    }
    
    @Override
    public void cleanUp(State defaultState, ContentProperty contentProperty,
            SiteMeshContext siteMeshContext) {
    }
}
{% endcodeblock %}
然后在sitemesh3.xml中增加以下代码：
{% codeblock %}
<content-processor>
	<tag-rule-bundle class="com.lt.common.ext.sitemesh3.MyTagRuleBundle" />
</content-processor>
{% endcodeblock %}
这样就可以在装饰器文件中使用下面代码：

	<sitemesh:write property='sidebar' />