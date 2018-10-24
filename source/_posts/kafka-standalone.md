---
title: kafka 单节点安装
date: 2018-10-24 08:32:40
tags: kafka
categories: bigdata
---

本文将说下`kafka`的单节点安装，我的机器为`Ubuntu 16.04 LTS`，下面的安装过程参考：
http://kafka.apache.org/quickstart

### 第一步：我们要将`kafka`安装包下载回来
截止本文写时，它的最新版本为`2.0.0`，可以在它的[官网][link_id_kafka_2.12-2.0.0.tgz]下载。
``` bash
$ cd /home/hewentian/ProjectD/
$ wget https://www.apache.org/dist/kafka/2.0.0/kafka_2.12-2.0.0.tgz
$ wget https://www.apache.org/dist/kafka/2.0.0/kafka_2.12-2.0.0.tgz.sha512

验证下载文件的完整性，在下载的时候要将 SHA512 文件也下载回来
$ sha512sum -c kafka_2.12-2.0.0.tgz.sha512
kafka_2.12-2.0.0.tgz: OK

$ tar xzf kafka_2.12-2.0.0.tgz
```

### 第二步：启动服务器
kafka需要用到zookeeper，所以必须首先启动zookeeper。在高版本的kafka发行包中，已经内置zookeeper，我们直接使用即可。
``` bash
$ cd /home/hewentian/ProjectD/kafka_2.12-2.0.0/
$ ./bin/zookeeper-server-start.sh config/zookeeper.properties
```
启动成功后，会看到如下输出：

    [2018-10-24 09:14:29,072] INFO Server environment:java.io.tmpdir=/tmp (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,072] INFO Server environment:java.compiler=<NA> (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,072] INFO Server environment:os.name=Linux (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,072] INFO Server environment:os.arch=amd64 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,072] INFO Server environment:os.version=4.13.0-32-generic (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,072] INFO Server environment:user.name=hewentian (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,073] INFO Server environment:user.home=/home/hewentian (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,073] INFO Server environment:user.dir=/home/hewentian/ProjectD/kafka_2.12-2.0.0 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,091] INFO tickTime set to 3000 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,091] INFO minSessionTimeout set to -1 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,091] INFO maxSessionTimeout set to -1 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-10-24 09:14:29,111] INFO Using org.apache.zookeeper.server.NIOServerCnxnFactory as server connection factory (org.apache.zookeeper.server.ServerCnxnFactory)
    [2018-10-24 09:14:29,121] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)

接着，打开另外一个终端，启动kafka服务器：
``` bash
$ cd /home/hewentian/ProjectD/kafka_2.12-2.0.0/
$ ./bin/kafka-server-start.sh config/server.properties
```

启动成功后，会看到如下输出：

    [2018-10-24 11:01:45,462] INFO [SocketServer brokerId=0] Started processors for 1 acceptors (kafka.network.SocketServer)
    [2018-10-24 11:01:45,494] INFO Kafka version : 2.0.0 (org.apache.kafka.common.utils.AppInfoParser)
    [2018-10-24 11:01:45,494] INFO Kafka commitId : 3402a8361b734732 (org.apache.kafka.common.utils.AppInfoParser)
    [2018-10-24 11:01:45,497] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)

### 第三步：创建topic
创建一个名字叫`test`的topic，只有一个分区和一个副本，打开另外一个终端：
``` bash
$ cd /home/hewentian/ProjectD/kafka_2.12-2.0.0/
$ ./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic "test".

查看所有创建的topic
$ ./bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```

### 第四步：往topic发送消息
kafka自带一个命令行的客户端，用于从文件中或者标准输入中读取消息并且发送到kafka集群，默认每一行会被作为一条消息发送：
``` bash
$ cd /home/hewentian/ProjectD/kafka_2.12-2.0.0/
$ ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>This is a message
>This is another message
>
```

### 第五步：消费topic中的消息
kafka同样自带一个命令行的消费者，它会将消息输出到标准输出：
``` bash
$ cd /home/hewentian/ProjectD/kafka_2.12-2.0.0/
$ ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message

```

这样，一个简单的单节点`kafka`服务器就搭建完成了，接下来我们将尝试搭建多节点的集群。

[link_id_kafka_2.12-2.0.0.tgz]: https://www.apache.org/dist/kafka/2.0.0/kafka_2.12-2.0.0.tgz
