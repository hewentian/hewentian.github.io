---
title: elasticsearch 单节点安装
date: 2018-09-16 11:18:34
tags: elasticsearch
categories: bigdata
---

本文将说下`elasticsearch`的单节点安装，我的机器为`Ubuntu 16.04 LTS`，当前用户为hewentian

首先，我们要将`elasticsearch`安装包下载回来，截止本文写时，它的最新版本为`6.4.0`，可以在它的[官网][link_id_elasticsearch-6.4.0.tar.gz]下载，当然，我们也可以从这里下载 [elasticsearch-6.4.0.tar.gz](/download/elasticsearch-6.4.0.tar.gz) 和 [elasticsearch-6.4.0.tar.gz.sha512](/download/elasticsearch-6.4.0.tar.gz.sha512)，推荐从`elasticsearch`官网下载最新版本。

``` bash
$ cd /home/hewentian/ProjectD/
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz.sha512

验证下载文件的完整性，在下载的时候要将 SHA512 文件也下载回来
$ sha512sum -c elasticsearch-6.4.0.tar.gz.sha512 
elasticsearch-6.4.0.tar.gz: OK

$ tar xzf elasticsearch-6.4.0.tar.gz
```

对`elasticsearch`进行设置（目前是单节点，所以也可以不对`elasticsearch.yml`进行设置，可直接跳过）：
``` bash
$ cd /home/hewentian/ProjectD/elasticsearch-6.4.0/config
$ vi elasticsearch.yml 		# 增加下面的配置

cluster.name: hewentian-cluster	# 配置集群的名字
node.name: node-1		# 配置集群下的节点的名字

network.host: 127.0.0.1		# 设置本机IP地址，这里可不设置，但是在集群环境中必须设置

http.port: 9200			# 默认也是这个端口

下面的配置保持默认即可，它会将数据和日志保存到elasticsearch-6.4.0目录下的data和logs目录
#path.data: /path/to/data
# 
# Path to log files:
#
#path.logs: /path/to/logs
```

内存大小的设置，根据机器内存大小而设置，一般不超过系统总内存的一半：
``` bash
$ cd /home/hewentian/ProjectD/elasticsearch-6.4.0/config
$ vi jvm.options

-Xms1g
-Xmx1g
```

启动`elasticsearch`：
``` bash
$ cd /home/hewentian/ProjectD/elasticsearch-6.4.0/bin
$ ./elasticsearch
```

我们也可以在启动的时候加上参数`-d`，以后台运行方式启动：`./elasticsearch -d`。这样单节点版本的`elasticsearch`就安装完了，在浏览器中输入如下地址：http://localhost:9200/

	{
	"name" : "node-1",
  	"cluster_name" : "hewentian-cluster",
  	"cluster_uuid" : "ihLk1iOlTEis2PkQrLhmLQ",
  	"version" : {
    	"number" : "6.4.0",
    	"build_flavor" : "default",
    	"build_type" : "tar",
    	"build_hash" : "595516e",
    	"build_date" : "2018-08-17T23:18:47.308994Z",
    	"build_snapshot" : false,
    	"lucene_version" : "7.4.0",
    	"minimum_wire_compatibility_version" : "5.6.0",
    	"minimum_index_compatibility_version" : "5.0.0"
  	},
  	"tagline" : "You Know, for Search"
	}

如果你见到上面的输出，证明安装成功了。


不过，你在安装的过程中，有可能会遇到下面的问题之一：

	ERROR: [3] bootstrap checks failed
	[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
	[2]: max number of threads [2048] for user [hewentian] is too low, increase to at least [4096]
	[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决方法如下，就算它不报上面的错误，我们也应该根据机器的性能，对下面的选项作相应配置（下面是根据我机器的情况作的配置）：
``` bash
$ su root			# 必须在 root 下才有权限修改系统配置文件
Password: 
$ vi /etc/security/limits.conf	# 添加如下配置
* soft nofile 65536		# 上面第一个错误有提示
* hard nofile 131072		# 一般为 soft nofile 的2倍
* soft nproc 4096		# 这个设置线程数
* hard nproc 8192

$ vi /etc/sysctl.conf		# 添加如下配置
vm.max_map_count=262144

$ sysctl -p			# 最后执行这个命令，你会见到如下输出
vm.max_map_count=262144

$ exit				# 退出 root
```

#### elasticsearch-head的安装
为了更方便的与elasticsearch交互，我们还要安装`elasticsearch-head`插件，安装步骤如下：
``` bash
$ cd /home/hewentian/ProjectD/gitHub
$ git clone https://github.com/mobz/elasticsearch-head.git
$ cd elasticsearch-head
$ npm install 		# 这个安装过程可能会报错，但是一般不影响。安装完之后，运行下面的命令即可
$ npm run start 	# 运行此命令，你会看到如下输出

> elasticsearch-head@0.0.0 start /home/hewentian/ProjectD/gitHub/elasticsearch-head
> grunt server

Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100

也可以使用下面这种方式启动
$ nohup npm run start &
```
安装结束之后，可以试着打开这个连接：http://localhost:9100/
这个连接可以打开，就证明`elasticsearch-head`安装成功，但是你可能会发现，它无法连上`elasticsearch`。因为，我们还没有对`elasticsearch`进行设置：
``` bash
$ cd /home/hewentian/ProjectD/elasticsearch-6.4.0/config
$ vi elasticsearch.yml 	# 增加下面的配置

允许跨域，否则 elasticsearch head 不能访问 elasticsearch
http.cors.enabled: true
http.cors.allow-origin: "*"
```

配置好之后，重启`elasticsearch`，就可以使用`elasticsearch-head`访问`elasticsearch`了，如下图所示：
![](/img/elasticsearch-head.png "elasticsearch-head")


### elasticsearch使用示例
使用示例我不打算自已写，因为`elasticsearch`官方的`README.textile`已经写得非常详细了。下面的示例摘自`elasticsearch-6.4.0/README.textile`：

要单独创建一个索引：
<pre>
curl -XPUT 'http://localhost:9200/music/'

{"acknowledged":true,"shards_acknowledged":true,"index":"music"}
</pre>

删除索引的命令，如下：
<pre>
curl -XDELETE 'http://localhost:9200/music/'

{"acknowledged":true}
</pre>


#### Indexing

Let's try and index some twitter like information. First, let's index some tweets (the @twitter@ index will be created automatically):

<pre>
curl -XPUT 'http://localhost:9200/twitter/doc/1?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T13:12:00",
    "message": "Trying out Elasticsearch, so far so good?"
}'

curl -XPUT 'http://localhost:9200/twitter/doc/2?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T14:12:12",
    "message": "Another tweet, will it be indexed?"
}'

curl -XPUT 'http://localhost:9200/twitter/doc/3?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "elastic",
    "post_date": "2010-01-15T01:46:38",
    "message": "Building the site, should be kewl"
}'
</pre>


#### Getting

Now, let's see if the information was added by GETting it:

<pre>
curl -XGET 'http://localhost:9200/twitter/doc/1?pretty=true'
curl -XGET 'http://localhost:9200/twitter/doc/2?pretty=true'
curl -XGET 'http://localhost:9200/twitter/doc/3?pretty=true'

</pre>

The results show below:
<pre>
{
  "_index" : "twitter",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "kimchy",
    "post_date" : "2009-11-15T13:12:00",
    "message" : "Trying out Elasticsearch, so far so good?"
  }
}

{
  "_index" : "twitter",
  "_type" : "doc",
  "_id" : "2",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "Another tweet, will it be indexed?"
  }
}

{
  "_index" : "twitter",
  "_type" : "doc",
  "_id" : "3",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "elastic",
    "post_date" : "2010-01-15T01:46:38",
    "message" : "Building the site, should be kewl"
  }
}
</pre>


#### Updating

The updating operation is the same as Indexing.

<pre>
curl -XPUT 'http://localhost:9200/twitter/doc/1?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T13:12:00",
    "message": "Trying out Elasticsearch, so far so good? yes"
}'

{
  "_index" : "twitter",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
</pre>


#### Deleting

The deleting operation is also easy.

<pre>
curl -XDELETE 'http://localhost:9200/twitter/doc/1'

{"_index":"twitter","_type":"doc","_id":"1","_version":3,"result":"deleted","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":2,"_primary_term":1}
</pre>

#### Searching

Mmm search..., shouldn't it be elastic?
Let's find all the tweets that @kimchy@ posted:

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?q=user:kimchy&pretty=true'

{
  "took" : 42,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "twitter",
        "_type" : "doc",
        "_id" : "2",
        "_score" : 0.2876821,
        "_source" : {
          "user" : "kimchy",
          "post_date" : "2009-11-15T14:12:12",
          "message" : "Another tweet, will it be indexed?"
        }
      },
      {
        "_index" : "twitter",
        "_type" : "doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "user" : "kimchy",
          "post_date" : "2009-11-15T13:12:00",
          "message" : "Trying out Elasticsearch, so far so good?"
        }
      }
    ]
  }
}
</pre>

We can also use the JSON query language Elasticsearch provides instead of a query string:

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match" : { "user": "kimchy" }
    }
}'
</pre>

Just for kicks, let's get all the documents stored (we should see the tweet from @elastic@ as well):

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match_all" : {}
    }
}'
</pre>

We can also do range search (the @post_date@ was automatically identified as date)

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "range" : {
            "post_date" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" }
        }
    }
}'
</pre>

There are many more options to perform search, after all, it's a search product no? All the familiar Lucene queries are available through the JSON query language, or through the query parser.

#### Multi Tenant - Indices and Types

Man, that twitter index might get big (in this case, index size == valuation). Let's see if we can structure our twitter system a bit differently in order to support such large amounts of data.

Elasticsearch supports multiple indices. In the previous example we used an index called @twitter@ that stored tweets for every user.

Another way to define our simple twitter system is to have a different index per user (note, though that each index has an overhead). Here is the indexing curl's in this case:

<pre>
curl -XPUT 'http://localhost:9200/kimchy/doc/1?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T13:12:00",
    "message": "Trying out Elasticsearch, so far so good?"
}'

curl -XPUT 'http://localhost:9200/kimchy/doc/2?pretty' -H 'Content-Type: application/json' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T14:12:12",
    "message": "Another tweet, will it be indexed?"
}'
</pre>

The above will index information into the @kimchy@ index. Each user will get their own special index.

Complete control on the index level is allowed. As an example, in the above case, we would want to change from the default 5 shards with 1 replica per index, to only 1 shard with 1 replica per index (== per twitter user). Here is how this can be done (the configuration can be in yaml as well):

<pre>
curl -XPUT http://localhost:9200/another_user?pretty -H 'Content-Type: application/json' -d '
{
    "index" : {
        "number_of_shards" : 1,
        "number_of_replicas" : 1
    }
}'
</pre>

Search (and similar operations) are multi index aware. This means that we can easily search on more than one
index (twitter user), for example:

<pre>
curl -XGET 'http://localhost:9200/kimchy,another_user/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match_all" : {}
    }
}'
</pre>

Or on all the indices:

<pre>
curl -XGET 'http://localhost:9200/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match_all" : {}
    }
}'
</pre>

{One liner teaser}: And the cool part about that? You can easily search on multiple twitter users (indices), with different boost levels per user (index), making social search so much simpler (results from my friends rank higher than results from friends of my friends).


#### Cat Index

<pre>
curl -XGET 'http://localhost:9200/_cat/indices?v'

health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   twitter 1aZo0hSfRkKG1INAEZgpnQ   5   1          2            0       14kb           14kb
yellow open   music   ytI7cirvQXOi1hsrvAMGGA   5   1          0            0      1.2kb          1.2kb
</pre>


[link_id_elasticsearch-6.4.0.tar.gz]: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.tar.gz
