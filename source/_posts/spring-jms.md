title: spring集成JMS(ActiveMQ)
date: 2015-09-11 17:59:10
tags: [Spring MVC,jms]
---
一、下载与安装ActiveMQ
	<a href="http://activemq.apache.org/download.html">http://activemq.apache.org/download.html</a>
	我下载的是最新的5.12.0版，解压后安装目录下运行bin/activemq start 
	
二、配置POM文件
	看包括springside在内的很多demo的时候看到都是配置的active-core的依赖，而官网5.12.0下面推荐的配置是activemq-all,没太弄明白，后来突然发现页面有一行小字:
>If you need more fine grained control of your dependencies (activemq-all is an uber jar) pick and choose from the various components activemq-client, activemq-broker, activemq-xx-store etc.

英文不好惹的祸啊，如果是中文早就看到了...往前几个版本的文档看了下，从5.8就开始用activemq-client + activemq-broker替代activemq-core了，大概看了一下active-all,把几个不同版本的active-xx-store包含进去了。

最后我的配置：
{% codeblock pom.xml %}
 
 		<activemq.version>5.12.0</activemq.version>
 		
 		......
 
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
        </dependency>

		<!-- 我配置了slf4j和logback,所以把log4j的依赖exclusion -->
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-client</artifactId>
            <version>${activemq.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-broker</artifactId>
            <version>${activemq.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-jdbc-store</artifactId>
            <version>${activemq.version}</version>
        </dependency>
        
        ......

{% endcodeblock %}

接下来要实现Producer和Consumer

首先是Producer,可以实现MessageProducer接口，但接口中有很多方法不需要实现，所以没有继承该接口：
{% codeblock Producer.java %}
package com.climbran.jms;

import javax.jms.Destination;
import org.springframework.jms.core.JmsTemplate;

/**
 * JMS消息生产者.
 * @author swe
 */

public class Producer {

	private JmsTemplate jmsTemplate;
	private Destination notifyQueue;
	private Destination notifyTopic;

	public void sendQueue(final String message) {
		sendMessage(message, notifyQueue);
	}

	public void sendTopic(final String message) {
		sendMessage(message, notifyTopic);
	}

	private void sendMessage(String message, Destination destination) {

		jmsTemplate.convertAndSend(destination, message);
	}

	public void setJmsTemplate(JmsTemplate jmsTemplate) {
		this.jmsTemplate = jmsTemplate;
	}

	public void setNotifyQueue(Destination notifyQueue) {
		this.notifyQueue = notifyQueue;
	}

	public void setNotifyTopic(Destination nodifyTopic) {
		this.notifyTopic = nodifyTopic;
	}
}

{% endcodeblock %}

接下来实现消费者(Consumer),Spring通过DefaultMessageListenerContainer内部初始化一个taskExecutor来执行监听的任务，这里实现监听器就好了：

{% codeblock MyMessageListener.java %}
package com.climbran.jms;

import javax.jms.MapMessage;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 消息的异步被动接收者.
 * @author swe
 */
public class MyMessageListener implements MessageListener {

	private static Logger logger = LoggerFactory.getLogger(NotifyMessageListener.class);


	/**
	 * MessageListener回调函数.
	 */
	@Override
	public void onMessage(Message message) {
		try {
		
			TextMessage textMessage = (TextMessage) message;
			// 打印消息详情
			logger.info(""+textMessage.getText());

		} catch (Exception e) {
			logger.error("处理消息时发生异常.", e);
		}
	}
}
{% endcodeblock %}

接下来就是配置文件，配置了1个生产者，3个消费者，其中一个监听queue，两个监听topic：
{% codeblock applicationContext-jms.xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <description>JMS配置</description>

    <!-- ActiveMQ 连接工厂 -->
    <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://localhost:61616" />
    </bean>

    <!-- Spring Caching 连接工厂 -->
    <bean id="cachingConnectionFactory" class="org.springframework.jms.connection.CachingConnectionFactory">
        <property name="targetConnectionFactory" ref="connectionFactory" />
        <property name="sessionCacheSize" value="10" />
    </bean>

    <!-- Queue定义 -->
    <bean id="notifyQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="q.notify" />
    </bean>

    <!-- Topic定义 -->
    <bean id="notifyTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="t.notify" />
    </bean>

    <!-- Spring JMS Template -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="cachingConnectionFactory" />
    </bean>

    <!-- 使用Spring JmsTemplate的消息生产者 -->
    <bean id="notifyMessageProducer" class="com.climbran.jms.Producer">
        <property name="jmsTemplate" ref="jmsTemplate" />
        <property name="notifyQueue" ref="notifyQueue" />
        <property name="notifyTopic" ref="notifyTopic" />
    </bean>

    <!-- 异步接收Queue消息Container,DefaultMessageListenerContainer内部初始化建立的一个taskExecutor执行监听的任务-->
    <!-- 相当于配置Consumer,并为Consumer设置了Listenser,同时为Consumer设置了destination-->
    <bean id="queueContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="notifyQueue" />
        <property name="messageListener" ref="notifyMessageListener" />
        <property name="concurrentConsumers" value="10" />
    </bean>

    <!-- 异步接收Topic消息Container -->
    <bean id="topicContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="notifyTopic" />
        <property name="messageListener" ref="notifyMessageListener" />
    </bean>

    <bean id="topicContainer2" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="notifyTopic" />
        <property name="messageListener" ref="notifyMessageListener" />
    </bean>

    <!-- 异步接收消息处理类 -->
    <bean id="notifyMessageListener" class="com.climbran.jms.MyMessageListener" />
</beans>
{% endcodeblock %}

最后业务层调用：
{% codeblock lang:java %}
	producer.sendTopic("a topic message");
    producer.sendQueue("a queue message");
{% endcodeblock %}

结果打印出2条"a topic message"，1条"a queue message"