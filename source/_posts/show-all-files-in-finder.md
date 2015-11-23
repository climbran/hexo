title: Mac os中显示隐藏文件
date: 2015-11-23 12:45:33
tags: [mac os]
---
在终端中输入如下命令：
{% codeblock lang:shell %}
defaults write ~/Library/Preferences/com.apple.finder AppleShowAllFiles -bool true
{% endcodeblock %}
>true 改成 false 就可以不再显示隐藏文件