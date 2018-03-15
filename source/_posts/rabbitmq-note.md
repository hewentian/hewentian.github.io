---
title: rabbitMQ学习笔记
date: 2018-01-17 12:06:58
tags: rabbitmq
categories: bigdata
---
在安装好`rabbitMQ`后，执行`rabbitmq-plugins enable rabbitmq_management`命令，开启Web管理插件，这样我们就可以通过浏览器来进行管理了。默认地址为：
http://localhost:15672/
并使用默认用户guest登录，密码也为guest。这个guest用户只能在安装rabbitMQ的机器上登录，如果要在其他机器登录，则用guest登录后，再创建其他用户即可，创建的用户要授权，否则用API是无法访问的，有可能会报错。因为新创建的用户默认是没有权限访问`/`的，可以在WEB上面授权，或者用命令授权。
列出用户权限
``` bash
$ sudo rabbitmqctl list_users

Listing users ...
hewentian [administrator]
guest [administrator]

授权
$ sudo rabbitmqctl set_permissions -p / hewentian '.*' '.*' '.*'
```
该命令使用户hewentian具有`/`这个virtual host中所有资源的配置、写、读权限以便管理其中的资源

其使用也是非常容易入门的：
在spring boot项目的application.properties中配置关于rabbitMQ的连接和用户信息
``` xml
spring.application.name=rabbitmq-demo

spring.rabbitmq.host=10.1.32.97
spring.rabbitmq.port=5672
spring.rabbitmq.username=hewentian
spring.rabbitmq.password=12345678

```
创建消息生产者Sender，将消息发送到`myqueue`这个队列
``` java
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class Sender {

	@Autowired
	private AmqpTemplate rabbitTemplate;

	public void send() {
		String context = "hello " + new Date();
		this.rabbitTemplate.convertAndSend("myqueue", context);
	}
}
```
创建消息消费者Receiver，对队列`myqueue`进行监听
``` java
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "myqueue")
public class Receiver {

	@RabbitHandler
	public void process(String ctx) {
		System.out.println("Receiver : " + ctx);
	}
}
```
创建RabbitMQ的配置类RabbitConfig
``` java
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

	@Bean
	public Queue helloQueue() {
		return new Queue("myqueue");
	}
}
```

这样一个简单的例子就完成了。更多详细例子，请参考[这里](https://github.com/hewentian/mq-demo)。另外，下面这两篇文章，值得我们认真看下：
http://www.rabbitmq.com/amqp-0-9-1-quickref.html
http://www.rabbitmq.com/tutorials/amqp-concepts.html

The default exchange is a direct exchange with no name (empty string) pre-declared by the broker.

#### Direct Exchange
Direct exchanges are often used to distribute tasks between multiple workers (instances of the same application) in a round robin manner. When doing so, it is important to understand that, in AMQP 0-9-1, messages are load balanced between consumers and not between queues.

#### Fanout Exchange
A fanout exchange routes messages to all of the queues that are bound to it and the routing key is ignored. If N queues are bound to a fanout exchange, when a new message is published to that exchange a copy of the message is delivered to all N queues. Fanout exchanges are ideal for the broadcast routing of messages.

Because a fanout exchange delivers a copy of a message to every queue bound to it, its use cases are quite similar:

* Massively multi-player online (MMO) games can use it for leaderboard updates or other global events
* Sport news sites can use fanout exchanges for distributing score updates to mobile clients in near real-time
* Distributed systems can broadcast various state and configuration updates
* Group chats can distribute messages between participants using a fanout exchange (although AMQP does not have a built-in concept of presence, so XMPP may be a better choice)

#### Topic Exchange
Topic exchanges route messages to one or many queues based on matching between a message routing key and the pattern that was used to bind a queue to an exchange. The topic exchange type is often used to implement various publish/subscribe pattern variations. Topic exchanges are commonly used for the multicast routing of messages.

Topic exchanges have a very broad set of use cases. Whenever a problem involves multiple consumers/applications that selectively choose which type of messages they want to receive, the use of topic exchanges should be considered.

Example uses:

* Distributing data relevant to specific geographic location, for example, points of sale
* Background task processing done by multiple workers, each capable of handling specific set of tasks
* Stocks price updates (and updates on other kinds of financial data)
* News updates that involve categorization or tagging (for example, only for a particular sport or team)
* Orchestration of services of different kinds in the cloud
* Distributed architecture/OS-specific software builds or packaging where each builder can handle only one architecture or OS

#### Headers Exchange
A headers exchange is designed for routing on multiple attributes that are more easily expressed as message headers than a routing key. Headers exchanges ignore the routing key attribute. Instead, the attributes used for routing are taken from the headers attribute. A message is considered matching if the value of the header equals the value specified upon binding.

It is possible to bind a queue to a headers exchange using more than one header for matching. In this case, the broker needs one more piece of information from the application developer, namely, should it consider messages with any of the headers matching, or all of them? This is what the "x-match" binding argument is for. When the "x-match" argument is set to "any", just one matching header value is sufficient. Alternatively, setting "x-match" to "all" mandates that all the values must match.

Headers exchanges can be looked upon as "direct exchanges on steroids". Because they route based on header values, they can be used as direct exchanges where the routing key does not have to be a string; it could be an integer or a hash (dictionary) for example.

#### Consumers
Storing messages in queues is useless unless applications can consume them. In the AMQP 0-9-1 Model, there are two ways for applications to do this:

* Have messages delivered to them ("push API")
* Fetch messages as needed ("pull API")

With the "push API", applications have to indicate interest in consuming messages from a particular queue. When they do so, we say that they register a consumer or, simply put, subscribe to a queue. It is possible to have more than one consumer per queue or to register an exclusive consumer (excludes all other consumers from the queue while it is consuming).

#### Connections
AMQP connections are typically long-lived. AMQP is an application level protocol that uses TCP for reliable delivery. AMQP connections use authentication and can be protected using TLS (SSL). When an application no longer needs to be connected to an AMQP broker, it should gracefully close the AMQP connection instead of abruptly closing the underlying TCP connection.


未完待续。。。
