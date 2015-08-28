title: hexo中使用代码块
date: 2015-08-28 14:29:53
tags: [hexo]
---
使用hexo后一直用&lt;pre>加&lt;code>标签表示代码块，但是一遇到'<'符号就出先各种奇怪错误，还每次情况都不一样，今天在一篇文档发现正确使用方式：
{% codeblock %}
{ %codeblock [title] [lang:language] [url] [link text] % }
code snippet
{ % endcodeblock % }
{% endcodeblock %}

注意要去掉花括号与%之间的空格
