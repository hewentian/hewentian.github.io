---
title: redis 高可用配置
date: 2018-08-14 14:53:19
tags: redis
categories: bigdata
---

## 主从配置
要配置主从，我们必须安装两台`redis`：主服务器为`master`，从服务器为`slave`。安装步骤请参考我的上一篇 [redis 的安装使用][redis-standalone]

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


## 哨兵配置
下面演示在`docker`里配置`一主两从三哨兵`的redis哨兵模式。

#### 先部署一主两从
先在宿主机建立保存数据的目录和设置好配置文件：
``` bash
$ mkdir -p /root/db/redis-master
$ cd /root/db/redis-master
$ mkdir data conf
$ cd conf
$ vi redis.conf

bind 192.168.56.113    # 这里要指定IP地址。如果用 127.0.0.1，则 sentinel 将获取到 127.0.0.1 ，这样将导致客户端连接到它本机了
port 6379
requirepass abc123
masterauth abc123

appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
```

将配置好的文件复制两份作为从库来修改
``` bash
$ cd /root/db
$ cp -r redis-master redis-replica-1
$ cp -r redis-master redis-replica-2
$
$ vi redis-replica-1/conf/redis.conf

bind 192.168.56.113
port 6380
requirepass abc123
masterauth abc123
replicaof 192.168.56.113 6379

appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes


$ vi redis-replica-2/conf/redis.conf

bind 192.168.56.113
port 6381
requirepass abc123
masterauth abc123
replicaof 192.168.56.113 6379

appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
```

启动一主两从
``` bash
$ sudo docker pull redis:7.0.2
$
$ sudo docker run \
    -itd --name redis-master \
    --net=host \
    -v /root/db/redis-master/conf/redis.conf:/etc/redis/redis.conf \
    -v /root/db/redis-master/data:/data \
    redis:7.0.2 redis-server /etc/redis/redis.conf

$ sudo docker run \
    -itd --name redis-replica-1 \
    --net=host \
    -v /root/db/redis-replica-1/conf/redis.conf:/etc/redis/redis.conf \
    -v /root/db/redis-replica-1/data:/data \
    redis:7.0.2 redis-server /etc/redis/redis.conf

$ sudo docker run \
    -itd --name redis-replica-2 \
    --net=host \
    -v /root/db/redis-replica-2/conf/redis.conf:/etc/redis/redis.conf \
    -v /root/db/redis-replica-2/data:/data \
    redis:7.0.2 redis-server /etc/redis/redis.conf
```


查看连接情况：
``` bash
$ redis-cli -h redis.hewentian.com -p 6379 -a abc123 -n 0 --raw
$ redis.hewentian.com:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.56.113,port=6380,state=online,offset=98,lag=1
slave1:ip=192.168.56.113,port=6381,state=online,offset=98,lag=0
master_failover_state:no-failover
master_replid:2539e5441c16b58f2b934274db472690a9dd9139
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:98
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:98
redis.hewentian.com:6379>


$ redis-cli -h redis.hewentian.com -p 6380 -a abc123 -n 0 --raw
redis.hewentian.com:6380> info replication
# Replication
role:slave
master_host:192.168.56.113
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_read_repl_offset:532
slave_repl_offset:532
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:2539e5441c16b58f2b934274db472690a9dd9139
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:532
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:518
redis.hewentian.com:6380>


$ redis-cli -h redis.hewentian.com -p 6381 -a abc123 -n 0 --raw
redis.hewentian.com:6381> info replication
# Replication
role:slave
master_host:192.168.56.113
master_port:6379
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_read_repl_offset:588
slave_repl_offset:588
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:2539e5441c16b58f2b934274db472690a9dd9139
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:588
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:43
repl_backlog_histlen:546
redis.hewentian.com:6381>
```


由上面的信息可知，一主两从部署成功。下面开始部署三哨兵。
``` bash
$ cd /root/db
$ vi sentinel-1.conf

port 26379
requirepass abc123
dir /tmp
sentinel monitor mymaster 192.168.56.113 6379 2
sentinel auth-pass mymaster abc123
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes


$ vi sentinel-2.conf

port 26380
requirepass abc123
dir /tmp
sentinel monitor mymaster 192.168.56.113 6379 2
sentinel auth-pass mymaster abc123
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes


$ vi sentinel-3.conf

port 26381
requirepass abc123
dir /tmp
sentinel monitor mymaster 192.168.56.113 6379 2
sentinel auth-pass mymaster abc123
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```

启动三哨兵
``` bash
$ sudo docker run \
    -itd --name redis-sentinel-1 \
    --net=host \
    -v /root/db/sentinel-1.conf:/etc/redis/sentinel.conf \
    redis:7.0.2 redis-sentinel /etc/redis/sentinel.conf

$ sudo docker run \
    -itd --name redis-sentinel-2 \
    --net=host \
    -v /root/db/sentinel-2.conf:/etc/redis/sentinel.conf \
    redis:7.0.2 redis-sentinel /etc/redis/sentinel.conf

$ sudo docker run \
    -itd --name redis-sentinel-3 \
    --net=host \
    -v /root/db/sentinel-3.conf:/etc/redis/sentinel.conf \
    redis:7.0.2 redis-sentinel /etc/redis/sentinel.conf
```


查看连接情况：
``` bash
$ redis-cli -h redis.hewentian.com -p 26379 -a abc123 --raw
redis.hewentian.com:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.56.113:6379,slaves=2,sentinels=3
redis.hewentian.com:26379>


$ redis-cli -h redis.hewentian.com -p 26380 -a abc123 --raw
redis.hewentian.com:26380> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.56.113:6379,slaves=2,sentinels=3
redis.hewentian.com:26380>


$ redis-cli -h redis.hewentian.com -p 26381 -a abc123 --raw
redis.hewentian.com:26381> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.56.113:6379,slaves=2,sentinels=3
redis.hewentian.com:26381>
```


其中一个哨兵的启动日志：
``` bash
$ sudo docker logs -f redis-sentinel-1
1:X 15 Dec 2021 00:21:50.548 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:X 15 Dec 2021 00:21:50.549 # Redis version=7.0.2, bits=64, commit=00000000, modified=0, pid=1, just started
1:X 15 Dec 2021 00:21:50.549 # Configuration loaded
1:X 15 Dec 2021 00:21:50.550 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 7.0.2 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

1:X 15 Dec 2021 00:21:50.551 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:X 15 Dec 2021 00:21:50.561 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:21:50.561 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:21:50.561 # Sentinel ID is 4d87c515992607402e2d787d768798aa40708ccf
1:X 15 Dec 2021 00:21:50.561 # +monitor master mymaster 192.168.56.113 6379 quorum 2
1:X 15 Dec 2021 00:21:50.561 * +slave slave 192.168.56.113:6380 192.168.56.113 6380 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:21:50.563 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:21:50.563 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:21:50.563 * +slave slave 192.168.56.113:6381 192.168.56.113 6381 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:21:50.565 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:21:50.565 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:22:06.426 * +sentinel sentinel 6c0a9d267de6b16a3f7e6ff7bd16c10c84299c5b 192.168.56.113 26380 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:22:06.432 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:22:06.433 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:22:18.257 * +sentinel sentinel 53c09ff57b5927ca5ccf4d76216bcb926c38ad1e 192.168.56.113 26381 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:22:18.267 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:22:18.267 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
```


由上面的信息可知，三哨兵部署成功。下面将`redis-master`停掉，模拟宕机的情况。
``` bash
$ sudo docker stop redis-master
```

查看`redis-sentinel-1`的日志输出如下：
``` bash
1:X 15 Dec 2021 00:42:14.088 # +sdown master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:14.173 # +odown master mymaster 192.168.56.113 6379 #quorum 2/2
1:X 15 Dec 2021 00:42:14.173 # +new-epoch 1
1:X 15 Dec 2021 00:42:14.173 # +try-failover master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:14.182 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:42:14.182 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:42:14.183 # +vote-for-leader 6c0a9d267de6b16a3f7e6ff7bd16c10c84299c5b 1
1:X 15 Dec 2021 00:42:14.208 # 4d87c515992607402e2d787d768798aa40708ccf voted for 6c0a9d267de6b16a3f7e6ff7bd16c10c84299c5b 1
1:X 15 Dec 2021 00:42:14.209 # 53c09ff57b5927ca5ccf4d76216bcb926c38ad1e voted for 6c0a9d267de6b16a3f7e6ff7bd16c10c84299c5b 1
1:X 15 Dec 2021 00:42:14.242 # +elected-leader master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:14.242 # +failover-state-select-slave master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:14.308 # +selected-slave slave 192.168.56.113:6380 192.168.56.113 6380 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:14.308 * +failover-state-send-slaveof-noone slave 192.168.56.113:6380 192.168.56.113 6380 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:14.364 * +failover-state-wait-promotion slave 192.168.56.113:6380 192.168.56.113 6380 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:15.022 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:42:15.023 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:42:15.023 # +promoted-slave slave 192.168.56.113:6380 192.168.56.113 6380 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:15.023 # +failover-state-reconf-slaves master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:15.104 * +slave-reconf-sent slave 192.168.56.113:6381 192.168.56.113 6381 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:15.285 # -odown master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:16.063 * +slave-reconf-inprog slave 192.168.56.113:6381 192.168.56.113 6381 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:16.064 * +slave-reconf-done slave 192.168.56.113:6381 192.168.56.113 6381 @ mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:16.129 # +failover-end master mymaster 192.168.56.113 6379
1:X 15 Dec 2021 00:42:16.129 # +switch-master mymaster 192.168.56.113 6379 192.168.56.113 6380
1:X 15 Dec 2021 00:42:16.130 * +slave slave 192.168.56.113:6381 192.168.56.113 6381 @ mymaster 192.168.56.113 6380
1:X 15 Dec 2021 00:42:16.131 * +slave slave 192.168.56.113:6379 192.168.56.113 6379 @ mymaster 192.168.56.113 6380
1:X 15 Dec 2021 00:42:16.135 # Could not rename tmp config file (Device or resource busy)
1:X 15 Dec 2021 00:42:16.136 # WARNING: Sentinel was not able to save the new configuration on disk!!!: Device or resource busy
1:X 15 Dec 2021 00:42:46.200 # +sdown slave 192.168.56.113:6379 192.168.56.113 6379 @ mymaster 192.168.56.113 6380
```

由日志可知，端口为`6380`的redis已经被选举为master节点，我们看下`6380`、`6381`的情况：
``` bash
redis.hewentian.com:6380> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.56.113,port=6381,state=online,offset=262678,lag=1
master_failover_state:no-failover
master_replid:dc5c403d30736f872625b3669a7a721234cfe92f
master_replid2:2539e5441c16b58f2b934274db472690a9dd9139
master_repl_offset:262678
second_repl_offset:237869
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:262664
redis.hewentian.com:6380>


redis.hewentian.com:6381> info replication
# Replication
role:slave
master_host:192.168.56.113
master_port:6380
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:264421
slave_repl_offset:264421
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:dc5c403d30736f872625b3669a7a721234cfe92f
master_replid2:2539e5441c16b58f2b934274db472690a9dd9139
master_repl_offset:264421
second_repl_offset:237869
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:43
repl_backlog_histlen:264379
redis.hewentian.com:6381>
```


最后将`redis-master`重新启动。
``` bash
$ sudo docker start redis-master
```

查看`redis-sentinel-1`的日志输出如下：
``` bash
1:X 15 Dec 2021 00:51:52.101 # -sdown slave 192.168.56.113:6379 192.168.56.113 6379 @ mymaster 192.168.56.113 6380
1:X 15 Dec 2021 00:52:02.065 * +convert-to-slave slave 192.168.56.113:6379 192.168.56.113 6379 @ mymaster 192.168.56.113 6380
```

由日志可知，端口为`6379`的redis已经变成slave节点，我们看下`6379`、`6380`、`6381`的情况：
``` bash
redis.hewentian.com:6379> info replication
# Replication
role:slave
master_host:192.168.56.113
master_port:6380
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:363284
slave_repl_offset:363284
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:dc5c403d30736f872625b3669a7a721234cfe92f
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:363284
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:354813
repl_backlog_histlen:8472
redis.hewentian.com:6379>


redis.hewentian.com:6380> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.56.113,port=6381,state=online,offset=367435,lag=1
slave1:ip=192.168.56.113,port=6379,state=online,offset=367568,lag=0
master_failover_state:no-failover
master_replid:dc5c403d30736f872625b3669a7a721234cfe92f
master_replid2:2539e5441c16b58f2b934274db472690a9dd9139
master_repl_offset:367568
second_repl_offset:237869
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:367554
redis.hewentian.com:6380>


redis.hewentian.com:6381> info replication
# Replication
role:slave
master_host:192.168.56.113
master_port:6380
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:408147
slave_repl_offset:408147
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:dc5c403d30736f872625b3669a7a721234cfe92f
master_replid2:2539e5441c16b58f2b934274db472690a9dd9139
master_repl_offset:408147
second_repl_offset:237869
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:43
repl_backlog_histlen:408105
redis.hewentian.com:6381>
```


[redis-standalone]: ../../../../2018/08/07/redis-standalone/

