---
title: elasticsearch 的安装
date: 2018-09-16 11:18:34
tags: elasticsearch
categories: bigdata
---

### 单节点的安装
本文将说下`elasticsearch`的单节点安装，我的机器为`Ubuntu 18.04 LTS`，当前用户为`hadoop`

首先，我们要将`elasticsearch`安装包下载回来，截止本文写时，它的最新版本为`8.4.0`，可以在它的[官网](https://www.elastic.co/downloads/elasticsearch)下载。

``` bash
$ cd /home/hadoop/
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.0-linux-x86_64.tar.gz
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.0-linux-x86_64.tar.gz.sha512

验证下载文件的完整性
$ sha512sum -c elasticsearch-8.4.0-linux-x86_64.tar.gz.sha512
elasticsearch-8.4.0-linux-x86_64.tar.gz: OK

$ tar -xzf elasticsearch-8.4.0-linux-x86_64.tar.gz
```

对`elasticsearch`进行设置（目前是单节点，所以也可以不对`elasticsearch.yml`进行设置，可直接跳过）：
``` bash
$ cd /home/hadoop/elasticsearch-8.4.0/config
$ vi elasticsearch.yml 		     # 增加下面的配置

cluster.name: hewentian-cluster	 # 配置集群的名字
node.name: node-1		         # 配置集群下的节点的名字

network.host: 192.168.56.113	 # 设置本机IP地址，这里可不设置，但是在集群环境中必须设置。如果在这里指定了IP，则localhost可能无法使用，这个要注意

http.port: 9200			         # 默认也是这个端口

下面的配置保持默认即可，它会将数据和日志保存到elasticsearch-8.4.0目录下的data和logs目录
#path.data: /path/to/data
# 
# Path to log files:
#
#path.logs: /path/to/logs

#cluster.initial_master_nodes: ["node-1"]  # 启动后，这个会自动产生
```

内存大小的设置，根据机器内存大小而设置，一般不超过系统总内存的一半：
``` bash
$ cd /home/hadoop/elasticsearch-8.4.0/config
$ vi jvm.options

-Xms1g
-Xmx1g
```

### 修改linux系统参数
你在安装的过程中，有可能会遇到下面的问题之一：

        bootstrap check failure [1] of [2]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
        bootstrap check failure [2] of [2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决方法如下，就算它不报上面的错误，我们也应该根据机器的性能，对下面的选项作相应配置（下面是根据我机器的情况作的配置）：
``` bash
$ su root			            # 必须在 root 下才有权限修改系统配置文件
Password: 
$ vi /etc/security/limits.conf	# 添加如下配置
* soft nofile 65536             # 上面第一个错误有提示
* hard nofile 131072            # 一般为 soft nofile 的2倍
* soft nproc 4096               # 这个设置线程数
* hard nproc 8192

$ vi /etc/sysctl.conf		    # 添加如下配置
vm.max_map_count=262144

$ sysctl -p			            # 最后执行这个命令，你会见到如下输出
vm.max_map_count=262144

$ exit				            # 退出 root
```

设置好之后，要重新登录，才会生效。


### 启动elasticsearch
注意，不能以root帐号启动elasticsearch。要用普通用户，并且要以 sudo 方式启动。

``` bash
$ cd /home/hadoop/elasticsearch-8.4.0/
$ sudo su hadoop
$ ./bin/elasticsearch


✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  wnh=7lOdD*uF=TpYny2x

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  da2e917d8c6cf3b1d32068092cff23985ad4fcc0c1eb0e6a215744ca6f1cc4e4

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjQuMCIsImFkciI6WyIxOTIuMTY4LjU2LjExMzo5MjAwIl0sImZnciI6ImRhMmU5MTdkOGM2Y2YzYjFkMzIwNjgwOTJjZmYyMzk4NWFkNGZjYzBjMWViMGU2YTIxNTc0NGNhNmYxY2M0ZTQiLCJrZXkiOiI1Z3lUdDRJQnVuYXpXOHNIdFFYUjpNZzdYSzRlLVFTbUxCQnVOQm5qNXhRIn0=

ℹ️ Configure other nodes to join this cluster:
• Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjQuMCIsImFkciI6WyIxOTIuMTY4LjU2LjExMzo5MjAwIl0sImZnciI6ImRhMmU5MTdkOGM2Y2YzYjFkMzIwNjgwOTJjZmYyMzk4NWFkNGZjYzBjMWViMGU2YTIxNTc0NGNhNmYxY2M0ZTQiLCJrZXkiOiI1d3lUdDRJQnVuYXpXOHNIdFFYazpoRzBUVzlwZVE4eWtJLW5Sdll0ZGFRIn0=

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.4.0`
```


You can complete the following actions at any time:
Reset the password of the elastic built-in superuser with
'bin/elasticsearch-reset-password -u elastic'.

Generate an enrollment token for Kibana instances with
'bin/elasticsearch-create-enrollment-token -s kibana'.

Generate an enrollment token for Elasticsearch nodes with
'bin/elasticsearch-create-enrollment-token -s node'.


我们也可以在启动的时候加上参数`-d`，以后台运行方式启动：`./bin/elasticsearch -d -p pid`，进程id将会记录在`$ES_HOME`目录下，相应的停止命令为：`pkill -F pid`。

这样单节点版本的`elasticsearch`就安装完了。


### 检查elasticsearch是否正确启动
使用命令
        curl --cacert $ES_HOME/config/certs/http_ca.crt -u elastic https://localhost:9200


``` bash
$ curl --cacert /home/hadoop/elasticsearch-8.4.0/config/certs/http_ca.crt -u elastic https://192.168.56.113:9200
Enter host password for user 'elastic':
{
  "name" : "node-1",
  "cluster_name" : "hewentian-cluster",
  "cluster_uuid" : "XnyeZ-UQSWmyocWOUSgyKA",
  "version" : {
    "number" : "8.4.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "f56126089ca4db89b631901ad7cce0a8e10e2fe5",
    "build_date" : "2022-08-19T19:23:42.954591481Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

如果你见到上面的输出，证明安装成功了。


### 安装kibana
kibana的版本，要和elasticsearch的版本一致，我们都下载`8.4.0`，可以在它的[官网](https://www.elastic.co/downloads/kibana)下载。

``` bash
$ cd /home/hadoop/
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-8.4.0-linux-x86_64.tar.gz
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-8.4.0-linux-x86_64.tar.gz.sha512

验证下载文件的完整性
$ sha512sum -c kibana-8.4.0-linux-x86_64.tar.gz.sha512
kibana-8.4.0-linux-x86_64.tar.gz: OK

$ tar -xzf kibana-8.4.0-linux-x86_64.tar.gz
```

配置kibana
``` bash
$ vi /home/hadoop/kibana-8.4.0/config/kibana.yml

server.port: 5601
server.host: "192.168.56.113"
```

启动 Kibana
``` bash
$ /home/hadoop/kibana-8.4.0/
$ ./bin/kibana


[2022-08-20T07:33:26.679+08:00][INFO ][node] Kibana process configured with roles: [background_tasks, ui]
[2022-08-20T07:33:36.041+08:00][INFO ][http.server.Preboot] http server running at http://192.168.56.113:5601
[2022-08-20T07:33:36.141+08:00][INFO ][plugins-system.preboot] Setting up [1] plugins: [interactiveSetup]
[2022-08-20T07:33:36.144+08:00][INFO ][preboot] "interactiveSetup" plugin is holding setup: Validating Elasticsearch connection configuration…
[2022-08-20T07:33:36.214+08:00][INFO ][root] Holding setup until preboot stage is completed.


i Kibana has not been configured.

Go to http://192.168.56.113:5601/?code=299251 to get started.
```

也可以以后台方式启动kibana：`nohup ./bin/kibana 2>&1 &`

打开 Kibana
1. 在浏览器打开`http://192.168.56.113:5601`，输入Enrollment token即可连接到 Elasticsearch。
2. 如果token过期，则要进入`$ES_HOME`目录，使用命令`bin/elasticsearch-create-enrollment-token -s kibana`重新生成一个token。
3. 用新生成的 token 粘贴到 kibana 页面上，此时kibana后台会产生一个验证码，类似：Your verification code is:  952 566，在页面上输入。
4. 最后，输入elasticsearch的用户名、密码，即可登录到kibana。


登录 Kibana 后，点击左则菜单的"Dev Tools"，即可以打开搜索的窗口。


### 集群的安装
https://www.elastic.co/guide/en/elasticsearch/reference/8.4/add-elasticsearch-nodes.html

我们将安装3个节点的集群。在3个节点上面，都要`修改linux系统参数`，具体设置方式见上面单节点安装。

我们将上面已经安装好的单节点，作为我们集群的第一个节点。然后，我们在另外两台机器上面，分别安装elasticsearch。

在`192.168.56.112`上面安装：
``` bash
$ cd /home/hadoop/
$ tar -xzf elasticsearch-8.4.0-linux-x86_64.tar.gz
$ vi elasticsearch-8.4.0/config/elasticsearch.yml

cluster.name: hewentian-cluster
node.name: node-2

network.host: 192.168.56.112
http.port: 9200

#transport.host    # Defaults to the address given by network.host
#transport.port    # Defaults to 9300-9400

#discovery.seed_hosts: ["192.168.56.111", "192.168.56.112", "192.168.56.113"]  # 这个不用配置，会自动产生
#cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

内存大小的设置，参考上面单节点安装。

到`192.168.56.113`节点上面的elasticsearch安装目录生成一个新的token：
``` bash
$ cd /home/hadoop/elasticsearch-8.4.0/
$ ./bin/elasticsearch-create-enrollment-token -s node
```

然后回到我们准备安装的新节点，启动elasticsearch，将`<enrollment-token>`替换成上面产生的token：
``` bash
$ cd /home/hadoop/elasticsearch-8.4.0/
$ ./bin/elasticsearch --enrollment-token <enrollment-token>
```

启动后，会自动在`/home/hadoop/elasticsearch-8.4.0/config`目前下生成一个`certs`目录，这个目录中的证书和`192.168.56.113`上的是一样的。

验证安装是否成功：
        curl --cacert /home/hadoop/elasticsearch-8.4.0/config/certs/http_ca.crt -u elastic https://192.168.56.112:9200

将上面的安装步骤，在`192.168.56.111`上面安装也执行一次。到此，集群安装完毕。


查看集群状态
``` kibana
GET _cat/health

GET _cat/nodes
```


重启集群步骤：
1. Disable shard allocation.
``` kibana
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
```

2. Stop indexing and perform a flush.
``` kibana
POST /_flush
```

3. 重启所有节点，重启节点的时候使用`./bin/elasticsearch`就可以了，不用再带`<enrollment-token>`参数。

4. Re-enable allocation.
``` kibana
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```


删除节点：
``` kibana
POST /_cluster/voting_config_exclusions?node_names=node_name

eg:
POST /_cluster/voting_config_exclusions?node_names=node-3
```

删除的节点重启后，会自动重新加入集群的。


