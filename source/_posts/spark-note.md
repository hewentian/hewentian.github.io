---
title: spark 学习笔记
date: 2020-01-14 19:57:24
tags: spark
categories: bigdata
---

为便于学习spark，这里安装standalone模式，在下面三台机器搭建一个简单的集群。我们将slave3作为spark的master节点，而slave1、slave2作为从节点。

    slave1:
        ip: 192.168.56.111
        hostname: hadoop-host-slave-1
    slave2:
        ip: 192.168.56.112
        hostname: hadoop-host-slave-2
    slave3:
        ip: 192.168.56.113
        hostname: hadoop-host-slave-3

安装过程参考官方文档：
http://spark.apache.org/docs/latest/spark-standalone.html

目前spark的稳定版本为2.4.4，可以在下面的地址找到然后下载。
http://spark.apache.org/downloads.html
https://www.apache.org/dyn/closer.lua/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz

在slave3中启动spark作为master
``` bash
$ tar xf spark-2.4.4-bin-hadoop2.7.tgz
$ cd spark-2.4.4-bin-hadoop2.7/
$ ./sbin/start-master.sh
```

启动后，在启动日志中可以看到部分输出，如下：

        20/01/14 16:54:24 INFO Master: Starting Spark master at spark://hadoop-host-slave-3:7077
        20/01/14 16:54:24 INFO Master: Running Spark version 2.4.4
        20/01/14 16:54:25 INFO Utils: Successfully started service 'MasterUI' on port 8080.
        20/01/14 16:54:25 INFO MasterWebUI: Bound MasterWebUI to 0.0.0.0, and started at http://hadoop-host-slave-3:8080
        20/01/14 16:54:25 INFO Master: I have been elected leader! New state: ALIVE


接着分别在slave1、slave2中启动spark作为从节点
``` bash
$ tar xf spark-2.4.4-bin-hadoop2.7.tgz
$ cd spark-2.4.4-bin-hadoop2.7/
$ ./sbin/start-slave.sh spark://hadoop-host-slave-3:7077
```

在浏览器中可以通过访问如下地址，看到整个集群的情况
http://hadoop-host-slave-3:8080/


可以通过交互式命令连接到集群

        ./bin/spark-shell --master spark://IP:PORT

也可以将程序打包成jar包，然后在主节点上面提交给集群，如下

        ./bin/spark-submit --class com.hewentian.spark.SparkPi --master spark://hadoop-host-slave-3:7077 /home/hadoop/spark-1.0-SNAPSHOT.jar


### 读取mysql数据
``` bash
$ cd /home/hadoop/spark-2.4.4-bin-hadoop2.7/
$ ./bin/spark-shell --jars /home/hadoop/spark-2.4.4-bin-hadoop2.7/jars/mysql-connector-java-5.1.25.jar

scala> val sqlContext = spark.sqlContext
scala> val df = sqlContext.read.format("jdbc").option("url", "jdbc:mysql://mysql.hewentian.com:3306/bfg_db?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull").option("driver", "com.mysql.jdbc.Driver").option("user", "bfg_db").option("password", "iE1zNB?A91*YbQ9hK").option("dbtable", "student").load()
```

未完待续……


