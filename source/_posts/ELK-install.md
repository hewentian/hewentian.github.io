---
title: ELK 日志系统的搭建
date: 2018-10-02 11:09:56
tags: elasticsearch
categories: bigdata
---

本篇将介绍 ELK 日志系统的搭建，我们将在一台机器上面搭建，系统配置如下：
![](/img/system-property.png "系统配置")


`logstash`的整体结构图如下：
![](/img/elk-structure.png "来源：https://www.elastic.co/guide/en/logstash/current/static/images/basic_logstash_pipeline.png")


我们将使用`redis`作为上图中的`INPUTS`，而`elasticsearch`作为上图中的`OUTPUTS`，这也是`logstash`官方的推荐。而它们的安装可以参考以下例子：
`redis`的安装请参考：[redis 的安装使用][link_id_redis-install]
`elasticsearch`的安装请参考：[elasticsearch 单节点安装][link_id_elasticsearch-install]

** 注意：elasticsearch、logstash、kibana它们的版本最好保持一致，这里都是使用6.4.0版本。 **

### `kibana`的安装将在本篇的稍后介绍，下面先介绍下`logstash`的安装

首先，我们要将`logstash`安装包下载回来，可以在它的[官网][link_id_logstash-6.4.0.tar.gz]下载，当然，我们也可以从这里下载 [logstash-6.4.0.tar.gz](https://pan.baidu.com/s/10p4YqzwSk1ixLqvSuv2sAA "百度网盘")，推荐从`logstash`官网下载对应版本。

``` bash
$ cd /home/hewentian/ProjectD/
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz.sha512

验证下载文件的完整性，在下载的时候要将 SHA512 文件也下载回来
$ sha512sum -c logstash-6.4.0.tar.gz.sha512 
logstash-6.4.0.tar.gz: OK

$ tar xzf logstash-6.4.0.tar.gz
```

解压后，得到目录`logstash-6.4.0`，可以查看下它包含有哪些文件
``` bash
$ cd /home/hewentian/ProjectD/logstash-6.4.0
$ ls

bin           data          lib          logstash-core             NOTICE.TXT  x-pack
config        Gemfile       LICENSE.txt  logstash-core-plugin-api  tools
CONTRIBUTORS  Gemfile.lock  logs         modules                   vendor
```

#### 测试安装是否成功：以标准输入、标准输出作为input, output
``` bash
$ cd /home/hewentian/ProjectD/logstash-6.4.0/bin
$ ./logstash -e 'input { stdin { } } output { stdout { } }'

Sending Logstash logs to /home/hewentian/ProjectD/logstash-6.4.0/logs which is now configured via log4j2.properties
[2018-10-02T14:25:37,017][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2018-10-02T14:25:38,201][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"6.4.0"}
[2018-10-02T14:25:41,748][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50}
[2018-10-02T14:25:41,919][INFO ][logstash.pipeline        ] Pipeline started successfully {:pipeline_id=>"main", :thread=>"#<Thread:0x4c1685e4 run>"}
The stdin plugin is now waiting for input:
[2018-10-02T14:25:41,990][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2018-10-02T14:25:42,396][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}

#此时窗口在等待输入

hello world

#下面是logstash的输出结果

{
      "@version" => "1",
    "@timestamp" => 2018-10-02T06:25:59.608Z,
       "message" => "hello world",
          "host" => "hewentian-Lenovo-IdeaPad-Y470"
}
```
从上面的测试结果可知，软件安装正确，下面开始我们的定制配置。


配置文件放在config目录下，此目录下已经有一个示例配置，因为我们要将redis作为我们的INPUTS，所以我们要建立它的配置文件：
``` bash
$ cd /home/hewentian/ProjectD/logstash-6.4.0/config
$ cp logstash-sample.conf logstash-redis.conf
$ 
$ vi logstash-redis.conf
```

在`logstash-redis.conf`中配置如下，这里暂未配置FILTERS（后面会讲到如何配置）：
``` bash
# Sample Logstash configuration for creating a simple
# Redis -> Logstash -> Elasticsearch pipeline.

input {
  redis {
    type => "systemlog"
    host => "127.0.0.1"
    port => 6379
    password => "abc123"
    db => 0
    data_type => "list"
    key => "systemlog"
  }
}

output {
  if [type] == "systemlog" {
    elasticsearch {
      hosts => ["http://127.0.0.1:9200"]
      index => "redis-systemlog-%{+YYYY.MM.dd}"
      #user => "elastic"
      #password => "changeme"
    }
  }
}
```

在启动`logstash`前，验证一下配置文件是否正确，这是一个好习惯：
``` bash
$ cd /home/hewentian/ProjectD/logstash-6.4.0/bin
$ ./logstash -f ../config/logstash-redis.conf -t
```
如果你见到如下输出，则配置正确：

	Sending Logstash logs to /home/hewentian/ProjectD/logstash-6.4.0/logs which is now configured via log4j2.properties
	[2018-09-30T16:32:45,043][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.queue", :path=>"/home/hewentian/ProjectD/logstash-6.4.0/data/queue"}
	[2018-09-30T16:32:45,064][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.dead_letter_queue", :path=>"/home/hewentian/ProjectD/logstash-6.4.0/data/dead_letter_queue"}
	[2018-09-30T16:32:46,030][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
	Configuration OK
	[2018-09-30T16:32:50,630][INFO ][logstash.runner          ] Using config.test_and_exit mode. Config Validation Result: OK. Exiting Logstash

接下来，就可以启动logstash了：
``` bash
$ cd /home/hewentian/ProjectD/logstash-6.4.0/bin
$ ./logstash -f ../config/logstash-redis.conf
```

如果见到如下输出，则启动成功：

	[2018-09-30T16:34:44,175][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}

#### 下面进行简单的测试
我们首先，往redis中推入3条记录：
``` bash
$ cd /home/hewentian/ProjectD/redis-4.0.11_master/src
$ ./redis-cli -h 127.0.0.1
127.0.0.1:6379> AUTH abc123
OK
127.0.0.1:6379> lpush systemlog hello world
(integer) 2
127.0.0.1:6379> lpush systemlog '{"name":"Tim Ho","age":23,"student":true}'
(integer) 1
```

启动elastchsearch-head可以看到数据已经进入到es中了：
![](/img/elk-head-1.png "elk-head-1")

你会发现上面推到`systemlog`中的信息如果是JSON格式，则在elasticsearch中会自动解析到相应的field中，否则会放到默认的field：`message`中。


### kibana的安装
`kibana`的安装很简单，将`kibana`安装包下载回来，可以在它的[官网][link_id_kibana-6.4.0-linux-x86_64.tar.gz]下载，当然，我们也可以从这里下载 [kibana-6.4.0-linux-x86_64.tar.gz](https://pan.baidu.com/s/1-h0z7DR2uhuwhCn0_vNc4w "百度网盘")，推荐从`kibana`官网下载对应版本。

``` bash
$ cd /home/hewentian/ProjectD/
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz.sha512

验证下载文件的完整性，在下载的时候要将 SHA512 文件也下载回来
$ sha512sum -c kibana-6.4.0-linux-x86_64.tar.gz.sha512 
kibana-6.4.0-linux-x86_64.tar.gz: OK

$ tar xzf kibana-6.4.0-linux-x86_64.tar.gz
```

对`kibana`配置要查看的`elasticsearch`，只需修改如下配置项即可，如果是在本机安装`elasticsearch`，并且使用默认的9200端口，则无需配置。
``` bash
$ cd /home/hewentian/ProjectD/kibana-6.4.0-linux-x86_64/config
$ vi kibana.yml

#修改如下配置项，如果使用默认的，则无需修改
#server.port: 5601
#elasticsearch.url: "http://localhost:9200"
#elasticsearch.username: "user"
#elasticsearch.password: "pass"
```

接着启动`kibana`：
``` bash
$ cd /home/hewentian/ProjectD/kibana-6.4.0-linux-x86_64/bin
$ ./kibana # 或者以后台方式运行 nohup ./kibana &
```

打开浏览器，并输入下面的地址：
http://localhost:5601

你将看到如下界面：
![](/img/elk-kibana-1.png "kibana初始界面")

点击上图中的`[Management]->[Index Patterns]->[Create index pattern]`，输入`index name：redis-systemlog-*`，如下图
![](/img/elk-kibana-2.png "kibana配置index name界面")

点击`[Next step]`按钮，并在接下来的界面中的`Time Filter field name`中选择`I don't want to user the Time Filter`，最后点击`Create index pattern`完成创建。接着点击左则的`[Discover]`并在左则的界面中选择中`redis-systemlog-*`，你将看到如下结果：
![](/img/elk-kibana-3.png "kibana查询界面")

至此，简单的 ELK 基本搭建完毕。下面展示一个简单的配置示例：
``` bash
# Sample Logstash configuration for creating a simple
# Redis -> Logstash -> Elasticsearch pipeline.

input {
  # system log
  redis {
    type => "systemlog"
    host => "127.0.0.1"
    port => 6379
    password => "abc123"
    db => 0
    data_type => "list"
    key => "systemlog"
    codec  => "json"
  }
  
  # user log
  redis {
    type => "userlog"
    host => "127.0.0.1"
    port => 6379
    password => "abc123"
    db => 0
    data_type => "list"
    key => "userlog"
    codec  => "json"
  }
}

output {
  elasticsearch {
    hosts => ["http://127.0.0.1:9200"]
    index => "%{type}-%{+YYYY.MM}"
  }
}
```


### 下面我们将继续探索它的高级功能。

很多时候，对于`systemlog`中的某条信息（不一定是JSON格式），如果我们只需要某些信息，那我们又怎样做呢？这里就需要使用FILTERS了。

在FILTERS中使用grok正则表达式，关于grok，可以参见这里的说明：
https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html

未完，待续……

[link_id_redis-install]: ../../../../2018/08/07/redis-install/
[link_id_elasticsearch-install]: ../../../../2018/09/16/elasticsearch-install/
[link_id_logstash-6.4.0.tar.gz]: https://artifacts.elastic.co/downloads/logstash/logstash-6.4.0.tar.gz
[link_id_kibana-6.4.0-linux-x86_64.tar.gz]: https://artifacts.elastic.co/downloads/kibana/kibana-6.4.0-linux-x86_64.tar.gz
