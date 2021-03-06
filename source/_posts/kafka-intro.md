---
title: kafka 介绍
date: 2018-11-12 09:04:49
tags: kafka
categories: bigdata
---

这是一篇译文，因英文水平有限，翻译未免有不足之处。如果想看原文，请访问这里：
http://kafka.apache.org/intro

** 如果想简单体验一下kafka，可以阅读我上两篇介绍的 [kafka 单节点安装][link_id_kafka-standalone]、[kafka 集群的搭建][link_id_kafka-cluster] **


### Apache Kafka是一个分布式的流媒体平台，那么，它到底指的是什么呢？

流媒体平台有3个主要的性能指标：

1. 发布和订阅消息流，类似于消息队列或者企业消息系统；
2. 以容错方式、持久化存储流数据；
3. 实时处理流数据。


kafka通常应用于两种广泛的场景：

1. 在系统或应用程序之间构建可靠的用于传输实时数据的管道; 
2. 构建实时的流数据处理程序，来转换或处理数据流。


### 为了弄清楚kafka到底是怎样完成这些功能的，从下面开始我们钻研和探究一下kafka的功能

首先，了解一下几个概念：

1. kafka可以以集群方式运行于一台或者多台服务器，这些服务器可以分布在不同的数据中心；
2. kafka集群将流式数据分类存储，这种类别通常被称为主题；
3. 每一条消息由键、值和时间戳组成。


![](/img/kafka-1.png "来源：http://kafka.apache.org/20/images/kafka-apis.png")

kafka有4个核心API：

1. [Producer API][link_id_producer_api]允许应用程序将一条消息发布到一个或者多个kafka主题中；
2. [Consumer API][link_id_consumer_api]允许应用程序订阅一个或者多个主题，并且处理被发往其中的数据流；
3. [Streams API][link_id_streams_api]允许一个应用充当流式处理器：从作为输入流的一个或多个主题中消费消息，然后将处理过的消息输出到另一个或多个主题中。高效地将输入流的数据转换、传输到输出流中；
4. [Connector API][link_id_connector_api]允许建立和重用已有的生产者或消费者，它们连接着某个kafka主题，而这些主题是和已存在的应用和数据系统连接着的。例如，关系数据库的连接器将会捕获对表的每一个修改。

在kafka中，客户端和服务器端的通讯是通过一个简单、高效、语言无关的[TCP协议][link_id_tcp_protocol]完成的。此协议是有版本代差的，但新版本向后兼容旧版本。我们提供一个JAVA客户端连接kafka，但是其他语言的客户端也提供。消费者和服务端建立的是长连接。


### 主题和日志（存储策略）
我们首先钻研一下kafka中为处理流记录而提供的核心抽象概念--主题。

主题就是一个分类，或者说是专为发布消息而命名的。在kafka中主题通常有多个订阅者，也就是说一个主题可以有零个、一个或者多个消费者，这些消费者都订阅写往其中的消息。

对每一个主题，kafka集群都使用分区存储，像下面这样：
![](/img/kafka-2.png "来源：http://kafka.apache.org/20/images/log_anatomy.png")

每一个分区中的消息都是按顺序存储的，持续往该分区中存放的数据的顺序都是不可改变的，结构化存储。分区中的每一条消息都会被分配一个有序的ID号，被称为偏移量，用于唯一标示该分区中的每一条消息。

kafka集群会持久化发布到它的每一条消息，无论它们是否已经被消费过，可以通过配置文件配置该消息存放多久。例如，如果保存策略被设置为2天，那么当一条消息发布2天之内，它都是可以被消费的，只是一旦被消费之后，它就会被删掉以释放空间。kafka是持续高性能的，这与存储于它的数据大小关系不大，因此长期保存数据，都是没问题的。


![](/img/kafka-3.png "来源：http://kafka.apache.org/20/images/log_consumer.png")

实际上，唯一存储于每一个消费者中的元数据是偏移量或者该消费者在这个分区中访问存储数据的位置。偏移量由消费者控制：通常，消费者中保存的偏移量随着它消费消息，将呈线性增长。但是，实际上，由于这个偏移量是由消费者控制的，所以它可以指定它消费的任何位置上的消息。例如：一个消费者可以重置到一个旧的偏移量来处理旧的数据或者跳过大部分记录，然后从当前位置开始消费消息。每个消费者的偏移量在kafka服务器中也是有存储的。

这个组合功能意味着kafka消费者是轻量级的，它们的连接和断开对集群和其他消费者影响极小。例如，你可以使用我们的命令行工具去不停地显示最新添加到某个主题的内容，而这，不会对任何订阅这个主题的消费者产生影响。

分区在存储中扮演着不同的目的：首先，它允许存储的数据量超过单台服务器允许的规模。每个单独的分区能储存的数据量取决于它所在的服务器磁盘的大小等因素，但是一个主题可以有多个分区，因此它能储存任意数量的数据。其次，分区充当并行处理的单元--同时能处理的并发数。


### 分布式
一个主题的分区分布于kafka集群中的多台服务器中，每一台服务器都可以处理数据和向共享分区发送请求。为了容错，每一个分区都可以配置一定的副本数。

每一个分区都有一台服务器担当主服务器，可以有零个或者多个从服务器。主服务器处理对分区的所有读和写请求，从服务器由主服务器调度。如果主服务器挂掉了，从服务器中会自动产生一个新的主服务器。集群中的每一台服务器既充当某个分区的主服务器，又充当其他分区的从服务器，所以整个集群是负载均衡的。


### 地域复制
kafka的MirrorMaker为你的集群提供跨地域复制支持。使用MirrorMaker，消息可以跨越多个数据中心或者不同的云区进行同步。你可以在主从模式下用于备份和恢复，或者在主主模式下使数据更靠近你的用户，或者支持数据本地请求。


### 生产者
生产者往它们选定的主题中发送消息的时候，应该为每一条消息指定它要发送到的分区。我们可以使用环形策略，简单地使数据平均分配于所有分区中，也可以根据消息中的语义来自动地选择分区。接下来将会说下分区的使用。


### 消息者
我们可以对消费者分组，每个组有一个组名。每一条发送到指定主题中的消息，都会被订阅了这个主题的同一个组中的一个消费者消费。同一个组中的消费者可以在同一台机器或者多台机器中。

如果所有消费者实例都在同一个组中，那么所有消息都会高效地平均发送到所有消费者实例。
如果所有消费者实例分布在不同的组中，那么每条消息都会被广播到所有组中的一个消费者。

![](/img/kafka-4.png "来源：http://kafka.apache.org/20/images/consumer-groups.png")

上图所示：该kafka集群有2台服务器，4个分区（P0-P3），有2个消费者组。消费者组A有2个消费者实例，而消费者组B有4个。

通常，主题都会有少量的消费者组，在逻辑上看，一个消费者组就是一个订阅者。每个组包含多个消费者，这能很好的实现扩展和容错。在订阅的语义上：订阅者只不过是一群消费者，而不是一个。

消费的方式在kafka中的实现是通过将分区分配给所有消费者实例，因此在任何时刻，每一个实例都是一个”公平共享“分区的唯一消费者。维护组中成员关系的方式在kafak中是通过kafka协议自动实现的：如果新的实例加进组，那么它将从其他组员中获取一个分区（如果这个组员处理两个以上分区）；如果一个实例挂掉了，那么它所处理的分区将被分配给组中剩下的成员们。

kafka对每一个分区中的消息都只提供一个总的顺序，同一个主题中不同分区中的顺序各不相同。每个分区排序组织该分区中数据的能力能满足大部分应用的需求。但是，如果你想要一个所有消息的总顺序，可以通过为这个主题设置一个分区来实现，不过，这意味着一个消费者组中只能有一个消费者来处理该主题的消息。


### 多租户架构
你可以以多租户架构方式部署kafka。多租户架构可以通过配置，来指定哪个主题可以生产和消费数据，并且支持设置操作指标。管理员可以定义和限制所有请求的指标，以控制客户端能使用的服务器资源数。

更多相关信息，请访问[这里](https://kafka.apache.org/documentation/#security)。


### 保证
在高层次看kafka提供以下保证：

1. 一个生产者发往指定主题中指定分区中的消息，将会按照它们发送的顺序出现。例如：如果一个生产者发送消息M1、M2，如果M1先发送，那么M1将会首先出现在那个分区中，并且M1的顺序号要比M2的小；
2. 消费者按顺序读取存储在主题中的消息；
3. 如果一个主题有 N 个副本，那么我们能承受高达 N-1 台服务器同时挂掉，而不会丢失任何消息。

更多关于这些保证的描述将在相关章节中详细说明。


### kafka作为一个消息系统
kafka的流媒体概念与传统的企业消息系统相比有什么不同？

消息传输在传统上有2种模型：队列和发布-订阅。在队列模型中，一组消费者从一台服务器读取消息，每个消息只会发送到其中一个消费者；在发布-订阅模型中，每条消息都会广播到所有消费者。这两种模型各自都有优势和不足。队列的优势是允许你将消息平均分配给所有消费者处理，这能扩展系统的处理能力。不幸的是，队列不能有多个订阅者，消息一旦被其中一个消费者读取就会被删掉。发布-订阅模型允许你将消息广播到所有消费者，这种方式不能扩展处理能力，因为每条消息都会被发送到所有订阅者。

消费者组的概念在kafka中通常包含上述两种概念。对队列来说，消费者组允许你将数据平均分配给所有消费者组来处理；对发布-订阅来说，kafka允许你将消息广播到所有消费者组。

kafka模型的优势是每个主题都有这两种属性：它能扩展处理能力，同时也支持多个订阅者。我们不需要选择其中一个，或者另外一个。

kafka相比于传统的消息系统，它更能保证消息的顺序。

传统队列会将消息按顺序保存在服务器上，如果多个消费者同时消费这个队列，那么服务器将按消息的存储顺序来分发给消费者。然而，尽管服务器按顺序分发消息，但是消息是异步的发送到每个消费者的，所以不同的消费者接收到消息的顺序可能不同。这意味着，在并行处理的情况下，消息的顺序将不能保证。在消息系统中通常有一个概念：唯一消费者，它允许只有一个消费者消费一个队列，当然这也意味着这种情况下不存在并行处理。

kafka在这方面做得比较好。在主题中，它有一个并行的概念--分区。kafka既能保证消息的顺序，又能在多个消费者之间保持负载均衡。通过将主题的分区分配给指定的消费者组，每个分区只能被消费者组中的一个消费者消费，来实现的。这样，我们能确保这个消费者是这个分区的唯一消费者和按顺序消费这个分区中的消息。尽管主题有很多个分区，我们仍能在多个消费者实例之间保持负载均衡。值得注意的是，消费者组中消费者的数量不能多于分区数。


### kafka作为一个存储系统
任何允许发布与消费消息分离的消息队列，实际上充当了目前使用的消息存储系统。kafka的不同之处在于它还是一个非常优秀的存储系统。

写入kafka的数据将会写入磁盘，并且进行副本复制以实现容错。kafka允许生产者等待确认，在收到回复之后才会认为写成功，并且即使写入的服务器失败了，也能保证这条消息是存在的。

kafka能很好地使用磁盘结构来扩容：无论服务器上有 50KB 还是 50TB 的持久化数据，kafka的性能都是一样的，不会随着数据的增多而出现性能下降。

由于kafka可以大规模的存储数据，并且允许客户控制其读取位置，您可以将kafka作为一种专用于高性能、低延迟提交日志存储，并且能复制和传播的分布式文件系统。

更多关于kafka的提交日志存储和副本复制的设计，请访问[这里](https://kafka.apache.org/documentation/#design)。


### kafka作为一个流媒体处理系统
kafka仅仅提供读取、写入和存储数据流是不够的，最终目的是实现流的实时处理。

在kafka中，流处理器是指持续地从输入主题获取数据流，对获取到的数据流执行某种处理，并将处理过的数据，持续地输出到输出主题中。

例如，零售店的应用程序可能会将销售额和货物作为输入流，通过相关计算，然后输出重新排序和根据此数据计算的价格调整的流。

可以直接使用生产者和消费者的相关API进行简单处理。但是，对于更复杂的转换，kafka提供了完全集成的Streams API。这允许构建一些应用程序去执行非普通处理任务、计算流的聚合或者将流连接在一起。

它能有效地解决此类应用程序面临的难题：处理无序数据，在代码更改时重新处理输入流，执行有状态计算等。

流式API构建在kafka提供的核心基础功能上：它使用生产者和消费者API进行输入，使用kafka进行有状态存储，并在流处理器实例之间使用相同的组机制来实现容错。


### 把碎片整合在一起
将消息传递、存储和流处理组合在一起可能看起来没多大用处，但它对于kafka作为流媒体平台的作用至关重要。

像HDFS这种分布式文件系统允许存储静态文件以进行批处理。kafka系统是高效的，它允许存储和处理过去的历史数据。

传统的企业消息系统允许处理你订阅之后到达的数据。以这种方式构建的应用程序只能处理在它订阅之后到达的未来数据。

kafka结合了这两种功能，这种组合对于kafka作为流媒体应用程序平台以及流数据管道的使用至关重要。

通过组合存储和低延迟订阅，流应用程序可以以相同的方式处理过去和未来的数据。也就是说，单个应用程序也可以处理历史存储的数据，而不是在它处理到达最后一条记录时结束，它可以在未来数据到达时继续处理。这就是流处理包含批处理以及消息驱动应用程序的一般概念。

同样，对于流数据管道，通过组合订阅实时事件，可以将kafka用作极低延迟的管道; 另外，能够可靠地存储数据，也使得可以将其用于必须保证安全的核心数据的传输，或者与仅定期加载数据的离线系统或可能长时间停机以进行扩展和维护集成。它的流处理能力使它可以实时的转换数据。

更多关于kafka提供的保证、API和功能的信息，请参阅其余的[文档](http://kafka.apache.org/documentation.html)。

这里介绍一个很好用的kafka可视化工具kafkatool，下载地址：
http://www.kafkatool.com/

kafkatool连接kafka服务器后，记得要将kafkatool中topic的Message类型设置为String，否则将看到字节码。


### 创建主题
``` bash
> bin/kafka-topics.sh --bootstrap-server broker_host:port --create --topic my-topic --partitions 1 \
    --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1
```


### 修改主题
增加分区数，分区只能增加不能减少
``` bash
> bin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic my_topic_name \
    --partitions 40
```


### 删除主题
``` bash
> bin/kafka-topics.sh --bootstrap-server broker_host:port --delete --topic my_topic_name
```


[link_id_kafka-standalone]: ../../../../2018/10/24/kafka-standalone/
[link_id_kafka-cluster]: ../../../../2018/10/27/kafka-cluster/
[link_id_producer_api]: http://kafka.apache.org/documentation.html#producerapi
[link_id_consumer_api]: http://kafka.apache.org/documentation.html#consumerapi
[link_id_streams_api]: http://kafka.apache.org/documentation/streams
[link_id_connector_api]: http://kafka.apache.org/documentation.html#connect
[link_id_tcp_protocol]: https://kafka.apache.org/protocol.html

