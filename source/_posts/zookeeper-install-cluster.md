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
$ mkdir -p /home/hewentian/zookeeperCluster/server1
$ mkdir -p /home/hewentian/zookeeperCluster/server1/data
```

### 2.将`zookeeper-3.4.6.tar.gz`放到`/home/hewentian/zookeeperCluster/server1`目录下，并执行如下脚本解压
``` bash
$ cd /home/hewentian/zookeeperCluster/server1
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
	dataDir=/home/hewentian/zookeeperCluster/server1/data
	clientPort=2181  # 这台服务器的端口为2181这里为默认值
	server.1=127.0.0.1:2888:3888
	server.2=127.0.0.1:2889:3889
	server.3=127.0.0.1:2890:3890

### 6.在`/home/hewentian/zookeeperCluster/server1/data`目录下建`myid`文件并在其中输入1，只输入1，代表server.1
``` bash
$ cd /home/hewentian/zookeeperCluster/server1/data
$ vi myid
```
这样第一台服务器已经配置完毕。


### 7.接下来我们将`server1`复制为`server2`, `server3`
``` bash
$ cd /home/hewentian/zookeeperCluster
$ cp -r server1 server2
$ cp -r server1 server3
```

### 8.将`server2/data`目录下的`myid`的内容修改为2
``` bash
$ cd /home/hewentian/zookeeperCluster/server2/data
$ vi myid
```
同理，将将`server3/data`目录下的`myid`的内容修改为3
``` bash
$ cd /home/hewentian/zookeeperCluster/server3/data
$ vi myid
```

### 9.修改`server2`的配置文件
``` bash
$ cd /home/hewentian/zookeeperCluster/server2/zookeeper-3.4.6/conf/
$ vi zoo.cfg
```
仅修改两处地方即可，要修改的地方如下：

	dataDir=/home/hewentian/zookeeperCluster/server2/data  # 这里是数据保存的位置
	clientPort=2182                                        # 这台服务器的端口为2182

同理，修改`server3`的配置文件
``` bash
$ cd /home/hewentian/zookeeperCluster/server3/zookeeper-3.4.6/conf/
$ vi zoo.cfg
```
仅修改两处地方即可，要修改的地方如下：

	dataDir=/home/hewentian/zookeeperCluster/server3/data  # 这里是数据保存的位置
	clientPort=2183                                        # 这台服务器的端口为2183


### 10.到目前为此，我们已经将3台`zookeeper`服务器都配置好了。接下来，我们要将他们都启动
启动server1
``` bash
$ cd /home/hewentian/zookeeperCluster/server1/zookeeper-3.4.6/bin/
$ ./zkServer.sh start
```
启动server2
``` bash
$ cd /home/hewentian/zookeeperCluster/server2/zookeeper-3.4.6/bin/
$ ./zkServer.sh start
```
启动server3
``` bash
$ cd /home/hewentian/zookeeperCluster/server3/zookeeper-3.4.6/bin/
$ ./zkServer.sh start
```

### 11.当三台服务器都启动好了，我们分别连到server1、server2、server3：
连接到server1
``` bash
$ cd /home/hewentian/zookeeperCluster/server1/zookeeper-3.4.6/bin/
$ ./zkCli.sh -server 127.0.0.1:2181
```
连接到server2
``` bash
$ cd /home/hewentian/zookeeperCluster/server2/zookeeper-3.4.6/bin/
$ ./zkCli.sh -server 127.0.0.1:2182
```
连接到server1
``` bash
$ cd /home/hewentian/zookeeperCluster/server3/zookeeper-3.4.6/bin/
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

集群部署结束。

### 参数说明：
1. tickTime：zookeeper中使用的基本时间单位，毫秒值；
2. initLimit：由于zookeeper集群中包含多台server，其中一台为leader，其余的为follower。initLimit参数配置初始化连接时，follower和leader之间的最长心跳时间，此时该参数设置为5，说明时间限制为5倍tickTime, 即：5 * 2000 = 10000ms = 10s；
3. syncLimit：该参数配置leader和follower之间发送消息，请求和应答的最大时间长度。此时该参数设置为2，说明时间限制为2倍tickTime，即4000ms；
4. dataDir: 数据存放目录，可以是任意目录；
5. dataLogDir: log目录，同样可以是任意目录。如果没有设置该参数，将使用和dataDir相同的设置；
6. clientPort: 监听client连接的端口号；
7. server.X=A:B:C：其中X是一个数字，表示这是第几号server。A是该server所在的IP地址；B配置该server和集群中的leader交换消息所使用的端口；C配置选举leader时所使用的端口。由于配置的是伪集群模式，所以各个server的B，C参数必须不同。
