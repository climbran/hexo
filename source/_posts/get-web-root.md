title: Spring获取webapp路径
date: 2016-02-25 15:13:11
tags: [Spring MVC,webapp]
---
通过ServletContext对象获取路径：
{% codeblock %}
	WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();
	ServletContext servletContext = webApplicationContext.getServletContext();
	String webapp = servletContext.getRealPath("/");
{% endcodeblock %}
	