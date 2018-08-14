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

#### 下面的内容摘自`${REDIS_PATH}/README.md`

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

    % cd src
    % ./redis-cli
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
