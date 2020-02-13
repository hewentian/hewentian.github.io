---
title: spark 学习笔记
date: 2020-01-14 19:57:24
tags: spark
categories: bigdata
---

### standalone模式
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


我们可以通过配置，在一台机器一键启动所有节点的spark进程。分别在三个节点执行如下操作：
``` bash
$ cd spark-2.4.4-bin-hadoop2.7/conf
$ cp slaves.template slaves
$ vi slaves

去掉文件中原来的localhost配置，并加入下面两行
hadoop-host-slave-1
hadoop-host-slave-2
```

这样在slave3节点上就可以通过如下命令，一次将master、slave进程启动
``` bash
$ cd spark-2.4.4-bin-hadoop2.7/
$ ./sbin/start-all.sh
```


### 读取mysql数据
``` bash
$ cd /home/hadoop/spark-2.4.4-bin-hadoop2.7/
$ ./bin/spark-shell --jars /home/hadoop/spark-2.4.4-bin-hadoop2.7/jars/mysql-connector-java-5.1.25.jar

scala> val sqlContext = spark.sqlContext
scala> val df = sqlContext.read.format("jdbc").option("url", "jdbc:mysql://mysql.hewentian.com:3306/bfg_db?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull").option("driver", "com.mysql.jdbc.Driver").option("user", "bfg_db").option("password", "iE1zNB?A91*YbQ9hK").option("dbtable", "student").load()
```


### yarn模式
Spark支持将作业提交到yarn上运行，此时不需要启动master节点，也不需要启动worker节点。

同样需在slaves文件下配置worker节点的机器，像standalone模式那样。

只需在spark中指定hadoop的配置文件目录即可，在所有spark节点中执行如下操作：
``` bash
$ cd spark-2.4.4-bin-hadoop2.7/conf
$ cp spark-env.sh.template spark-env.sh
$ vi spark-env.sh

配置这行即可
HADOOP_CONF_DIR=/home/hadoop/hadoop-2.7.3/etc/hadoop
```

要保证hdfs和yarn已经启动。

以交互方式连接到yarn模式的spark
        ./bin/spark-shell --master yarn --deploy-mode client

也可以将程序打包成jar包，然后在主节点上面提交给集群，如下
        ./bin/spark-submit --class path.to.your.Class --master yarn --deploy-mode cluster [options] <app jar> [app options]

例如：
        ./bin/spark-submit \
        --class org.apache.spark.examples.SparkPi \
        --master yarn \
        --deploy-mode cluster \
        --driver-memory 4g \
        --executor-memory 2g \
        --executor-cores 1 \
        --queue thequeue \
        examples/jars/spark-examples*.jar \
        10


如果还想将此yarn模式配置成基于zookeeper的高可用，如将主机`hadoop-host-slave-2`配置成备用的master。则需如下配置：
分别在所有spark节点中执行如下操作
``` bash
$ cd spark-2.4.4-bin-hadoop2.7/conf
$ vi spark-env.sh

配置zookeeper地址
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=hadoop-host-master:2181,hadoop-host-slave-1:2181,hadoop-host-slave-2:2181 -Dspark.deploy.zookeeper.dir=/spark"
```

启动顺序：
1. 启动的zookeeper集群；
2. 启动的hdfs和yarn；
3. 在主节点`hadoop-host-slave-3`执行如下脚本，启动所有服务`./sbin/start-all.sh`；
4. 在节点`hadoop-host-slave-2`执行如下脚本，启动它作为备份主节点`./sbin/start-master.sh`。

通过浏览器，观看集群状态：
http://hadoop-host-slave-2:8080/
http://hadoop-host-slave-3:8080/


未完待续……


