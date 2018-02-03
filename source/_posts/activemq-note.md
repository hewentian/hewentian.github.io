---
title: activeMQ学习笔记
date: 2018-01-23 20:46:34
tags: activemq
categories: bigdata
---

### Installation Procedure for Unix
Unix Binary Installation
This procedure explains how to download and install the binary distribution on a Unix system.
**NOTE**: There are several alternative ways to perform this type of installation.

1. Download the activemq zipped tarball file to the Unix machine, using either a browser or a tool, i.e., wget, scp, ftp, etc. for example:
(see [Download](http://activemq.apache.org/download.html) -> "The latest stable release")
``` bash
$ wget http://activemq.apache.org/path/tofile/apache-activemq-x.x.x-bin.tar.gz
```

2. Extract the files from the zipped tarball into a directory of your choice. For example:
``` bash
$ cd [activemq_install_dir]
$ tar zxvf activemq-x.x.x-bin.tar.gz
```

### Starting ActiveMQ
On Unix:
From a command shell, change to the installation directory and run ActiveMQ as a foregroud process:
``` bash
$ cd [activemq_install_dir]/bin
$ ./activemq console
```
From a command shell, change to the installation directory and run ActiveMQ as a daemon process:
``` bash
$ cd [activemq_install_dir]/bin
$ ./activemq start
```

### Stopping ActiveMQ
For both Windows and Unix installations, terminate ActiveMQ by typing "CTRL-C" in the console or command shell in which it is running.
If ActiveMQ was started in the background on Unix, the process can be killed, with the following:
``` bash
$ cd [activemq_install_dir]/bin
$ ./activemq stop
```

### Testing the Installation
Using the administrative interface

* Open the administrative interface
* URL: http://127.0.0.1:8161/admin/
* Login: admin
* Passwort: admin
* Navigate to "Queues"
* Add a queue name and click create
* Send test message by klicking on "Send to"

### Listen port
ActiveMQ's default port is 61616. From another window run netstat and search for port 61616.From a Unix command shell, type:
``` bash
$ netstat -nl | grep 61616
tcp6       0      0 :::61616                :::*                    LISTEN
```

部分示例可以在[这里](https://github.com/hewentian/mq-demo)找到。

### 安全问题
ObjectMessage objects depend on Java serialization of marshal/unmarshal object payload. This process is generally considered unsafe as malicious payload can exploit the host system. That's why starting with versions 5.12.2 and 5.13.0, ActiveMQ enforces users to explicitly whitelist packages that can be exchanged using ObjectMessages.

ActiveMQ从5.12.2、5.13.0开始，如果传送的消息是一个`JavaBean`就要设置一个传输对象包名的白名单列表，否则会报如下错：

	javax.jms.JMSException: Failed to build body from content. Serializable class not available to broker. Reason: java.lang.ClassNotFoundException: Forbidden class java.lang.Integer! This class is not trusted to be serialized as ObjectMessage payload. Please take a look at http://activemq.apache.org/objectmessage.html for more information on how to configure trusted classes.
	at org.apache.activemq.util.JMSExceptionSupport.create(JMSExceptionSupport.java:36)
	at org.apache.activemq.command.ActiveMQObjectMessage.getObject(ActiveMQObjectMessage.java:213)
	at com.hewentian.activemq.bean.Consumer$1.onMessage(Consumer.java:48)
	at org.apache.activemq.ActiveMQMessageConsumer.dispatch(ActiveMQMessageConsumer.java:1404)
	at org.apache.activemq.ActiveMQSessionExecutor.dispatch(ActiveMQSessionExecutor.java:131)
	at org.apache.activemq.ActiveMQSessionExecutor.iterate(ActiveMQSessionExecutor.java:202)
	at org.apache.activemq.thread.PooledTaskRunner.runTask(PooledTaskRunner.java:133)
	at org.apache.activemq.thread.PooledTaskRunner$1.run(PooledTaskRunner.java:48)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

根据提示，我们打开[http://activemq.apache.org/objectmessage.html](http://activemq.apache.org/objectmessage.html)，可以发现解决此报错的详细说明。解决方法有二：
**注意： 我是用方法二解决的，方法一试了，无效。**

1. The `setTrustedPackages()` method allows you to set the list of trusted packages you want to be to unserialize, like
``` java
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");
factory.setTrustedPackages(new ArrayList(Arrays.asList("org.apache.activemq.test,com.hewentian.activemq.bean".split(","))));
```

2. The `setTrustAllPackages()` allows you to turn off security check and trust all classes. It's useful for testing purposes.
``` java
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");
factory.setTrustAllPackages(true);
```

如果是springBoot工程，你也可以在`application.properties`作如下设置：
``` java
spring.activemq.packages.trust-all=true

or

spring.activemq.packages.trusted=<package1>,<package2>,<package3>
```

### 消息的消费者接收消息可以采用两种方式：

1. consumer.receive() 或 consumer.receive(int timeout)；
2. 使用setMessageListener。
采用第一种方式，消息的接收者会一直等待下去，直到有消息到达，或者超时。后一种方式会注册一个监听器，当有消息到达的时候，会回调它的onMessage()方法。
