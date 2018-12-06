---
title: hadoop 集群的搭建
date: 2018-12-04 09:12:43
tags: hadoop
categories: bigdata
---

本篇将说说hadoop集群的搭建，这里说的集群是真集群，不是伪集群。不过，这里的真集群是在虚拟机环境中搭建的。

在我的笔记本电脑中，安装虚拟机VirtualBox，在虚拟机中安装三台服务器：master、slave1、slave2来搭建hadoop集群。安装好VirtualBox后，启动它。依次点`File -> Host Network Manager -> Create`，来创建一个网络和虚拟机中的机器通讯，这个地址是：`192.168.56.1`，也就是我们外面实体机的地址（仅和虚拟机中的机器通讯使用）。如下图：

![](/img/hadoop-1.png "虚拟机网络配置")

我们使用`ubuntu 18.04`来作为我们的服务器，先在虚拟机中安装好一台服务器master，将Jdk、hadoop在上面安装好，然后将master克隆出slave1、slave2。以master为namenode节点，slave1、slave2作为datanode节点。相关配置如下：

    master：
        ip: 192.168.56.110
        hostname: hadoop-host-master
    slave1:
        ip: 192.168.56.111
        hostname: hadoop-host-slave-1
    slave2:
        ip: 192.168.56.112
        hostname: hadoop-host-slave-2


### 下面开始master的安装
在虚拟机中安装`master`的过程中我们会设置一个用户用于登录，我们将用户名、密码都设为`hadoop`，当然也可以为其他名字，其他安装过程略。安装好之后，使用默认的网关配置NAT，NAT可以访问外网，我们将`jdk-8u102-linux-x64.tar.gz`和`hadoop-2.7.3.tar.gz`从它们的官网下载到用户的`/home/hadoop/`目录下。或在实体机中通过SCP命令传进去。然后将网关设置为`Host-only Adapter`，如下图所示。

![](/img/hadoop-2.png "网络配置")

网关设置好了之后，我们接下来配置IP地址。在`master`中`[Settings] -> [Network] -> [Wired 这里打开] -> [IPv4]`按如下设置：

![](/img/hadoop-3.png "网络配置")


### 管理集群
在上面的IP等配置好之后，我们选择关闭master，注意不是直接关闭，而是在关闭的时候选择`Save the machine state`。然后在虚拟机中选中`master -> Start 下拉箭头 -> Headless start`，然后在我们实体机中通过ssh直接登录到master。
``` bash
$ ssh hadoop@192.168.56.110
```

我们可以在实体机通过配置`/etc/hosts`，加上如下配置：

    192.168.56.110	hadoop-host-master

然后就可以通过如下方式登录了
``` bash
$ ssh hadoop@hadoop-host-master
```

在实体机中通过下面的配置，就可以无密码登录了：
``` bash
$ ssh-copy-id hadoop@hadoop-host-master
```

** 下面的操作，均是在实体机中通过SSH到虚拟机执行的操作。 **


### 安装ssh openssh rsync
项系统已安装，则省略下面的安装操作
``` bash
$ sudo apt-get install ssh openssh-server rsync
```

如果上述命令无法执行，请先执行如下命令：
``` bash
$ sudo apt-get update
```

JDK的安装请参考我之前的笔记：[安装 JDK][link_id_install-jdk]，这里不再赘述。安装到此目录`/usr/local/jdk1.8.0_102/`下，记住此路径，下面会用到。下在进行hadoop的安装。

``` bash
$ cd /home/hadoop/
$ tar xf hadoop-2.7.3.tar.gz
```

解压后得到hadoop-2.7.3目录，hadoop的程序和相关配置就在此目录中。


### 建保存数据的目录
``` bash
$ cd /home/hadoop/hadoop-2.7.3
$ mkdir -p hdfs/tmp
$ mkdir -p hdfs/name
$ mkdir -p hdfs/data
$
$ chmod -R 777 hdfs/
```


### 配置文件浏览
hadoop的配置文件都位于下面的目录下：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/etc/hadoop
$ ls 
capacity-scheduler.xml      httpfs-env.sh            mapred-env.sh
configuration.xsl           httpfs-log4j.properties  mapred-queues.xml.template
container-executor.cfg      httpfs-signature.secret  mapred-site.xml.template
core-site.xml               httpfs-site.xml          slaves
hadoop-env.cmd              kms-acls.xml             ssl-client.xml.example
hadoop-env.sh               kms-env.sh               ssl-server.xml.example
hadoop-metrics2.properties  kms-log4j.properties     yarn-env.cmd
hadoop-metrics.properties   kms-site.xml             yarn-env.sh
hadoop-policy.xml           log4j.properties         yarn-site.xml
hdfs-site.xml               mapred-env.cmd
```


### 配置hadoop-env.sh，加上JDK绝对路径
JDK的路径就是上面安装JDK的时候的路径：
``` xml
export JAVA_HOME=/usr/local/jdk1.8.0_102/
```


### 配置core-site.xml，在该文件中加入如下内容
``` xml
<configuration>
        <property>
             <name>hadoop.tmp.dir</name>
             <value>file:/home/hadoop/hadoop-2.7.3/hdfs/tmp</value>
             <description>Abase for other temporary directories.</description>
        </property>
        <property>
             <name>fs.defaultFS</name>
             <value>hdfs://hadoop-host-master:9000</value>
        </property>
</configuration>
```


### 配置hdfs-site.xml，在该文件中加入如下内容
``` xml
<configuration>
        <property>
             <name>dfs.nameservices</name>
             <value>hadoop-cluster</value>
        </property>
        <property>
             <name>dfs.replication</name>
             <value>2</value>
        </property>
        <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/home/hadoop/hadoop-2.7.3/hdfs/name</value>
        </property>
        <property>
             <name>dfs.datanode.data.dir</name>
             <value>file:/home/hadoop/hadoop-2.7.3/hdfs/data</value>
        </property>
</configuration>
```

至此，master中要安装的通用环境配置完成。在虚拟机中将master复制出slave1、slave2。并参考上面配置IP地址的方法将slave1的ip配置为:`192.168.56.111`，slave2的ip配置为：`192.168.56.112`。


### 配置主机名
配置master的主机名为`hadoop-host-master`，在master节点执行如下操作：
``` bash
$ sudo vi /etc/hostname

修改为如下内容
hadoop-host-master
```

配置slave1的主机名为`hadoop-host-slave-1`，在slave1节点执行如下操作：
``` bash
$ sudo vi /etc/hostname

修改为如下内容
hadoop-host-slave-1
```

配置slave2的主机名为`hadoop-host-slave-2`，在slave2节点执行如下操作：
``` bash
$ sudo vi /etc/hostname

修改为如下内容
hadoop-host-slave-2
```

** 注意：各个节点的主机名一定要不同，否则相同主机名的节点，只会有一个连得上namenode节点，并且集群会报错，修改主机名后，要重启才生效。 **


### 配置域名解析
分别对master、slave1和slave2都执行如下操作：
``` bash
$ sudo vi /etc/hosts

修改为如下内容
127.0.0.1	localhost
192.168.56.110	hadoop-host-master
192.168.56.111	hadoop-host-slave-1
192.168.56.112	hadoop-host-slave-2
```

至此，集群配置完成，下面将启动集群。


### 启动集群
首先启动nodename节点，也就是master，首次启动的时候，要格式化namenode。
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./bin/hdfs namenode -format    # 再次启动的时候不需要执行此操作
$ ./sbin/hadoop-daemon.sh start namenode
$
$ jps    # 查看是否启动成功
4016 Jps
2556 NameNode
```

接下来启动datanode节点，也就是slave1、slave2，在这两台服务器上都执行如下启动脚本。
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/hadoop-daemon.sh start datanode
$
$ jps    # 查看是否启动成功
2451 Jps
2162 DataNode
```

可以在实体机的浏览器中输入：
http://hadoop-host-master:50070/
来查看是否启动成功。

![](/img/hadoop-4.png "hadoop管理界面：Overview")

切换tab到Datanodes可以看到有2个datanode节点，如下图所示：

![](/img/hadoop-5.png "hadoop管理界面:Datanodes")

切换到`Utilities -> Browse the file system`，如下图所示：

![](/img/hadoop-6.png "hadoop管理界面:Browse the file system")

从上面的界面可以，目前HDFS中没有任何文件。我们尝试往其中放一个文件，就将我们的hadoop压缩包放进去，在namenode节点中执行如下操作：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./bin/hdfs dfs -put /home/hadoop/hadoop-2.7.3.tar.gz /

$ ./bin/hdfs dfs -ls /
Found 1 items
-rw-r--r--   2 hadoop supergroup  214092195 2018-12-06 12:20 /hadoop-2.7.3.tar.gz
```
我们在图形界面中查看，如下图：

![](/img/hadoop-7.png "hadoop管理界面:Browse the file system")

我们点击列表中的文件，将会显示它的数据具体分布在哪些节点上，如下图：

![](/img/hadoop-8.png "hadoop管理界面:Browse the file system")


### 停掉集群
先停掉datanode节点，在slave1、slave2上面执行如下命令：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/hadoop-daemon.sh stop datanode
```

然后停掉namenode节点，在master上面执行如下命令：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/hadoop-daemon.sh stop namenode
```


### 集中式管理集群
如果我们的集群里面有成千上万台机器，在每一台机器上面都这样来启动，肯定是不行的。下面我们将通过配置，只在一台机器上面执行一个脚本，就将整个集群启动。

配置SSH无密码登陆，分别在master、slave1和slave2上面执行如下脚本：
``` bash
$ ssh-keygen -t rsa -P ""
```

在master上面执行如下脚本：
``` bash
$ ssh-copy-id hadoop-host-master
$ ssh-copy-id hadoop-host-slave-1
$ ssh-copy-id hadoop-host-slave-2
```
每执行一条命令的时候，都先输入yes，然后再输入目标机器的登录密码。

如果能成功运行如下命令，则配置免密登录其他机器成功。
``` bash
$ ssh hadoop-host-master
$ ssh hadoop-host-slave-1
$ ssh hadoop-host-slave-2
```

在master上面执行如下脚本：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/etc/hadoop/
$ vi slaves    # 加入如下内容
$
hadoop-host-slave-1
hadoop-host-slave-2
```
当执行`start-dfs.sh`时，它会去slaves文件中找从节点。

在master上面启动整个集群：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/start-dfs.sh
```

分别在master、slave1和slave2上面通过`jps`命令可以看到整个集群已经成功启动。同样的，停掉整个集群的命令，如下，同样是在master上面执行：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/stop-dfs.sh
```

相关操作，如下图所示：

![](/img/hadoop-9.png "hadoop集中式管理")

** 注意：在主节点执行`start-dfs.sh`，主节点的用户名必须和所有从节点的用户名相同。因为主节点服务器以这个用户名去远程登录到其他从节点的服务器中，所以在所有的生产环境中控制同一类集群的用户一定要相同。 **


### 启动YARN
分别在master、slave1和slave2上面都建如下目录：
``` bash
$ cd /home/hadoop/hadoop-2.7.3
$ mkdir -p yarn/nm
$
$ chmod -R 777 yarn/
```

分别在master、slave1和slave2上面都按如下方式配置mapred-site.xml，刚解压的hadoop是没有mapred-site.xml的，但是有mapred-site.xml.template，我们修改文件名，并作如下配置：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/etc/hadoop/
$ mv mapred-site.xml.template mapred-site.xml
$ vi mapred-site.xml

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

分别在master、slave1和slave2上面都按如下方式配置yarn-site.xml
``` bash
$ cd /home/hadoop/hadoop-2.7.3/etc/hadoop/
$ vi yarn-site.xml

<configuration>
    <!-- 指定ResourceManager的地址 -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop-host-master</value>
    </property>
    <!-- 指定reducer获取数据的方式 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>file:/home/hadoop/hadoop-2.7.3/yarn/nm</value>
    </property>
</configuration>
```

配置好了，下面开始启动：
在master上面启动resourcemanager
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/yarn-daemon.sh start resourcemanager
```

在slave1、slave2上面分别启动nodemanager
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/yarn-daemon.sh start nodemanager
```

我们可以通过浏览器，查看资源管理器：
http://hadoop-host-master:8088/

![](/img/hadoop-10.png "yarn资源管理")

点击图中的`Active Nodes`可以看到下图的详情，（如果`Unhealthy Nodes`有节点，则可能是由于虚拟机中主机的磁盘空间不足所致）。

![](/img/hadoop-11.png "yarn资源管理")

当然相应的停止命令如下：
``` bash
$ ./sbin/yarn-daemon.sh stop resourcemanager
$ ./sbin/yarn-daemon.sh stop nodemanager
```

如果有配置集中式管理，我们也可以通过在master上面通过一个命令启动、停止YARN
``` bash
$ ./sbin/start-yarn.sh    # 启动yarn
$ ./sbin/stop-yarn.sh    # 停止yarn
```

或者在master上面，通过一个命令启动hadoop和yarn
``` bash
$ ./sbin/start-all.sh
或者按顺序执行如下两个命令
$ ./sbin/start-dfs.sh
$ ./sbin/start-yarn.sh
```

![](/img/hadoop-12.png "yarn集中启动管理")


### 启动MR作业日志管理器
在nodename节点，也就是master，启动MR作业日志管理器。
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./sbin/mr-jobhistory-daemon.sh start historyserver
```
它同样有自已的图形界面：
http://hadoop-host-master:19888/

![](/img/hadoop-13.png "MR作业日志管理器")


### 尝试向集群中提交一个mapReduce任务
我们在namenode节点中向集群提交一个计算圆周率的mapReduce任务：
``` bash
$ cd /home/hadoop/hadoop-2.7.3/
$ ./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar pi 4 10
```

![](/img/hadoop-14.png "计算圆周率")

![](/img/hadoop-15.png "计算圆周率")

从上图可以看出，圆周率已经被计算出来：`3.40`。另外，在yarn中也可以看到任务的执行情况：
![](/img/hadoop-16.png "计算圆周率")


至此， 集群搭建完毕。

[link_id_install-jdk]: ../../../../2017/12/08/install-jdk/
