---
title: elasticsearch 学习笔记
date: 2018-09-18 10:21:01
tags: elasticsearch
categories: bigdata
---

参考资料：
[Elasticsearch 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html "Elasticsearch 权威指南")
https://github.com/searchbox-io/Jest/tree/master/jest/src/test/java/io/searchbox/core


首先，你必须至少有一台`elasticsearch`服务器可以使用，如果还没安装，可以参考我的上两篇 [elasticsearch 单节点安装][link_id_elasticsearch-install]、[elasticsearch 集群的搭建][link_id_elasticsearch-cluster]

使用JAVA API来操作`elasticsearch`的例子可以在这里找到：[EsJestUtil.java][link_id_EsJestUtil]、[EsJestDemo.java][link_id_EsJestDemo]


#### 要单独创建一个索引
<pre>
curl -XPUT 'http://localhost:9200/user_index' -H 'Content-Type: application/json' -d '{
    "settings" : {
        "index" : {
            "number_of_shards" : 4,
            "number_of_replicas" : 1
        }
    }
}'

{"acknowledged":true,"shards_acknowledged":true,"index":"user_index"}
</pre>


#### 删除索引的命令
<pre>
curl -XDELETE 'http://localhost:9200/user_index/'

{"acknowledged":true}
</pre>


#### 为user_index中的user创建mapping
<pre>
curl -XPUT 'http://localhost:9200/user_index/_mapping/user' -H 'Content-Type: application/json' -d '{
    "properties": {
        "id": {
            "type": "long",
            "index": "false"
        },
        "name": {
            "type": "keyword"
        },
        "age": {
            "type": "integer"
        },
        "tags": {
            "type": "keyword",
            "boost": 3.0
       },
       "birthday": {
            "type": "date",
            "format": "strict_date_optional_time || epoch_millis || yyyy-MM-dd HH:mm:ss"
       }
   }
}'
</pre>


#### ES中的一些概念
**cluster**
代表一个集群，集群中有多个节点，其中有一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。es的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部来看es集群，在逻辑上是个整体，你与任何一个节点的通信和与整个es集群通信是等价的。

**shards**
代表索引分片，es可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。

**replicas**
代表索引副本，es可以设置多个索引的副本，副本的作用一是提高系统的容错性，当某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高es的查询效率，es会自动对搜索请求进行负载均衡。

**recovery**
代表数据恢复或叫数据重新分布，es在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

**river**
代表es的一个数据源，也是其它存储方式（如：数据库）同步数据到es的一个方法。它是以插件方式存在的一个es服务，通过读取river中的数据并把它索引到es中，官方的river有couchDB的，RabbitMQ的，Twitter的，Wikipedia的。

**gateway**
代表es索引快照的存储方式，es默认是先把索引存放到内存中，当内存满了时再持久化到本地硬盘。gateway对索引快照进行存储，当这个es集群关闭再重新启动时就会从gateway中读取索引备份数据。es支持多种类型的gateway，有本地文件系统（默认），分布式文件系统，Hadoop的HDFS和amazon的s3云存储服务。

**discovery.zen**
代表es的自动发现节点机制，es是一个基于p2p的系统，它先通过广播寻找存在的节点，再通过多播协议来进行节点之间的通信，同时也支持点对点的交互。

**Transport**
代表es内部节点或集群与客户端的交互方式，默认内部是使用tcp协议进行交互，同时它支持http协议（json格式）、thrift、servlet、memcached、zeroMQ等的传输协议（通过插件方式集成）。


Date formats can be customised, but if no format is specified then it uses the default:

	"strict_date_optional_time||epoch_millis"

if you set it like this:

	PUT my_index
	{
  	"mappings": {
    	"_doc": {
      	"properties": {
        	"date": {
          	"type":   "date",
          	"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        	}
      	}
    	}
  	}
	}

you can use the below method to set date:
<pre>
PUT my_index/_doc/1
{ "date": "2015-01-01 12:10:30" } 

PUT my_index/_doc/2
{ "date": "2015-01-01T12:10:30Z" } 

PUT my_index/_doc/3
{ "date": 1420070400001 }
</pre>

#### 文件的部分更新
文档是不可变的：他们不能被修改，只能被替换。 update API 必须遵循同样的规则。 从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， update API 简单使用与之前描述相同的 检索-修改-重建索引 的处理过程。 区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。通过减少检索和重建索引步骤之间的时间，我们也减少了其他进程的变更带来冲突的可能性。

方法一：update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。 例如，我们增加字段 tags 和 views 到我们的博客文章，如下所示：

	POST /website/blog/1/_update
	{
   		"doc" : {
      		"tags" : [ "testing" ],
      		"views": 0
   		}
	}

测试示例：
<pre>
先插件一条数据：
curl -XPUT 'http://localhost:9200/facebook/tuser/1?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "tim",
    "post_date": "2009-11-15T13:12:00",
    "message": "Elasticsearch, so far so good?"
}'

再将这条数所的 message 字段修改一下
curl -XPOST 'http://localhost:9200/facebook/tuser/1/_update' -H 'Content-Type: application/json' -d '
{
    "doc":{
    	"message": "Elasticsearch, so far so good? yes"
	}
}'
</pre>


方法二：使用脚本部分更新文档编辑
脚本可以在 update API中用来改变 _source 的字段内容， 它在更新脚本中称为 ctx._source 。 例如，我们可以使用脚本来增加博客文章中 views 的数量：

	POST /website/blog/1/_update
	{
   		"script" : "ctx._source.views+=1"
	}
<pre>
curl -XPOST 'http://127.0.0.1:9200/facebook/tuser/1/_update' -H 'Content-Type: application/json' -d '
{
    "script" : "ctx._source.message=\"yes, you are right.\""
}'
</pre>

下面的命令可以列出每个 Index 所包含的 Type
``` bash
$ curl -XGET 'http://127.0.0.1:9200/user_index/_mapping?pretty=true'
```

1. cluster.name 
配置es的集群名称，默认是elasticsearch，不同的集群用名字来区分，es会自动发现在同一网段下的es，配置成相同集群名字的各个节点形成一个集群。如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群。
2. http.port 
设置对外服务的http端口，默认为9200。不能相同，否则会冲突。

未完待续……

[link_id_elasticsearch-install]: ../../../../2018/09/16/elasticsearch-install "elasticsearch 单节点安装"
[link_id_elasticsearch-cluster]: ../../../../2018/09/17/elasticsearch-cluster "elasticsearch 集群的搭建"
[link_id_EsJestUtil]: https://github.com/hewentian/studyResource/blob/master/src/main/java/com/hewentian/util/EsJestUtil.java
[link_id_EsJestDemo]: https://github.com/hewentian/studyResource/blob/master/src/main/java/com/hewentian/es/EsJestDemo.java