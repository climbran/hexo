title: Spring MVC中文乱码问题
date: 2015-05-17 13:26:27
tags: [tomcat,Spring MVC]
---
Spring MVC请求时经常出现中文乱码，查看配置文件，web.xml中已配置：
<pre>
<xmp><filter>  
	<filter-name>CharacterEncodingFilter</filter-name>  
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
    <init-param>  
        <param-name>encoding</param-name>  
        <param-value>utf-8</param-value>  
    </init-param>
</filter> 
<filter-mapping>  
    <filter-name>CharacterEncodingFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
</xmp></pre>
度娘后发现该配置只对post请求有效，对于get请求还要对tomcat的service.xml进行配置，具体配置如下：
<pre><xmp><Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" useBodyEncodingForURI="true"/></xmp></pre>
上面配置对ajax的get请求无效，如果要同时对ajax请求有效则改为
<pre><xmp><Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" URIEncoding="UTF-8"/></xmp></pre>