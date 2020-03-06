---
title: zookeeper 单机版安装方法
date: 2017-12-06 16:33:23
tags: zookeeper
categories: bigdata
---

### 下面演示在 Ubuntu 16.04 LTS 下面安装 zookeeper
将`zookeeper-3.4.6.tar.gz`放到一个目录下，如`/home/hewentian/ProjectD`
执行如下脚本解压
``` bash
$ cd /home/hewentian/ProjectD
$ tar xzvf zookeeper-3.4.6.tar.gz
```
解压后得到zookeeper-3.4.6文件夹，进入conf目录
``` bash
$ cd zookeeper-3.4.6/conf/
```
执行如下命令，新建一个配置文件zoo.cfg
``` bash
$ cp zoo_sample.cfg zoo.cfg
```
修改zoo.cfg并在其中修改如下内容：

	tickTime=2000
	dataDir=/home/hewentian/ProjectD/zookeeper-3.4.6/data
	clientPort=2181

解压后的`zookeeper-3.4.6`目录下是没有`data`这个文件夹的，要执行如下命令创建
``` bash
$ cd /home/hewentian/ProjectD/zookeeper-3.4.6/
$ mkdir data
```

修改好后，我们就可以进入bin目录下启动zookeeper了
``` bash
$ cd /home/hewentian/ProjectD/zookeeper-3.4.6/bin
$ ./zkServer.sh start
```
启动后，我们可以连到zookeeper服务器，命令如下：
``` bash
$ cd /home/hewentian/ProjectD/zookeeper-3.4.6/bin
$ ./zkCli.sh -server 127.0.0.1:2181
```

### 一些操作例子，如下：
From here, you can try a few simple commands to get a feel for this simple command line interface. First, start by issuing the list command, as in ls, yielding:
``` bash
[zkshell: 8] ls /
[zookeeper]
```

Next, create a new znode by running create /zk_test my_data. This creates a new znode and associates the string "my_data" with the node. You should see:
``` bash
[zkshell: 9] create /zk_test my_data
Created /zk_test
```

Issue another ls / command to see what the directory looks like:
``` bash
[zkshell: 11] ls /
[zookeeper, zk_test]
```
        
Notice that the zk_test directory has now been created.
Next, verify that the data was associated with the znode by running the get command, as in:
``` bash
[zkshell: 12] get /zk_test
my_data
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 5
mtime = Fri Jun 05 13:57:06 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0
dataLength = 7
numChildren = 0
```

We can change the data associated with zk_test by issuing the set command, as in:
``` bash
[zkshell: 14] set /zk_test junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
[zkshell: 15] get /zk_test
junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
```
(Notice we did a get after setting the data and it did, indeed, change.

Finally, let's delete the node by issuing:
``` bash
[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
```
That's it for now. To explore more, continue with the rest of this document and see the Programmer's Guide.
