title: linux定时任务(crontab)
date: 2015-12-10 17:05:56
tags: [linux,定时任务,crontab]
---
Linux下可通过crontab命令来执行定时任务：

一.新增定时调度任务
 1、crontab -e 然后添加任务，wq保存退出，修改的/var/spool/cron目录下的文件，为当前用户增加定时任务
 2、vi /etc/crontab，wq保存退出，为系统增加定时任务
cron服务会每分钟读一次/var/spool/cron目录下的所有文件，还会读一次/etc/crontab文件
 

二.查看定时调度任务
 1、crontab -l 列出所有定时任务
 2、crontab -l -u username 列出用户的所有定时任务
 
三.删除定时调度任务
 1、crontab -r  删除所有定时任务
 2、crontab -r -u username 删除特定用户的任务
 
四.定时调度任务任务格式
	  Minute Hour  Day  Month Dayofweek   command
      分钟    小时  天   月     天每星期       命令
     每个字段代表的含义如下：
     Minute             每个小时的第几分钟执行该任务，0-59
     Hour               每天的第几个小时执行该任务，0-23
     Day                每月的第几天执行该任务，1-31
     Month              每年的第几个月执行该任务，1-12
     DayOfWeek     	     每周的第几天执行该任务，0-6，0表示周日
     Command            指定要执行的程序
     
   在这些字段里，除了“Command”是每次都必须指定的字段以外，其它字段皆为可选字段，可视需要决定。对于不指定的字段，要用“*”来填补其位置
{% codeblock %}  
   "-"，如八到十二点每个小时的30分: 30 8-12 * * *
   "/"，如每15分钟: */15 * * * *
   ","，如9点和12点: * 9,12 * * *
{% endcodeblock %}