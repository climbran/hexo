title: javascript延时（setTimeout()）
date: 2015-10-29 17:31:11
tags:  [javascript]
---
window.setTimeout(code,millisec)
window.setTimeout(code,millisec,lang)

code:传入函数或代码
millisec:时间，单位毫秒，1秒=1000毫秒
lang:脚本语言类型(JScript | VBScript | JavaScript)
>window.setTimeout("alert(1);",2000)<br>
window.setTimeout(function(){alert("hello")},2000)<br>

由于javascript是单线程，所以下面函数会进入死循环：
{% codeblock lang:javascript %}
	flag = true;
	window.setTimeout(function(){flag = false;},3000);
	while(flag){	
	}
	alert(ok);
{% endcodeblock %}
