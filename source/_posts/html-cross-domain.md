title: ajax跨域请求html
date: 2015-10-26 13:06:01
tags: [web,ajax]
---
上周接到一个需求，需要在页面中获取另一个html页面后嵌入当前页面，由于直接用iframe直接请求会多出一些乱起八糟的标签并且不方便编辑样式，于是选择使用ajax。

最初直接通过ajax请求，firefox中查看返回的状态码是200，但一直静入ajax的error分支，通过firebug查看发现是跨域问题，如果是跨域请求json用jsonp处理就好，可以跨域请求html还没有遇到过。

百度后了解到需要通过后台来使跨域请求变成同域，但没找到具体实施方案，于是先尝试直接后台redirect目标页面，再ajax请求后台，结果ajax接收302状态码，进入error分支，接着把ajax换成用complete接收，接收到一个object对象，打印出来是一堆js，便放弃这一方案，不知道有没变法通过ajax正确处理302？

最后是通过HttpClient请求页面，再通过response.getWriter获取PrintWriter输出到前段（没有使用Spring框架），在Spring中对应的实现如下
{% codeblock lang:java%}
package com.climbran.spring.controller;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.impl.conn.tsccm.ThreadSafeClientConnManager;
import org.apache.http.util.EntityUtils;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.apache.http.client.HttpClient;


import javax.servlet.http.HttpServletResponse;

/**
 * Created by swe on 15/10/26.
 */
@Controller
@RequestMapping("ajax")
public class AjaxTestController {

    @RequestMapping("getHTML")
    @ResponseBody
    public String getHTML(HttpServletResponse response){

        HttpClient httpclient = new DefaultHttpClient(new ThreadSafeClientConnManager());//创建一个客户端，类似打开一个浏览器
        HttpGet httpGet = new HttpGet("http://404.58.com/");
        String html = null;
        try {
            HttpResponse res = httpclient.execute(httpGet);
            HttpEntity entity = res.getEntity();
            html = EntityUtils.toString(entity);
            System.out.println(res.getStatusLine().getStatusCode());
            System.out.println(html);
        }
        catch (Exception e){
            e.printStackTrace();

        }
        response.setContentType("text/html;charset=UTF-8");
        return html;
    }
}
{% endcodeblock %}

前端代码如下：
{% codeblock lang:html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
  <script type="text/javascript" src="http://localhost:8080/static/js/jquery-1.8.3.min.js"></script>

  <title>Test</title>
</head>
<body>
  <div class="dd"></div>

<script type="text/javascript">

  $.ajax({
    type : 'get',
    url : '/ajax/getHTML',
    dataType : 'html',
    data:{
    },
    success : function(data) {
      $(".dd").html(data);
    }
  });
</script>
</body>
</html>

{% endcodeblock %}

HttpClient依赖如下：
{% codeblock lang:xml %}
<dependency>
	<groupId>org.apache.httpcomponents</groupId>
	<artifactId>httpclient</artifactId>
	<version>4.5.1</version>
</dependency>
{% endcodeblock %}