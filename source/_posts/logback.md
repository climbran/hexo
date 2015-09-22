title: logback配置
date: 2015-03-29 11:24:45
tags: [thirdparty]
---
为什么选logback:
1、更快的执行速度，基于log4j，logbakc重写了内部实现。
2、充分的测试。
3、logback-classic很好的实现了slf4j,便于迁移消息框架

配置如下：
{% codeblock pom.xml %}
	......
	
	<properties>
        ......
        <slf4j.version>1.7.8</slf4j.version>
        <logback.version>1.1.2</logback.version>
	</properties>
	
    <dependencies>	
    
    	......
		<!-- 项目不依赖于slf4j的具体实现(如logback、log4j、common-logging)，更换log模块时只要换掉对应的jar包即可 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        
		<!-- logback依赖 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- 第三方代码中调用log4j会被桥接到slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
            <scope>runtime</scope>
        </dependency>
        <!-- 第三方代码直接调用common-logging会被桥接到slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
            <scope>runtime</scope>
        </dependency>
        <!-- 第三方代码直接调用java.util.logging会被桥接到slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <version>${slf4j.version}</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- 示例：第三方包中引用commons-logging时通过exclusion属性去掉commons-logging依赖，再通过上面的配置将代码中commons-logging的引用桥接到slf4j上 -->
        <dependency>
			<groupId>org.seleniumhq.selenium</groupId>
			<artifactId>selenium-remote-driver</artifactId>
			<version>${selenium.version}</version>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>cglib</groupId>
					<artifactId>cglib-nodep</artifactId>
				</exclusion>
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
    </dependencies>
{% endcodeblock%}

{% codeblock src/main/resources/logback.xml %}
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss" />

	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
			<charset>UTF-8</charset> <!-- 此处设置字符集 -->
		</encoder>
	</appender>

 	<appender name="rollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>logs/web.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>logs/web.%d{yyyy-MM-dd}.log</fileNamePattern>
		</rollingPolicy>
		<encoder>
			<pattern>%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
	
	<appender name="requestLogFile" class="ch.qos.logback.core.rolling.RollingFileAppender">  
	 	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <fileNamePattern>logs/request.%d{yyyy-MM-dd}.log</fileNamePattern>  
            <maxHistory>30</maxHistory>  
        </rollingPolicy> 
	 	<encoder>
	 		<pattern>%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} %n%msg%n</pattern>
	 	</encoder>
	</appender> 
	 
	<!-- project default level -->
	<logger name="com.climbran.spring" level="INFO" />

	<!--log4jdbc -->
	<logger name="jdbc.sqltiming" level="INFO"/>

	<root level="INFO">
		<appender-ref ref="console" />
		<appender-ref ref="rollingFile" />
		<!--appender-ref ref="requestLogFile" /-->
	</root>
</configuration>
{% endcodeblock%}
{% codeblock Demo.java%}
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Demo {
	protected final Logger log = LoggerFactory.getLogger(this.getClass());
	
	publi  void test(){
		log.info("this is a test log");
	}
}
{% endcodeblock %}
做好以上配置后就配置成功了，但是很好奇logback.xml文件怎么没有配置路径，系统怎么知道我用的是什么消息框架？
	
看了会springside的quickstart项目，看完所有配置文件都没有找到相关配置，没有加载任何相关bean。后来突然反应过来这是个很傻X的问题，在执行Logger log = LoggerFactory.getLogger(this.getClass());时可能就自动去加载logback.xml文件了。<br>
按这个思路百度了一下，果然在maven项目下，logback会在src目录下按logback.groovy(这个是使用groovy语法的配置文件)、logback-test.xml、logback.xml的顺序自动加载，加载到一个就停止，如果都没有配置 则默认调用BasicConfigurator,创建一个最小化配置，所以项目配置中不需要显示配置路径。

默认最小化配置如下

    输出格式 %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n  
    输出方向：System.out
    输出级别：Debug

对于非maven项目，需要在logging.properties中添加：

	handlers = org.slf4j.bridge.SLF4JBridgeHandler 




