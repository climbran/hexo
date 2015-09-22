title: JMS入门
date: 2015-09-11 12:16:35
tags: [jms]
---
一、什么是JMS

JMS(java messgae service)是SUN开发的java消息系统的api,提供异步通信的服务。可用于订单处理、邮寄发送等耗时较长的操作，在减少加快响应时间的同时又能避免开多线程带来的同步问题(多线程在等待请求的io操作或其他服务器响应时一直占有临界资源，而异步调用将请求发送出去后就不用再关心)。JMS还能最大化降低应用程序与应用系统之间的耦合度。(消息发送方和接收方之间解耦)

应用场景示例：
电商网站大量用户提交订单时，使用正常的同步方式处理订单可能受订单处理服务器性能瓶颈影响，导致用户等待时间过长，当并发量太大时甚至会导致服务器的崩溃。这时使用JMS可以将用户请求都加入到消息队列中，然后给用户返回订单处理中。然后订单处理服务器逐个的处理消息队列中的订单。

二、JMS两种消息模型

1. 点到点（P2P）模型
生产者将消息发送到队列(Queue)中，消费者从队列中取出消息（eg:订单处理中，多台订单处理服务器从消息队列中取出订单）

>特点：
	(1)每条只能被接收一次，如果一条消息被一个消费者接收，那么其他的消费者就不能得到这条消息了。
	(2)只要消息没过期且没被其他消费者接受，消费者可能在任何时间接收消息队列中的消息，与生产者消费者运行先后无关。
	(3)收到消息后消费者必须确认消息已被接收，否则JMS服务提供者会认为该消息没有被接收。程序可以自动进行确认，不需要人工干预。
	(4)非持久的消息最多只发送一次,持久的消息严格发送一次。	
2. 发布/订阅（Pub/Sub）模型
生产者发布消息到某一主题(Topic),与主题相关的消费者都会收到消息（eg:系统发送系统邮件都某个用户组，该用户组的用户都会收到消息）

>特点：
	(1)每条消息可以有多个消费者，如果报纸和杂志一样，谁订阅了谁都可以获得。
	(2)订阅者只能接受他们订阅之后出版的消息,即订阅者必须先运行，再等待生产者的运行。
	
三、主要对象

* **Destination:**消息发送的目的地，即Queue或Topic
* **Message:**被发送的消息，有StreamMessage、MapMessage、TextMessage、ObjectMessage、BytesMessage、XMLMessage,最常用的是*TextMessage*(普通字符串，包含一个String)和*ObjectMessage*(包含一个可序列化的java对象)
* **Session:**与JMS建立的会话，通过Session才能创建Message
* **Connection:**与JMS建立的连接，可以从连接创建Session
* **ConnectionFactory:**用来创建Connection
* **Producer:**消息的生产者，用来发送消息,通过Session创建
* **MessageConsumer:**消息的消费者，用来接收消息,通过Session创建

基于ActiveMQ的示例如下：
{% codeblock lang:java %}
ConnectionFactory factory = new ActiveMQConnectionFactory("vm://localhost");
Queue queue = new ActiveMQQueue("demoQueue");

Connection connection = factory.createConnection();
connection.start();

Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);

Message message = session.createTextMessage("Hello world");
MessageProducer producer = session.createProducer(queue);
producer.send(message);

MessageConsumer consumer = session.createConsumer(queue);
Message receive = comsumer.receive();
{% endcodeblock %}

四、JMS的同步调用与异步调用

1. 同步调用
{% codeblock lang:java %}
	consumer.receive()或consumer.receive(init timeout)
{% endcodeblock %}
消息接受者会一直等待，直到有消息或超时

2. 异步调用
	通过MessageListener
{% codeblock lang:java %}	
	MessageConsumer comsumer = session.createConsumer(queue);  
	comsumer.setMessageListener(new MessageListener(){  
           @Override  
           public void onMessage(Message m) {  
               TextMessage textMsg = (TextMessage) m;  
               try {  
                   System.out.println(textMsg.getText());  
               } catch (JMSException e) {  
                   e.printStackTrace();  
               }  
           }           
    }); 
{% endcodeblock %}   
通过监听器监听，当有消息到达时调用onMessage方法 
   