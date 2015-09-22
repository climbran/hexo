title: 利用Filter统计时间开销
date: 2015-03-20 15:50:55
tags: [Spring MVC]
---
项目需要在日志中记录每个请求花费的时间开销，算是开始web开发的第一个任务，度娘到Filter这个东西，做出以下是实现：
{% codeblock lang:java %}

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class RequestLogFilter implements Filter {

	Logger logger = LoggerFactory.getLogger("@@@Monitor@@@");

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		// TODO Auto-generated method stub

	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {

		long startTime = System.currentTimeMillis();
		try {
			chain.doFilter(request, response);
		} catch (Exception e) {
			throw e;
		} finally {
			long stopTime = System.currentTimeMillis();
			long costTime = stopTime - startTime;
			String requestURL = ((HttpServletRequest) request).getRequestURI();
			
			if (costTime > 200) {
				logger.warn("start[{}] time[{}] tag[{}] message[]", startTime,
						costTime, requestURL);
			} else {
				logger.debug("start[{}] time[{}] tag[{}] message[]", startTime,
						costTime, requestURL);
			}
		}
	}

	@Override
	public void destroy() {
		// TODO Auto-generated method stub

	}

}

{% endcodeblock %}

在web.xml中增加配置:
{% codeblock %}
	<filter>
		<filter-name>RequestLogFilter</filter-name>
		<filter-class>com.lemon.web.filter.RequestLogFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>RequestLogFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
{% endcodeblock %}
此处得注意filterChain的顺序，filterChain根据filter在web.xml中的顺序从上往下，在一个栈中执行。