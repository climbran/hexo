title: bash环境变量学习
date: 2015-12-02 12:07:04
tags: [mac os,linux]
---
linux环境变量通常有
	/etc/profile 
	/etc/bashrc 
	~/.bash_profile(或~/.bash_login、~/.profile中的一个，优先级从左到右)
	~/.bashrc
四个文件

一、profile文件登录式shell打开时执行
	/etc/profile 和 ~/.bash_profile 登录（即打开一个login shell）时执行一次。
	/etc/profile先执行，对全部用户和所有类型的shell(如bash、csh、zch等)都有效，而~/.bash_profile只有当前用户的bash会去读，当没有时会依次尝试执行~/.bash_login、 ~/.profile，执行到一个则不再尝试。

二、bashrc文件非登录式shell打开时执行
	/etc/bashrc 和 ~/.bashrc 打开一个非登录式shell（non-login shell）会被读取，同样/etc/bashrc是对所有用户有效，~/.bashrc只对当前用户有效。

三、执行顺序
	/etc/profile先执行，再启动用户目录下的~/.bash_profile或~/.bash_login、 ~/.profile。
	如果~/.bash_profile存在则一般还会执行~/.bashrc，~/.bashrc中还会执行/etc/bashrc。 

四、登录式与非登录式shell
1、登录式（login shell）
	某用户由/bin/login登陆进系统后启动的shell，跟这个用户绑定。这个shell是用户登陆后启动的第一个进程。当bash以login shell启动时，它会执行/etc/profile中的命令，然后/etc/profile调用/etc/profile.d目录下的所有脚本；然后执行~/.bash_profile，~/.bash_profile调用~/.bashrc，最后~/.bashrc又调用/etc/bashrc。
2、非登录式shell(non-login shell)
	不需login而由某些程序启动的shell。还以Bash为例，当以非login方式启动时，它会调用~/.bashrc，随后~/.bashrc中调用/etc/bashrc，最后/etc/bashrc调用所有/etc/profile.d目录下的脚本。这个有兴趣的可以打开这些文件看一看。
	非login的shell主要包括以"#su","#su USERNAME"启动的shell，和图形终端（例如Ubuntu的Terminal），执行的脚本等等。
3、识别登录式与非登录式
	登录式shell在进程启动时传递的第0个参数是 “-shell name”,而非登录式shell时传递的是“shell name”。
	以bash为例，执行#echo $0,如果是login shell会输出 “-bash”;如果是non-login shell则会输出 “bash”。

五、Mac OS
	在Mac OS中，启动一个新的终端窗口就是打开了一个交互式shell,在我的电脑上测试，没有修改配置的情况下，/etc/profile中有：
	
	. /etc/bashrc
执行了/etc/bashrc

六、export命令
	export命令设置的环境变量只是保存在内存中，当shell关闭后就失效了，下次启动会重新读取配置文件中的信息，unset命令同理。