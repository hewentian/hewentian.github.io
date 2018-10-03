---
title: redis 的安装使用
date: 2018-08-07 10:50:27
tags: redis
categories: bigdata
---

#### 要使用`redis`我们首先要安装这个软件。下面，我们说说`redis`的安装过程：

首先，我们要将`redis`安装包下载回来，截止本文写时，`redis`官网发布的最新版本为`4.0.11`。当然，我们也可以从这里下载 [redis-4.0.11.tar.gz](/download/redis-4.0.11.tar.gz) 和 [redis-4.0.11.tar.gz.md5](/download/redis-4.0.11.tar.gz.md5)。推荐从`redis`官网下载最新版本。

``` bash
$ cd /home/hewentian/ProjectD/
$ wget http://download.redis.io/releases/redis-4.0.11.tar.gz

// 验证下载文件的完整性，在下载的时候要将MD5文件或者SHA256文件也下载回来
$ md5sum -c redis-4.0.11.tar.gz.md5 
redis-4.0.11.tar.gz: OK

$ tar xzf redis-4.0.11.tar.gz
$ cd redis-4.0.11/
$ make
```

#### After building Redis, it is a good idea to test it using:
``` bash
$ make test

// 如果见到下面的结果，证明我们已经成功安装
\o/ All tests passed without errors!
```

#### 下面的内容摘自`${REDIS_HOME}/README.md`

Running Redis
-------------

To run Redis with the default configuration just type:

    $ cd src
    $ ./redis-server

If you want to provide your redis.conf, you have to run it using an additional
parameter (the path of the configuration file):

    $ cd src
    $ ./redis-server /path/to/redis.conf

It is possible to alter the Redis configuration by passing parameters directly
as options using the command line. Examples:

    $ ./redis-server --port 9999 --slaveof 127.0.0.1 6379
    $ ./redis-server /etc/redis/6379.conf --loglevel debug

All the options in redis.conf are also supported as options using the command
line, with exactly the same name.


Playing with Redis
------------------

You can use redis-cli to play with Redis. Start a redis-server instance,
then in another terminal try the following:

    $ cd src
    $ ./redis-cli	# 如果需要连接到指定的redis可以使用：-h {IP_ADDRESS}参数，如：./redis-cli -h 127.0.0.1
    redis> ping
    PONG
    redis> set foo bar
    OK
    redis> get foo
    "bar"
    redis> incr mycounter
    (integer) 1
    redis> incr mycounter
    (integer) 2
    redis>

You can find the list of all the available commands at http://redis.io/commands.


  ***注意：启动Redis服务器的时候，请务必指定它的配置文件，否则有可能会出现意想不到的情况。***

#### 为Redis设置登录密码
  为Redis设置登录密码的方法比较简单，打开`${REDIS_HOME}/redis.conf`文件，找到`requirepass foobared`这一行，如下：

		################################## SECURITY ###################################

		# Require clients to issue AUTH <PASSWORD> before processing any other
		# commands.  This might be useful in environments in which you do not trust
		# others with access to the host running redis-server.
		#
		# This should stay commented out for backward compatibility and because most
		# people do not need auth (e.g. they run their own servers).
		#
		# Warning: since Redis is pretty fast an outside user can try up to
		# 150k passwords per second against a good box. This means that you should
		# use a very strong password otherwise it will be very easy to break.
		#
		# requirepass foobared

将`requirepass foobared`的注释打开，并把foobared设置成自己的密码即可，如将其设为abc123： 

	requirepass abc123
保存文件，并重启Redis。

在使用客户端登录的时候，将使用如下命令：

	$ cd src
	$ ./redis-cli -a abc123
	
	或者使用下面的方式更安全
	$ cd src
	$ ./redis-cli
	127.0.0.1:6379> AUTH abc123
	OK

