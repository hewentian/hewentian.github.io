---
title: kafka 集群的搭建
date: 2018-10-27 13:11:43
tags: kafka
categories: bigdata
---

本篇将说说kafka集群的搭建，如果你只是想简单体验一下kafka，可以直接使用我在上一篇介绍的 [kafka 单节点安装][link_id_kafka-standalone] 即可。

但是，如果你想在生产环境中使用，那么搭建一个集群可能更适合你。下面将说说kafka集群的安装使用，kafka同样是使用前面例子使用的`2.0.0`版本，我在一台机器上安装，所以这是伪集群，当修改为真集群的时候，只要将IP地址修改下即可，下面会说明。

首先，你得搭建 zookeeper 集群，因为高版本的kafka中内置了zookeeper组件，所以我们直接使用kafka中内置的zookeeper组件搭建zookeeper集群。但是，你也可以使用zookeeper独立的安装包来搭建zookeeper集群。两者的搭建方法都是一样的，可以参考 [zookeeper集群版安装方法][link_id_zookeeper-install-cluster]

### 计划在一台`Ubuntu Linux`服务器上部署3台`kafka`服务器，分别为`kafka1`, `kafka2`, `kafka3`
因为三台`kafka`服务器的配置都差不多，所以我们先设置好一台`kafka1`的配置，再将其复制成`kafka2`, `kafka3`并修改其中的配置即可。

下面使用kafka内置的zookeeper组件搭建zookeeper集群，我们将kafka的所有服务器都放在同一个目录下：

### 1.建目录，如下：
``` bash
$ cd /home/hewentian/ProjectD
$ mkdir kafkaCluster
```

### 2.将`kafka_2.12-2.0.0.tgz`放到`/home/hewentian/ProjectD/kafkaCluster`目录下，并执行如下脚本解压
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster
$ tar xzvf kafka_2.12-2.0.0.tgz

$ ls
kafka_2.12-2.0.0  kafka_2.12-2.0.0.tgz

$ rm kafka_2.12-2.0.0.tgz
$ mv kafka_2.12-2.0.0/ kafka1  # 为方便起见，将其命名为 kafka1

$ cd kafka1/
$ mkdir -p data/zk     # 存放zookeeper数据的目录
$ mkdir -p data/kafka  # 存放kafka数据的目录
$ mkdir logs           # 新解压的 kafka 没有此目录，需手动创建。因为重定向的日志logs/zookeeper.log需要此目录
```

### 3.修改`/home/hewentian/ProjectD/kafkaCluster/kafka1/config/zookeeper.properties`并在其中修改如下内容：
	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/home/hewentian/ProjectD/kafkaCluster/kafka1/data/zk  # 这里必须为绝对路径，否则有可能无法启动
	clientPort=2181                                               # 这台服务器的端口为2181这里为默认值
	server.1=127.0.0.1:2888:3888
	server.2=127.0.0.1:2889:3889
	server.3=127.0.0.1:2890:3890


### 4.在`/home/hewentian/ProjectD/kafkaCluster/kafka1/data/zk`目录下建`myid`文件并在其中输入1，只输入1，代表server.1
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1/data/zk
$ vi myid
```
这样第一台服务器已经配置完毕。


### 5.接下来我们将`kafka1`复制为`kafka2`, `kafka3`
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster
$ cp -r kafka1 kafka2
$ cp -r kafka1 kafka3
```

### 6.将`kafka2/data/zk`目录下的`myid`的内容修改为2
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2/data/zk
$ vi myid
```
同理，将将`kafka3/data/zk`目录下的`myid`的内容修改为3
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3/data/zk
$ vi myid
```

### 7.修改`kafka2`的配置文件
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2/config
$ vi zookeeper.properties
```
仅修改两处地方即可，要修改的地方如下：

	dataDir=/home/hewentian/ProjectD/kafkaCluster/kafka2/data/zk  # 这里是数据保存的位置
	clientPort=2182                                               # 这台服务器的端口为2182

同理，修改`kafka3`的配置文件
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3/config
$ vi zookeeper.properties
```
仅修改两处地方即可，要修改的地方如下：

	dataDir=/home/hewentian/ProjectD/kafkaCluster/kafka3/data/zk  # 这里是数据保存的位置
	clientPort=2183                                               # 这台服务器的端口为2183


### 8.到目前为此，我们已经将3台`zookeeper`服务器都配置好了。接下来，我们要将他们都启动
启动kafka1的zookeeper服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1
$ nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties > logs/zookeeper.log 2>&1 &
```
启动kafka2的zookeeper服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2
$ mkdir logs
$ nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties > logs/zookeeper.log 2>&1 &
```
启动kafka3的zookeeper服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ mkdir logs
$ nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties > logs/zookeeper.log 2>&1 &
```

### 9.当三台服务器都启动好了，我们分别连到三台zookeeper服务器：
连接到kafka1的zookeeper服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1
$ ./bin/zookeeper-shell.sh 127.0.0.1:2181
```
连接到kafka2的zookeeper服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2
$ ./bin/zookeeper-shell.sh 127.0.0.1:2182
```
连接到kafka3的zookeeper服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ ./bin/zookeeper-shell.sh 127.0.0.1:2183
```

可以通过查看`logs/zookeeper.log`文件，如果没有报错就说明zookeeper集群启动成功。


这样你在`kafka1`中的`zookeeper`所作的修改，都会同步到`kafka2`, `kafka3`。
例如你在kafka1的zookeeper服务器
``` bash
$ create /zk_test_cluster my_data_cluster
```
你在kafka2, kafka3的zookeeper客户端用
``` bash
$ ls /
```
都会看到节点zk_test_cluster

至此，zookeeper集群部署结束。

### 10.搭建kafka集群
配置`kafka1`服务器：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1
$ vi config/server.properties

broker.id=1                           # 这里设置为1，另外两台分别设置为2、3

listeners=PLAINTEXT://127.0.0.1:9092  # IP地址和端口，这里使用默认的 9092，另外两台分别使用9093、9094

log.dirs=/home/hewentian/ProjectD/kafkaCluster/kafka1/data/kafka

zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```

配置`kafka2`服务器：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2
$ vi config/server.properties

broker.id=2

listeners=PLAINTEXT://127.0.0.1:9093

log.dirs=/home/hewentian/ProjectD/kafkaCluster/kafka2/data/kafka

zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```

配置`kafka3`服务器：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ vi config/server.properties

broker.id=3

listeners=PLAINTEXT://127.0.0.1:9094

log.dirs=/home/hewentian/ProjectD/kafkaCluster/kafka3/data/kafka

zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
```

### 11.启动三台kafka服务器
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1
$ ./bin/kafka-server-start.sh -daemon /home/hewentian/ProjectD/kafkaCluster/kafka1/config/server.properties

$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2
$ ./bin/kafka-server-start.sh -daemon /home/hewentian/ProjectD/kafkaCluster/kafka2/config/server.properties

$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ ./bin/kafka-server-start.sh -daemon /home/hewentian/ProjectD/kafkaCluster/kafka3/config/server.properties
```

分别从三台kafka服务器中查看启动日志`logs/server.log`，如果没报错，并且看到如下输出，则启动成功：

	# kafka1 的输出
	[2018-10-27 15:48:54,890] INFO Kafka version : 2.0.0 (org.apache.kafka.common.utils.AppInfoParser)
	[2018-10-27 15:48:54,890] INFO Kafka commitId : 3402a8361b734732 (org.apache.kafka.common.utils.AppInfoParser)
	[2018-10-27 15:48:54,895] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)

	# kafka2 的输出
	[2018-10-27 15:49:22,694] INFO Kafka version : 2.0.0 (org.apache.kafka.common.utils.AppInfoParser)
	[2018-10-27 15:49:22,694] INFO Kafka commitId : 3402a8361b734732 (org.apache.kafka.common.utils.AppInfoParser)
	[2018-10-27 15:49:22,697] INFO [KafkaServer id=2] started (kafka.server.KafkaServer)

	# kafka3 的输出
	[2018-10-27 15:49:41,746] INFO Kafka version : 2.0.0 (org.apache.kafka.common.utils.AppInfoParser)
	[2018-10-27 15:49:41,746] INFO Kafka commitId : 3402a8361b734732 (org.apache.kafka.common.utils.AppInfoParser)
	[2018-10-27 15:49:41,749] INFO [KafkaServer id=3] started (kafka.server.KafkaServer)

至此，kafka集群搭建成功。下面，我们简单的试用一下。

### 12.创建topic
在任意一台kafka服务器上面创建topic，例如在kafka1上面创建一个名为 my-replicated-topic 的 topic，指定 1 个分区，3 个副本：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1
$ ./bin/kafka-topics.sh --create --zookeeper 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183 --replication-factor 3 --partitions 1 --topic my-replicated-topic

Created topic "my-replicated-topic".
```
上面的参数`--zookeeper`是集群列表，可以指定所有节点，也可以指定为部分列表。

查看topic的情况：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka1
$ ./bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2181 --topic my-replicated-topic

Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 3	Replicas: 3,2,1	Isr: 3,2,1
```

### 13.发送消息
往我们刚才创建的toipc中发送消息，在任意一台kafka上面都可以的，我们在kafka2上面执行：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2
$ ./bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --topic my-replicated-topic
>
>my test message 1
>my test message 2
>

```

### 14.消费消息
将我们刚刚发送的消息消费掉，我们从kafka3上面执行：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ ./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --from-beginning --topic my-replicated-topic

my test message 1
my test message 2

```
我们在生产者中发送消息，在消费者中就能实时的看到消息。


### 15.容错测试
从上面可知my-replicated-topic的leader为3，那我们将broker.id=3的进程杀掉：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ ps -ef | grep kafka3/config/server.properties
hewenti+ 22018  1897  5 17:19 pts/23   00:00:16 /usr/local/java/jdk1.8.0_102/bin/java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true

[中间省略部分]

-0.10.jar:/home/hewentian/ProjectD/kafkaCluster/kafka3/bin/../libs/zookeeper-3.4.13.jar kafka.Kafka /home/hewentian/ProjectD/kafkaCluster/kafka3/config/server.properties

$ kill -9 22018       # 单机环境下不能通过执行： ./bin/kafka-server-stop.sh 来杀掉当前目录下的kafka，它会杀掉全部kafka
```

再查看my-replicated-topic的情况：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ ./bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2181 --topic my-replicated-topic
Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 1	Replicas: 3,2,1	Isr: 1
```
由上面可见，leader已经变为1。并且，生产消息和消费消息一样可用，不受影响：
``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka2
$ ./bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --topic my-replicated-topic
>
>my test message 1
>my test message 2
>
> Tim Ho
>
```

``` bash
$ cd /home/hewentian/ProjectD/kafkaCluster/kafka3
$ ./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --from-beginning --topic my-replicated-topic

my test message 1
my test message 2

Tim Ho

```

未完，待续……


[link_id_kafka-standalone]: ../../../../2018/10/24/kafka-standalone/
[link_id_zookeeper-install-cluster]: ../../../../2017/12/06/zookeeper-install-cluster/
