---
title: redis 主从配置
date: 2018-08-14 14:53:19
tags: redis
categories: bigdata
---

要配置主从，我们必须安装两台`redis`：主服务器为`master`，从服务器为`slave`。安装步骤请参考我的上一篇 [redis 的安装使用][redis-install]

### 我们在一台机器上面配置主从，多台的配置是一样的，只要修改下`IP`和`PORT`即可。以我们上一篇安装好的一台`redis`为基础，它的安装路径为：`/home/hewentian/ProjectD/redis-4.0.11`
``` bash
$ cd /home/hewentian/ProjectD
$ mv redis-4.0.11 redis-4.0.11_master
$ cp -r redis-4.0.11_master/ redis-4.0.11_slave/
```

`master`服务器为：`/home/hewentian/ProjectD/redis-4.0.11_master`
`slave`服务器为：`/home/hewentian/ProjectD/redis-4.0.11_slave`

### `master`服务器使用默认端口`6379`，所以不用配置。下面我们配置`slave`服务器：
``` bash
$ cd /home/hewentian/ProjectD/redis-4.0.11_slave/
$ vi redis.conf

// 将默认端口改为6380，并加上 slaveof 127.0.0.1 6379 和 pidfile，在文件里面改动如下：
port 6380
slaveof 127.0.0.1 6379
pidfile /var/run/redis_6380.pid
```
保存文件并退出，`slave`服务器配置完成。


### 接着我们启动`master`和`slave`服务器
启动`master`服务器
``` bash
$ cd /home/hewentian/ProjectD/redis-4.0.11_master/src/
$ ./redis-server /home/hewentian/ProjectD/redis-4.0.11_master/redis.conf 
```

启动`slave`服务器
``` bash
$ cd /home/hewentian/ProjectD/redis-4.0.11_slave/src/
$ ./redis-server /home/hewentian/ProjectD/redis-4.0.11_slave/redis.conf 
```

在`master`服务器窗口中，可以看到如下信息：

	1514:M 14 Aug 15:58:33.568 * Slave 127.0.0.1:6380 asks for synchronization
	1514:M 14 Aug 15:58:33.568 * Full resync requested by slave 127.0.0.1:6380
	1514:M 14 Aug 15:58:33.568 * Starting BGSAVE for SYNC with target: disk
	1514:M 14 Aug 15:58:33.569 * Background saving started by pid 1584
	1584:C 14 Aug 15:58:33.573 * DB saved on disk
	1584:C 14 Aug 15:58:33.573 * RDB: 0 MB of memory used by copy-on-write
	1514:M 14 Aug 15:58:33.576 * Background saving terminated with success
	1514:M 14 Aug 15:58:33.576 * Synchronization with slave 127.0.0.1:6380 succeeded

在`slave`服务器窗口中，可以看到如下信息：

	1579:S 14 Aug 15:58:33.567 * Connecting to MASTER 127.0.0.1:6379
	1579:S 14 Aug 15:58:33.567 * MASTER <-> SLAVE sync started
	1579:S 14 Aug 15:58:33.568 * Non blocking connect for SYNC fired the event.
	1579:S 14 Aug 15:58:33.568 * Master replied to PING, replication can continue...
	1579:S 14 Aug 15:58:33.568 * Partial resynchronization not possible (no cached master)
	1579:S 14 Aug 15:58:33.569 * Full resync from master: 7f883c61e9326b040b150462901e70afd5c4a49c:0
	1579:S 14 Aug 15:58:33.576 * MASTER <-> SLAVE sync: receiving 176 bytes from master
	1579:S 14 Aug 15:58:33.577 * MASTER <-> SLAVE sync: Flushing old data
	1579:S 14 Aug 15:58:33.577 * MASTER <-> SLAVE sync: Loading DB in memory
	1579:S 14 Aug 15:58:33.577 * MASTER <-> SLAVE sync: Finished with success


从上面的信息中，我们可以知道，`slave`服务器已经连上`master`服务器。

### 测试同步数据
登录`master`服务器并在其中插入一个键
``` bash
$ cd /home/hewentian/ProjectD/redis-4.0.11_master/src/
$ ./redis-cli -p 6379

127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set name "Tim Ho"
OK
127.0.0.1:6379> get name
"Tim Ho"
127.0.0.1:6379>
```

登录`slave`服务器并在其中获取一个键
``` bash
$ cd /home/hewentian/ProjectD/redis-4.0.11_slave/src/
$ ./redis-cli -p 6380

127.0.0.1:6380> keys *
(empty list or set)
127.0.0.1:6380> keys *
1) "name"
127.0.0.1:6380> get name 
"Tim Ho"
127.0.0.1:6380> 
```

至此，主从配置完成。


[redis-install]: ../../../../2018/08/07/redis-install/