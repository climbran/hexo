title: http过渡到https实战分享
date: 2016-07-14 22:12:49
tags: [https]
---
>背景：为了防止链路劫持，公司将部分业务从http迁移到https，在线上已有大量业务的情况下，不可能直接整体切换到https，需要进行灰度切换，本文不关注https的实现原理，主要关注如何方便、快速的将线上服务从http灰度切换到https。

一、整体思路
&emsp;&emsp;对于网站来说，主要有最外层的页面、外层页面依赖的内层页面（如iframe）和加载的静态资源（js、css、img）。
&emsp;&emsp;而https页面中的http链接，叫做混合内容（mixed content）。根据混合内容被劫持后可能赵成的危害程度，混合内容又分为Optionally-blockable（如图片）和Blockable（如iframe、js、css、XMLHttpRequest）。其中Optionally-blockable加载时会在浏览器控制台输出警告信息，但能正常显示，Blockable则完全无法加载。
&emsp;&emsp;基于上述原因，我们首先需要让存放静态资源的cdn服务器能支持https访问，其次是内层页面需要支持https，最后是外层页面需要支持https。<br>
二、页面链接
&emsp;&emsp;服务器对域名提供https支持后，下一步是修改页面上的链接。例如 a 标签的href值，会根据uri前面的协议头来确定使用什么协议发起访问。
&emsp;&emsp;切换https最直接的办法当然是直接将协议头替换成https，但这样不能很方便的对不同用户进行灰度切换，如果要为这种方式支持灰度切换只有去牺牲代码的可维护性或进行多次上线。
&emsp;&emsp;还有种方式是在页面增加
{% codeblock lang:html %}
	<meta http-equiv="Content-Security-Policy" content="block-all-mixed-content">
{% endcodeblock %}&emsp;&emsp;这个标签的主要作用是将页面中所有http的混合类型都使用https协议，这种方式主要有两个问题，一是对于一个公司而言，通常是有很多不同站点和服务，可能有的站点不支持https协议，这样一刀在灵活性上有欠缺；第二点同样是灰度切换的问题，为了进行灰度切换需要有选择的为页面输出改标签，这部分代码要么全部切换后再次上线下掉，要么留着影响代码可读性和可维护性。&emsp;&emsp;最终我们选择的方式是协议相对URL(protocol-relative URL)，具体方式是将混合内容uri前面协议头去掉，例如{% codeblock lang:html %}
	<a href="//www.58.com">
{% endcodeblock %}
&emsp;&emsp;它会根据当前页面的访问协议来确定确定使用什么协议来进行访问，这样我们只需要在站点入口处（如登陆页面）根据灰度规则确定用户需要走什么协议，然后302跳转，该用户的后续访问就都会通过该协议，这样只需要一次上线，就能完成整个灰度切换过程。经测试，协议相对URL在js、css、ajax中都能正常使用。<br>
三、 后台跳转
&emsp;&emsp;一般来说，服务部署的tomcat外会由nginx层来做透明代理，我们只在客户端和nginx层之间使用https协议，而在nginx后的内网间通信还是使用http协议，这样可能导致web后台无法获取到正确的协议，使得redirect时全都使用http协议，解决的方式如下：
{% codeblock lang:xml %}
在tomcat的server.xml配置的engine段内增加这个配置
<Valve className="org.apache.catalina.valves.RemoteIpValve" remoteIpHeader="X-Forwarded-For" protocolHeader="X-Forwarded-Proto" protocolHeaderHttpsValue="https"/>

nginx的location里面加上这个
proxy_set_header X-Forwarded-Proto $scheme;
{% endcodeblock %}
&emsp;&emsp;这样在使用HttpServletResponse.sendRedirect时便能获取到客户端实际使用的协议。但需要注意，使用这种配置后在后端或tomcat获取ip时需要使用X-Real-IP（需要nginx中配置）才能获取到客户端ip(X-Forwarded-For会获取到空值)。<br>
四、兼容性问题
&emsp;&emsp;使用https协议无疑需要牺牲低版本浏览器的兼容性问题，主要是window xp下的ie6、7，需要根据实际情况作出取舍，可事先通过埋单统计出站点的浏览器分布情况。<br>
五、性能问题
&emsp;&emsp;我们在https灰度切换过程中使用埋点的方式对比了http和https协议下页面的加载时长，这种方式无法统计到页面开始加载前的网络时延，但由于加载过程中包含很多异步加载的元素，整体加载时长会包含这些元素从请求开始到加载完成的时间，所以该数据可以拿来分析https对比http的性能损失，目前来看整体加载时长增加了10%左右。