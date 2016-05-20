title: jetty启动时ArrayIndexOutOfBoundsException:48188
date: 2015-11-24 13:15:37
tags: [web]
---

之前启动jetty时突然开始报下面错误：
{% codeblock %}
2015-11-24 13:15:04.125:WARN:oeja.AnnotationParser:Problem processing jar entry com/ibm/icu/impl/data/LocaleElements_zh__PINYIN.class
java.lang.ArrayIndexOutOfBoundsException: 48188
	at org.objectweb.asm.ClassReader.readClass(Unknown Source)
	at org.objectweb.asm.ClassReader.accept(Unknown Source)
	at org.objectweb.asm.ClassReader.accept(Unknown Source)
	at org.eclipse.jetty.annotations.AnnotationParser.scanClass(AnnotationParser.java:658)
	at org.eclipse.jetty.annotations.AnnotationParser$2.processEntry(AnnotationParser.java:630)
	at org.eclipse.jetty.webapp.JarScanner.matched(JarScanner.java:155)
	at org.eclipse.jetty.util.PatternMatcher.matchPatterns(PatternMatcher.java:82)
	at org.eclipse.jetty.util.PatternMatcher.match(PatternMatcher.java:64)
	at org.eclipse.jetty.webapp.JarScanner.scan(JarScanner.java:78)
	at org.eclipse.jetty.annotations.AnnotationParser.parse(AnnotationParser.java:642)
	at org.eclipse.jetty.annotations.AnnotationParser.parse(AnnotationParser.java:651)
	at org.eclipse.jetty.annotations.AnnotationConfiguration.parseWebInfLib(AnnotationConfiguration.java:341)
	at org.eclipse.jetty.annotations.AnnotationConfiguration.configure(AnnotationConfiguration.java:99)
	at org.eclipse.jetty.webapp.WebAppContext.configure(WebAppContext.java:441)
	at org.eclipse.jetty.webapp.WebAppContext.startContext(WebAppContext.java:1229)
	at org.eclipse.jetty.server.handler.ContextHandler.doStart(ContextHandler.java:699)
	at org.eclipse.jetty.webapp.WebAppContext.doStart(WebAppContext.java:467)
	at org.mortbay.jetty.plugin.JettyWebAppContext.doStart(JettyWebAppContext.java:256)
	at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:59)
	at org.eclipse.jetty.server.handler.HandlerCollection.doStart(HandlerCollection.java:224)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.doStart(ContextHandlerCollection.java:167)
	at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:59)
	at org.eclipse.jetty.server.handler.HandlerCollection.doStart(HandlerCollection.java:224)
	at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:59)
	at org.eclipse.jetty.server.handler.HandlerWrapper.doStart(HandlerWrapper.java:90)
	at org.eclipse.jetty.server.Server.doStart(Server.java:262)
	at org.mortbay.jetty.plugin.JettyServer.doStart(JettyServer.java:65)
	at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:59)
	at org.mortbay.jetty.plugin.AbstractJettyMojo.startJetty(AbstractJettyMojo.java:511)
	at org.mortbay.jetty.plugin.AbstractJettyMojo.execute(AbstractJettyMojo.java:364)
	at org.mortbay.jetty.plugin.JettyRunMojo.execute(JettyRunMojo.java:516)
	at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:132)
	at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:208)
	at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:153)
	at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:145)
	at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject(LifecycleModuleBuilder.java:116)
	at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject(LifecycleModuleBuilder.java:80)
	at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build(SingleThreadedBuilder.java:51)
	at org.apache.maven.lifecycle.internal.LifecycleStarter.execute(LifecycleStarter.java:120)
	at org.apache.maven.DefaultMaven.doExecute(DefaultMaven.java:355)
	at org.apache.maven.DefaultMaven.execute(DefaultMaven.java:155)
	at org.apache.maven.cli.MavenCli.execute(MavenCli.java:584)
	at org.apache.maven.cli.MavenCli.doMain(MavenCli.java:216)
	at org.apache.maven.cli.MavenCli.main(MavenCli.java:160)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced(Launcher.java:289)
	at org.codehaus.plexus.classworlds.launcher.Launcher.launch(Launcher.java:229)
	at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode(Launcher.java:415)
	at org.codehaus.plexus.classworlds.launcher.Launcher.main(Launcher.java:356)
{% endcodeblock %}

从错误信息中基本可以确认是maven里配置的jetty插件的问题，但单单一个数组越界错误很难确认具体是什么问题，百度48188也没什么有用的答案。

虽然一直报错但是对程序运行没影响，再加上项目在赶进度就一直没处理，今天闲下来便好好解决下这个问题。

确定是jetty问题了，就先换个jetty版本试试，从原来的

	<version>8.1.4.v20120524</version>
换成

	<version>9.3.6.v20151106</version>
	
再启动jetty，还是报错，可是错误信息变了!
{% codeblock %}
java.lang.RuntimeException: Error scanning entry com/ibm/icu/impl/data/LocaleElements_zh__PINYIN.class from jar file:///Users/ywq/.m2/com/ibm/icu/icu4j/2.6.1/icu4j-2.6.1.jar
	at org.eclipse.jetty.annotations.AnnotationParser.parseJar(AnnotationParser.java:937)
	at org.eclipse.jetty.annotations.AnnotationParser.parse(AnnotationParser.java:851)
	at org.eclipse.jetty.annotations.AnnotationConfiguration$ParserTask.call(AnnotationConfiguration.java:163)
	at org.eclipse.jetty.annotations.AnnotationConfiguration$1.run(AnnotationConfiguration.java:548)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:654)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:572)
	at java.lang.Thread.run(Thread.java:745)
Caused by: 
java.lang.ArrayIndexOutOfBoundsException: 48188
	at org.objectweb.asm.ClassReader.readClass(Unknown Source)
	at org.objectweb.asm.ClassReader.accept(Unknown Source)
	at org.objectweb.asm.ClassReader.accept(Unknown Source)
	at org.eclipse.jetty.annotations.AnnotationParser.scanClass(AnnotationParser.java:1004)
	at org.eclipse.jetty.annotations.AnnotationParser.parseJarEntry(AnnotationParser.java:984)
	at org.eclipse.jetty.annotations.AnnotationParser.parseJar(AnnotationParser.java:933)
	at org.eclipse.jetty.annotations.AnnotationParser.parse(AnnotationParser.java:851)
	at org.eclipse.jetty.annotations.AnnotationConfiguration$ParserTask.call(AnnotationConfiguration.java:163)
	at org.eclipse.jetty.annotations.AnnotationConfiguration$1.run(AnnotationConfiguration.java:548)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:654)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:572)
	at java.lang.Thread.run(Thread.java:745)
{% endcodeblock %}

原来之前一直没注意到的的WARN信息才是重点，查看Maven Dependencies下果然有个icu4j-2.6.1.jar,在Dependency Hierarchy下发现是jaxen 1.1.1中依赖了icu4j这个jar包，于是把jaxen换成最新的1.1.6版本，重启jetty，问题解决了。

百度时发现不少类似问题但没有答案，都可以参考这样的思路解决。
