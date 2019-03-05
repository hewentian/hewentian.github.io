---
title: zookeeper 集群版安装方法
date: 2017-12-06 16:33:35
tags: zookeeper
categories: bigdata
---

### 计划在一台`Ubuntu Linux`服务器上部署3台`zookeeper`服务器，分别为`server1`, `server2`, `server3`
因为三台`zookeeper`服务器的配置都差不多，所以我们先设置好一台`server1`的配置，再将其复制成`server2`, `server3`并修改其中的配置即可。

### 1.建目录，如下：
``` bash
$ mkdir -p /home/hewentian/ProjectD/zookeeperCluster/server1
$ mkdir -p /home/hewentian/ProjectD/zookeeperCluster/server1/data
```

### 2.将`zookeeper-3.4.6.tar.gz`放到`/home/hewentian/ProjectD/zookeeperCluster/server1`目录下，并执行如下脚本解压
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server1
$ tar xzvf zookeeper-3.4.6.tar.gz
```

### 3.解压后得到`zookeeper-3.4.6`文件夹，进入`conf`目录
``` bash
$ cd zookeeper-3.4.6/conf/
```

### 4.执行如下命令，新建一个配置文件zoo.cfg
``` bash
$ cp zoo_sample.cfg zoo.cfg
```

### 5.修改zoo.cfg并在其中修改如下内容：
	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/home/hewentian/ProjectD/zookeeperCluster/server1/data # 这里必须为绝对路径，否则有可能无法启动
	clientPort=2181                 # 这台服务器的端口为2181（即默认值），当部署为真集群的时候各个节点的端口都是2181
	server.1=127.0.0.1:2888:3888    # 当部署成真集群的时候：
	server.2=127.0.0.1:2889:3889    # server.1、server.2和server.3后面的IP要修改成各个节点的真实IP或者域名
	server.3=127.0.0.1:2890:3890    # 并且server.2、server.3后面的两个端口要和server.1的是一样的，都是2888:3888

### 6.在`/home/hewentian/ProjectD/zookeeperCluster/server1/data`目录下建`myid`文件并在其中输入1，只输入1，代表server.1
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server1/data
$ vi myid
```
这样第一台服务器已经配置完毕。


### 7.接下来我们将`server1`复制为`server2`, `server3`
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster
$ cp -r server1 server2
$ cp -r server1 server3
```

### 8.将`server2/data`目录下的`myid`的内容修改为2
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server2/data
$ vi myid
```
同理，将将`server3/data`目录下的`myid`的内容修改为3
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server3/data
$ vi myid
```

### 9.修改`server2`的配置文件
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server2/zookeeper-3.4.6/conf/
$ vi zoo.cfg
```
仅修改两处地方即可，要修改的地方如下：

	dataDir=/home/hewentian/ProjectD/zookeeperCluster/server2/data  # 这里是数据保存的位置
	clientPort=2182                                                 # 这台服务器的端口为2182

同理，修改`server3`的配置文件
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server3/zookeeper-3.4.6/conf/
$ vi zoo.cfg
```
仅修改两处地方即可，要修改的地方如下：

	dataDir=/home/hewentian/ProjectD/zookeeperCluster/server3/data  # 这里是数据保存的位置
	clientPort=2183                                                 # 这台服务器的端口为2183


### 10.到目前为此，我们已经将3台`zookeeper`服务器都配置好了。接下来，我们要将他们都启动
启动server1
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server1/zookeeper-3.4.6/bin/
$ ./zkServer.sh start
```
启动server2
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server2/zookeeper-3.4.6/bin/
$ ./zkServer.sh start
```
启动server3
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server3/zookeeper-3.4.6/bin/
$ ./zkServer.sh start
```

### 11.当三台服务器都启动好了，我们分别连到server1、server2、server3：
连接到server1
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server1/zookeeper-3.4.6/bin/
$ ./zkCli.sh -server 127.0.0.1:2181
```
连接到server2
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server2/zookeeper-3.4.6/bin/
$ ./zkCli.sh -server 127.0.0.1:2182
```
连接到server1
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server3/zookeeper-3.4.6/bin/
$ ./zkCli.sh -server 127.0.0.1:2183
```

这样你在`server1`中所作的修改，都会同步到`server2`, `server3`。
例如你在server1中
``` bash
$ create /zk_test_cluster my_data_cluster
```
你在server2, server3的客户端用
``` bash
$ ls /
```
都会看到节点zk_test_cluster


### 12. 查看运行状态
``` bash
$ cd /home/hewentian/ProjectD/zookeeperCluster/server2/zookeeper-3.4.6/
$ ./bin/zkServer.sh status
JMX enabled by default
Using config: /home/hewentian/ProjectD/zookeeperCluster/server2/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: leader

$ cd /home/hewentian/ProjectD/zookeeperCluster/server3/zookeeper-3.4.6/
$ ./bin/zkServer.sh status
JMX enabled by default
Using config: /home/hewentian/ProjectD/zookeeperCluster/server3/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
```

可以看到server2是leader，其他两台是follower。

集群部署结束。


### zookeeper三种角式
leader：负责进行投票的发起和决议，更新系统状态；
follower：负责接受客户端的请求并向客户端返回结果，在选主过程中参与投票；
observer：接受客户端的连接，将写请求转发给leader，但不参与投票，只同步leader的状态，observer的目的是为了扩展系统，提高读取速度。follower和observer统称为leaner。
client：客户端，请求发起方。

配置observer比较简单，只要在`zoo.cfg`中将要配置成为observer的机器加个后缀即可，如下：

    server.3=127.0.0.1:2890:3890:observer

### zookeeper的节点类型
zookeeper有4种节点类型：
1. PERSISTENT                持久化节点
2. PERSISTENT_SEQUENTIAL     顺序自动编号持久化节点，这种节点会根据当前已存在的节点数自动加 1
3. EPHEMERAL                 临时节点， 客户端session超时这类节点就会被自动删除
4. EPHEMERAL_SEQUENTIAL      临时自动编号节点

** 注意：EPHEMERAL类型的节点不能有子节点 **


### 参数说明：
1. tickTime：zookeeper中使用的基本时间单位，毫秒值；
2. initLimit：由于zookeeper集群中包含多台server，其中一台为leader，其余的为follower。initLimit参数配置初始化连接时，follower和leader之间的最长心跳时间，此时该参数设置为5，说明时间限制为5倍tickTime, 即：5 * 2000 = 10000ms = 10s；
3. syncLimit：该参数配置leader和follower之间发送消息，请求和应答的最大时间长度。此时该参数设置为2，说明时间限制为2倍tickTime，即4000ms；
4. dataDir: 数据存放目录，可以是任意目录；
5. dataLogDir: log目录，同样可以是任意目录。如果没有设置该参数，将使用和dataDir相同的设置；
6. clientPort: 监听client连接的端口号；
7. server.X=A:B:C：其中X是一个数字，表示这是第几号server。A是该server所在的IP地址；B配置该server和集群中的leader交换消息所使用的端口；C配置选举leader时所使用的端口。由于配置的是伪集群模式，所以各个server的B，C参数必须不同。


### zookeeper能帮我们做什么
1. Hadoop使用zookeeper的事件处理确保整个集群只有一个NameNode存储配置信息等；
2. HBase使用zookeeper的事件处理确保整个集群只有一个HMaster，察觉HRegionServer联机和宕机、存储访问控制列表等。


### 一些要注意的
1. 注册的Watcher消息只会通知一次；
2. 节点需要奇数个的原因有二：（1）容错和偶数是一样的，所以没必要多台；（2）防止脑裂；
3. zookeeper两种模式：恢复模式（选主）和广播模式（Zab原子广播更新数据）；
4. zookeeper的4个特点：最终一致性、可靠性、原子性、顺序性（因为增删改都发给Leader）；
5. zookeeper的数据存放在内存中，但是会定期flush到磁盘dataDir的目录中。


通过java api使用zookeeper的例子可以参见这里： [ZookeeperUtil.java][link_id_ZookeeperUtil]、[zookeeper实现分布式锁、rmi高可用][link_id_zookeeper]

[link_id_ZookeeperUtil]: https://github.com/hewentian/hadoop-demo/blob/master/src/main/java/com/hewentian/hadoop/utils/ZookeeperUtil.java
[link_id_zookeeper]: https://github.com/hewentian/hadoop-demo/tree/master/src/main/java/com/hewentian/hadoop/zookeeper
