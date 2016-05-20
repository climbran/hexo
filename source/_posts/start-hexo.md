title: 开始使用hexo
date: 2015-08-16 17:35:32
tags: [hexo]
---
之前尝试折腾过octpress,不过使用起来实在感觉麻烦，今天发现hexo，尝试了一下感觉方便很多，需要安装node.js，基本根据下面两篇文档就能搞定：<br>
<a href="https://hexo.io/zh-cn/docs/">hexo文档</a><br>
<a href="http://www.wuchong.me/jacman/2014/11/20/how-to-use-jacman/">如何使用Jacman主题</a><br>

两个项目作者一个是台湾大学生，一个是北理大学生，膜拜一下<br><br>
配置起来很顺利，就是在配置category和tags插件的时候要生成带有categories和tags的文章后插件才能显示出来，配置好后以为没成功折腾了好一会。<br><br>
配置好后将源文件上传github，方便在其他地方使用。其他电脑上clone下来后需先配置好环境，再在工程目录下运行npm install。<br><br>

-----------2015.12.4更新---------------<br>
使用<a href="https://github.com/coneycode/hexo-generator-baidu-sitemap">hexo-generator-baidu-sitemap</a>生成sitemap,在github上clone后需要在工程目录下运行
{% codeblock %}
$ npm install hexo-generator-baidu-sitemap@0.1.1 --save
{% endcodeblock %}