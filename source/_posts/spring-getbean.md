title: spring中获取spring容器中的bean
date: 2015-05-06 23:35:26
tags: [Spring MVC]
---
{% codeblock SpringCtx.java %}
public final class SpringCtx {
	private SpringCtx() {
	}
	
	// Spring应用上下文环境  
	private static ApplicationContext context = new ClassPathXmlApplicationContext("/applicationContext.xml"); //new FileSystemXmlApplicationContext("classpath:applicationContext.xml");
 
	public static <T> T getBean(Class<T> requiredType) {
		T t = null;
		try {
			t = context.getBean(requiredType);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return t;
	}
}
{% endcodeblock %}