title: 自定义taglib动态生成站点地图
date: 2015-06-20 16:55:07
tags: [java,jsp]
---
一般通过xml文件配置站点地图时每个页面生成的面包屑都是固定的，现在需要动态生成面包屑(eg:首页-文章-文章名，其中文章名需要动态生成)，结合网上生成静态sitemap的代码做出以下实现：
{% codeblock com.climbran.tag.SiteMapTag.java%}
package com.climbran.tag;  
  
import java.io.InputStream;  
import java.util.ArrayList;  
import java.util.Iterator;  
import java.util.List;  
  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.jsp.JspException;  
import javax.servlet.jsp.tagext.TagSupport;  
  
import org.dom4j.Attribute;  
import org.dom4j.Document;  
import org.dom4j.Element;  
import org.dom4j.io.SAXReader;  
  
/** 
 * 站点导航标签的实现 
 * @author swe 
 * 
 */  
public class SiteMapTag extends TagSupport {  
  
    private static final long serialVersionUID = -3531938467909884528L;  
    private String currentFilePath;  
    private Element target;  
    private String dynamicTitle;
    private String dynamicTitle1;
    private String dynamicUrl;
    private String dynamicUrl1;
    private final static String dynamic = "{dynamic}"; 
    private final static String dynamic_1 = "{dynamic1}"; 
 
    @Override  
    public int doStartTag() throws JspException {  
        HttpServletRequest request = (HttpServletRequest) this.pageContext.getRequest();  
        currentFilePath = request.getRequestURI().replaceFirst(request.getContextPath(), "");  
        try {  
            Element root = (Element)pageContext.getServletContext().getAttribute("webSiteMapSet");  
            if(root==null){  
                SAXReader reader = new SAXReader();  
                InputStream inputStream = SiteMapTag.class.getClassLoader().getResourceAsStream("sitemap.xml");  
                Document document = reader.read(inputStream);  
                root = document.getRootElement();  
                pageContext.getServletContext().setAttribute("webSiteMapSet", root);  
            }  
            parseParent(root);  
            StringBuffer content = new StringBuffer("");  
            List<String> titles = new ArrayList<String>();  
            List<String> hrefs = new ArrayList<String>();  
              
            while(target!=null){  
                Attribute attTitle = target.attribute("title");  
                if(attTitle!=null){  
                    titles.add(attTitle.getText());  
                }  
                  
                Attribute attHref = target.attribute("href");  
                if(attHref!=null){  
                    hrefs.add(attHref.getText());  
                }else{  
                    hrefs.add("");  
                }  
                  
                target = target.getParent();  
            }  
            content.append("<div class='breadMenu'><ol class='breadcrumb clearfix'>");
            for (int i = titles.size()-1; i >=0; i--) {  
                String href = hrefs.get(i);  
                if(dynamicUrl!=null)
                	href=href.replace(dynamic, dynamicUrl);
                if(dynamicUrl1!=null)
                	href=href.replace(dynamic_1, dynamicUrl1);
                if(href.equals("")||i==0){  
                	if(titles.get(i).equals(dynamic))
                		content.append("<li>" + dynamicTitle +"</li>"); 
                	else if(titles.get(i).equals(dynamic_1))
                		content.append("<li>" + dynamicTitle1 +"</li>"); 
                	else
                		content.append("<li>" + titles.get(i)+"</li>");  
                }else{  
                	if(titles.get(i).equals(dynamic))
                		content.append("<li><a href='"+href+"'>"+dynamicTitle+"</a></li>");
                	else if(titles.get(i).equals(dynamic_1))
                		content.append("<li><a href='"+href+"'>"+dynamicTitle1+"</a></li>");
                	else
                		content.append("<li><a href='"+href+"'>"+titles.get(i)+"</a></li>");  
                }  
                  
            }  
            content.append("</ol></div>");
            if(content.length()>0){  
                this.pageContext.getOut().println(content.delete(content.length()-0, content.length()));  
            }  
              
  
        } catch (Exception e) {  
            e.printStackTrace();  
            throw new JspException(e);  
        }  
          
          
        return super.doStartTag();  
    }  
  
    private void parseParent(Element parent){  
        Iterator<Element> it = parent.elementIterator();  
        while(it.hasNext()){  
            Element temp = it.next();  
            Attribute attr = temp.attribute("path");  
            if(attr!=null){  
                if(attr.getText().equals(currentFilePath)){  
                    target = temp;  
                    return;  
                }  
            }  
            parseParent(temp);  
        }  
    }

	public String getDynamicTitle() {
		return dynamicTitle;
	}

	public void setDynamicTitle(String dynamicTitle) {
		this.dynamicTitle = dynamicTitle;
	}

	public String getDynamicTitle1() {
		return dynamicTitle1;
	}

	public void setDynamicTitle1(String dynamicTitle1) {
		this.dynamicTitle1 = dynamicTitle1;
	}

	public String getDynamicUrl() {
		return dynamicUrl;
	}

	public void setDynamicUrl(String dynamicUrl) {
		this.dynamicUrl = dynamicUrl;
	}

	public String getDynamicUrl1() {
		return dynamicUrl1;
	}

	public void setDynamicUrl1(String dynamicUrl1) {
		this.dynamicUrl1 = dynamicUrl1;
	}      
}  
{% endcodeblock %}

{% codeblock src/main/resources/sitemap.xml%}
<?xml version="1.0" encoding="UTF-8"?>
  
<!-- 配置文件说明， sitemap根节点,title是页面上显示的内容，  
    href是超链接，path是该请求页面的物理路径  
-->  

<sitemap title="首页" href="/home"> 
	<node path="/WEB-INF/views/article/index.jsp" href="/article" title="全部文章">
		<node path="/WEB-INF/views/article/new.jsp" href="/article/new" title="发表文章"/>
		<node path="/WEB-INF/views/article/show.jsp" href="/article/{dynamic}" title="{dynamic}"/><!-- 文章名称 -->
	</node>
</sitemap>
{% endcodeblock %}

{% codeblock WEB-INF/tlds/sitemap.tld lang:xml%}
<?xml version="1.0" encoding="UTF-8"?>
<taglib xmlns="http://java.sun.com/xml/ns/javaee"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"  
    version="2.1">  
      
    <description>站点导航标签</description>  
    <display-name>myTaglib siteMap</display-name>  
    <tlib-version>1.1</tlib-version>      
    <short-name>tagUtil</short-name>  
    <uri>http://tag.climbran.com/jsp/tagutil</uri>  
  
    <tag>  
        <description>面包屑标签</description>  
        <name>siteMap</name>  
        <tag-class>com.climbran.tag.SiteMapTag</tag-class>  
        <body-content>empty</body-content>  
        <attribute>
			<description>title变量</description>
			<name>dynamicTitle</name>
			<rtexprvalue>true</rtexprvalue>
		</attribute>
		<attribute>
			<description>url变量</description>
			<name>dynamicUrl</name>
			<rtexprvalue>true</rtexprvalue>
		</attribute>
		<attribute>
			<description>title1变量</description>
			<name>dynamicTitle1</name>
			<rtexprvalue>true</rtexprvalue>
		</attribute>
		<attribute>
			<description>url1变量</description>
			<name>dynamicUrl1</name>
			<rtexprvalue>true</rtexprvalue>
		</attribute>
    </tag>  
          
</taglib> 
{% endcodeblock %}

{% codeblock WEB-INF/views/article/show.jsp lang:html %}
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@taglib uri="http://tag.climbran.com/jsp/tagutil" prefix="tagUtil" %>
<head>
	<title>article</title>
</head>
<body>
	<tagUtil:siteMap dynamicUrl="${article.id}" dynamicTitle="${article.title}"/>	
</body>		
{% endcodeblock %}