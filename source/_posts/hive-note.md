---
title: hive 学习笔记
date: 2019-01-10 22:30:12
tags: hive
categories: bigdata
---

### Hive 介绍
hive的[官方文档][link_id_hive-home]中有对hive的详细介绍，这里不再赘述。我们用一句话描述如下：
The Apache Hive ™ data warehouse software facilitates reading, writing, and managing large datasets residing in distributed storage using SQL. Structure can be projected onto data already in storage. A command line tool and JDBC driver are provided to connect users to Hive.


### hive 的安装
安装过程参考这里：
https://cwiki.apache.org/confluence/display/Hive/GettingStarted

hive依赖于HADOOP，我们在上一篇[hadoop 集群的搭建HA][link_id_hadoop-cluster-ha]的基础上安装hive。

首先下载hive，我们下载的时候，要选择适合我们HADOOP版本的hive，我们下载的稳定版为[apache-hive-1.2.2-bin.tar.gz][link_id_hive-1-2-2]，我们将在HADOOP集群的namenode上面安装，即在master机器上面安装。将压缩包传到`/home/hadoop/`目录下。
``` bash
$ cd /home/hadoop/
$ tar xzvf apache-hive-1.2.2-bin.tar.gz
```

解压后得到目录`apache-hive-1.2.2-bin`，我们看下压缩包中的内容：
``` bash
$ cd /home/hadoop/apache-hive-1.2.2-bin
$ ls
bin  conf  examples  hcatalog  lib  LICENSE  NOTICE  README.txt  RELEASE_NOTES.txt  scripts
$
$ ls conf/
beeline-log4j.properties.template  hive-env.sh.template                 hive-log4j.properties.template
hive-default.xml.template          hive-exec-log4j.properties.template  ivysettings.xml
```

配置HADOOP_HOME：
``` bash
$ cd /home/hadoop/apache-hive-1.2.2-bin/conf/
$ cp hive-default.xml.template hive-site.xml
$ cp hive-env.sh.template hive-env.sh
$ vi hive-env.sh

HADOOP_HOME=/home/hadoop/hadoop-2.7.3
```

到这里，hive就配置好了，可以运行了。但，不妨看下下面的`配置hive元数据的存储位置`，因为生产环境可能会这样用。我们这次的安装将不会执行这个步骤。

配置hive元数据的存储位置（可选配置）
hive默认将元数据存储在`derby`数据库中（hive安装包自带），当然我们也可以选择存储在其他数据库，如mysql中。下面演示一下：
首先在MYSQL数据库中创建一个数据库，用于存储hive的元数据，我们就将库名创建为hive：
``` mysql
mysql> create database hive; 
```

然后配置hive使用mysql存储元数据：
``` bash
$ cd /home/hadoop/apache-hive-1.2.2-bin/conf/
$ vi hive-site.xml
```

修改下面部分，假定我们的数据库地址、用户名和密码如下：
``` xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.1.188:3306/hive</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
    <description>Username to use against metastore database</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>root</value>
    <description>password to use against metastore database</description>
</property>
```

最后，将mysql连接JDBC的jar包[mysql-connector-java-5.1.42.jar][link_id_mysql-connector-java]放到`apache-hive-1.2.2-bin/lib`目录下

好了，以上这部分是**可选配置**部分。


### 启动hive
初次启动hive，需在HDFS中创建几个目录，用于存储hive的数据，我们在安装hive的master节点执行如下命令：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./bin/hdfs dfs -mkdir /tmp
$ ./bin/hdfs dfs -mkdir -p /user/hive/warehouse
$
$ ./bin/hdfs dfs -chmod g+w /tmp
$ ./bin/hdfs dfs -chmod g+w /user/hive/warehouse
```

正式启动hive
``` bash
$ cd /home/hadoop/apache-hive-1.2.2-bin/bin
$ ./hive
```

启动的时候可能会报如下错误：

    Exception in thread "main" java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
    	at org.apache.hadoop.fs.Path.initialize(Path.java:205)
    	at org.apache.hadoop.fs.Path.<init>(Path.java:171)
    	at org.apache.hadoop.hive.ql.session.SessionState.createSessionDirs(SessionState.java:659)
    	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:582)
    	at org.apache.hadoop.hive.ql.session.SessionState.beginStart(SessionState.java:549)
    	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:750)
    	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:686)
    	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    	at java.lang.reflect.Method.invoke(Method.java:498)
    	at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
    	at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
    Caused by: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
    	at java.net.URI.checkPath(URI.java:1823)
    	at java.net.URI.<init>(URI.java:745)
    	at org.apache.hadoop.fs.Path.initialize(Path.java:202)
    	... 12 more

解决方法如下，先建目录：
``` bash
$ cd /home/hadoop/apache-hive-1.2.2-bin/
$ mkdir iotmp
```

将`hive-site.xml`中
1. 包含`${system:java.io.tmpdir}`的配置项替换为上面的路径`/home/hadoop/apache-hive-1.2.2-bin/iotmp`，一共有4处；
2. 包含`${system:user.name}`的配置项替换为`hadoop`。
修改项如下：
``` xml
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>/home/hadoop/apache-hive-1.2.2-bin/iotmp/hadoop</value>
    <description>Local scratch space for Hive jobs</description>
</property>
<property>
    <name>hive.downloaded.resources.dir</name>
    <value>/home/hadoop/apache-hive-1.2.2-bin/iotmp/${hive.session.id}_resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
<property>
    <name>hive.querylog.location</name>
    <value>/home/hadoop/apache-hive-1.2.2-bin/iotmp/hadoop</value>
    <description>Location of Hive run time structured log file</description>
</property>
<property>
    <name>hive.server2.logging.operation.log.location</name>
    <value>/home/hadoop/apache-hive-1.2.2-bin/iotmp/hadoop/operation_logs</value>
    <description>Top level directory where operation logs are stored if logging functionality is enabled</description>
</property>
```

重新启动hive：
``` bash
$ cd /home/hadoop/apache-hive-1.2.2-bin/bin
$ ./hive

Logging initialized using configuration in jar:file:/home/hadoop/apache-hive-1.2.2-bin/lib/hive-common-1.2.2.jar!/hive-log4j.properties
hive> show databases;
OK
default
Time taken: 0.821 seconds, Fetched: 1 row(s)
hive> 
    > use default;
OK
Time taken: 0.043 seconds
hive> 
    > show tables;
OK
Time taken: 0.094 seconds
hive> 
```

至此，hive安装成功。从上面可知，hive有一个默认的数据库`default`，并且里面一张表也没有。


### hive初体验
- A longer tutorial that covers more features of HiveQL:
https://cwiki.apache.org/confluence/display/Hive/Tutorial

- The HiveQL Language Manual:
https://cwiki.apache.org/confluence/display/Hive/LanguageManual

### 创建数据库：
``` bash
hive> CREATE DATABASE IF NOT EXISTS tim;
OK
Time taken: 0.323 seconds
hive>
    > show databases;
OK
default
tim
Time taken: 0.025 seconds, Fetched: 2 row(s)
```

同样，我们可以在HDFS中查看到：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./bin/hdfs dfs -ls /user/hive/warehouse
Found 1 items
drwxrwxr-x   - hadoop supergroup          0 2019-01-01 19:32 /user/hive/warehouse/tim.db
```

### 创建表
``` bash
hive> use tim;
OK
Time taken: 0.042 seconds
hive> 
    > CREATE TABLE IF NOT EXISTS t_user (
    > id INT,
    > name STRING COMMENT 'user name',
    > age INT COMMENT 'user age',
    > sex STRING COMMENT 'user sex',
    > birthday DATE COMMENT 'user birthday',
    > address STRING COMMENT 'user address'
    > )
    > COMMENT 'This is the use info table'
    > ROW FORMAT DELIMITED
    >  FIELDS TERMINATED BY '\t'
    > STORED AS TEXTFILE;
OK
Time taken: 0.521 seconds
hive> 
    > show tables;
OK
t_user
Time taken: 0.035 seconds, Fetched: 1 row(s)
hive> 
```

### 查看表结构
``` bash
hive> desc t_user;
OK
id                  	int
name                	string              	user name
age                 	int                 	user age
sex                 	string              	user sex
birthday            	date                	user birthday
address             	string              	user address
Time taken: 0.074 seconds, Fetched: 6 row(s)
hive> 
```

### 插入数据
``` bash
hive> INSERT INTO t_user(id, name, age, sex, birthday, address) VALUES(1, 'Tim Ho', 23, 'M', '1989-05-01', 'Higher Education Mega Center South, Guangzhou city, Guangdong Province');
Query ID = hadoop_20190102160558_640a90a7-9122-4650-af78-acb436e2643b
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1546186928725_0015, Tracking URL = http://hadoop-host-master:8088/proxy/application_1546186928725_0015/
Kill Command = /home/hadoop/hadoop-2.7.3/bin/hadoop job  -kill job_1546186928725_0015
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
2019-01-02 16:06:08,341 Stage-1 map = 0%,  reduce = 0%
2019-01-02 16:06:14,565 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.39 sec
MapReduce Total cumulative CPU time: 1 seconds 390 msec
Ended Job = job_1546186928725_0015
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to: hdfs://hadoop-cluster-ha/user/hive/warehouse/tim.db/t_user/.hive-staging_hive_2019-01-02_16-05-58_785_7094384272339204067-1/-ext-10000
Loading data to table tim.t_user
Table tim.t_user stats: [numFiles=1, numRows=1, totalSize=96, rawDataSize=95]
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1   Cumulative CPU: 1.39 sec   HDFS Read: 4763 HDFS Write: 162 SUCCESS
Total MapReduce CPU Time Spent: 1 seconds 390 msec
OK
Time taken: 17.079 seconds
hive> 
```

执行插入操作它会产生一个mapReduce任务。

### 查询数据
``` bash
hive> select * from t_user;
OK
1	Tim Ho	23	M	1989-05-01	Higher Education Mega Center South, Guangzhou city, Guangdong Province
Time taken: 0.196 seconds, Fetched: 1 row(s)
hive> 
    > select * from t_user where name='Tim Ho';
OK
1	Tim Ho	23	M	1989-05-01	Higher Education Mega Center South, Guangzhou city, Guangdong Province
Time taken: 0.258 seconds, Fetched: 1 row(s)
hive> 
    > select count(*) from t_user;
Query ID = hadoop_20190102161100_d60df721-539d-4e5b-a3db-a4951ac884b4
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1546186928725_0016, Tracking URL = http://hadoop-host-master:8088/proxy/application_1546186928725_0016/
Kill Command = /home/hadoop/hadoop-2.7.3/bin/hadoop job  -kill job_1546186928725_0016
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2019-01-02 16:11:10,739 Stage-1 map = 0%,  reduce = 0%
2019-01-02 16:11:16,997 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.05 sec
2019-01-02 16:11:23,280 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 2.37 sec
MapReduce Total cumulative CPU time: 2 seconds 370 msec
Ended Job = job_1546186928725_0016
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 2.37 sec   HDFS Read: 7285 HDFS Write: 2 SUCCESS
Total MapReduce CPU Time Spent: 2 seconds 370 msec
OK
1
Time taken: 24.444 seconds, Fetched: 1 row(s)
hive> 
```

由上面可知，执行简单的查询操作不会启动mapReduce，但执行像COUNT这样的统计操作将会产生一个mapReduce。

### 从文件中导入数据
语法：

    LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]

我们可以按定义表结构时的使用的字段分隔符(\t)，将数据存放在文本文件里，然后使用LOAD命令来导入。例如我们将数据存放在`/home/hadoop/user.txt`中：
``` txt
2	scott	25	M	1977-10-21	USA
3	tiger	21	F	1977-08-12	UK
```

然后在hive中执行LOAD命令：
``` bash
hive> LOAD DATA LOCAL INPATH '/home/hadoop/user.txt' INTO TABLE t_user;
Loading data to table tim.t_user
Table tim.t_user stats: [numFiles=2, numRows=0, totalSize=151, rawDataSize=0]
OK
Time taken: 0.214 seconds
hive> 
    > select * from t_user;
OK
1	Tim Ho	23	M	1989-05-01	Higher Education Mega Center South, Guangzhou city, Guangdong Province
2	scott	25	M	1977-10-21	USA
3	tiger	21	F	1977-08-12	UK
Time taken: 0.085 seconds, Fetched: 3 row(s)
hive> 
```

未完待续……

[link_id_hive-home]: https://hive.apache.org/
[link_id_hadoop-cluster-ha]: ../../../../2019/01/01/hadoop-cluster-ha/
[link_id_hive-1-2-2]: http://archive.apache.org/dist/hive/hive-1.2.2/
[link_id_mysql-connector-java]: http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.42/mysql-connector-java-5.1.42.jar

