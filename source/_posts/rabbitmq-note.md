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

这样一个简单的例子就完成了。更多详细例子，请参考[这里](https://github.com/hewentian/rabbitMQ-demo)。另外，下面这两篇文章，值得我们认真看下：
http://www.rabbitmq.com/amqp-0-9-1-quickref.html
http://www.rabbitmq.com/tutorials/amqp-concepts.html

未完等续。。。
