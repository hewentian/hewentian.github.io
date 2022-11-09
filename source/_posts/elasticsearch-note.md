---
title: elasticsearch 学习笔记
date: 2018-09-18 10:21:01
tags: elasticsearch
categories: bigdata
---

参考资料：
[Elasticsearch 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html "Elasticsearch 权威指南")
[Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html "Elasticsearch Reference")
http://es.xiaoleilu.com/080_Structured_Search/20_contains.html

首先，你必须至少有一台`elasticsearch`服务器可以使用，如果还没安装，可以参考我的上一篇 [elasticsearch 的安装][link_id_elasticsearch-install]

使用JAVA API来操作`elasticsearch`的例子可以在这里找到：[ElasticsearchUtil.java][link_id_ElasticsearchUtil]、[ElasticsearchDemo.java][link_id_ElasticsearchDemo]


### elasticsearch使用示例
`elasticsearch`的[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/elasticsearch-intro.html)写得非常详细。下面是一些快速连接：
https://www.elastic.co/guide/en/elasticsearch/reference/8.4/rest-apis.html
https://www.elastic.co/guide/en/elasticsearch/reference/8.4/indices.html
https://www.elastic.co/guide/en/elasticsearch/reference/8.4/query-dsl.html

下面的示例摘自官方文档：

**下面代码的缩进尽量不要使用tab，要使用空格**

如果使用curl命令，一律要加上ca证书，下面为简洁起见，省略了：
        curl --cacert /home/hewentian/ProjectD/elasticsearch-8.4.0/config/certs/http_ca.crt -u elastic


#### 创建索引
``` curl
curl -X PUT "https://localhost:9200/my-index-000001?pretty"
```

``` kibana
PUT /my-index-000001
```

创建索引的时候设置副本、分片
``` curl
curl -XPUT 'https://localhost:9200/my-index-000001' -H 'Content-Type: application/json' -d '{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  }
}'
```

``` kibana
PUT /my-index-000001
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  }
}
```

Default for number_of_shards is 1
Default for number_of_replicas is 1 (ie one replica for each primary shard)


#### 删除索引
``` kibana
DELETE /my-index-000001
```


#### 获取索引的settings
``` kibana
GET /my-index-000001/_settings
```


#### 修改索引的settings
elasticsearch在索引数据的时候，如果存在副本，那么主分片会将数据同时同步到副本。所以，如果当前插入大量数据，那么会对es集群造成一定的压力，所以我们最好把副本数设置为0；等数据建立完索引之后，再手动的将副本数更改到2，这样可以提高数据的索引效率。副本数最好是1-2个。

``` kibana
PUT /my-index-000001/_settings
{
  "index": {
    "number_of_replicas" : 2
  }
}
```


#### 索引创建mapping
``` kibana
PUT /my-index-000001/_mapping
{
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
            "type": "keyword"
       },
       "birthday": {
            "type": "date",
            "format": "strict_date_optional_time || epoch_millis || yyyy-MM-dd HH:mm:ss"
       }
   }
}
```


#### 获取索引的mapping
``` kibana
GET /my-index-000001/_mapping
```


#### Index API 索引数据
``` kibana
PUT /<target>/_doc/<_id>
POST /<target>/_doc/
PUT /<target>/_create/<_id>
POST /<target>/_create/<_id>
```

You cannot add new documents to a data stream using the PUT /<target>/_doc/<_id> request format. To specify a document ID, use the PUT /<target>/_create/<_id> format instead.

If the Elasticsearch security features are enabled, you must have the following index privileges for the target data stream, index, or index alias:

1. To add or overwrite a document using the PUT /<target>/_doc/<_id> request format, you must have the create, index, or write index privilege.
2. To add a document using the POST /<target>/_doc/, PUT /<target>/_create/<_id>, or POST /<target>/_create/<_id> request formats, you must have the create_doc, create, index, or write index privilege.
3. To automatically create a data stream or index with an index API request, you must have the auto_configure, create_index, or manage index privilege.

Automatic data stream creation requires a matching index template with data stream enabled.


Let's try and index some twitter like information. First, let's index some tweets (the @twitter@ index will be created automatically):

``` kibana
PUT /twitter/_create/1?pretty
{
  "user": "kimchy",
  "post_date": "2009-11-15T13:12:00",
  "message": "Trying out Elasticsearch, so far so good?"
}

PUT /twitter/_create/2?pretty
{
    "user": "kimchy",
    "post_date": "2009-11-15T14:12:12",
    "message": "Another tweet, will it be indexed?"
}

PUT /twitter/_create/3?pretty
{
    "user": "elastic",
    "post_date": "2010-01-15T01:46:38",
    "message": "Building the site, should be kewl"
}
```


### Get API 获取数据
Retrieves the specified JSON document from an index.

``` kibana
GET <index>/_doc/<_id>
HEAD <index>/_doc/<_id>
GET <index>/_source/<_id>
HEAD <index>/_source/<_id>
```

Now, let's see if the information was added by GETting it:
``` kibana
GET /twitter/_doc/1?pretty=true
GET /twitter/_doc/2?pretty=true
GET /twitter/_doc/3?pretty=true


The results show below:
{
  "_index": "twitter",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "user": "kimchy",
    "post_date": "2009-11-15T13:12:00",
    "message": "Trying out Elasticsearch, so far so good?"
  }
}

{
  "_index": "twitter",
  "_id": "2",
  "_version": 1,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "user": "kimchy",
    "post_date": "2009-11-15T14:12:12",
    "message": "Another tweet, will it be indexed?"
  }
}

{
  "_index": "twitter",
  "_id": "3",
  "_version": 1,
  "_seq_no": 2,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "user": "elastic",
    "post_date": "2010-01-15T01:46:38",
    "message": "Building the site, should be kewl"
  }
}
```


#### Update API
Updates a document using the specified script.
``` kibana
POST /<index>/_update/<_id>
```

If both doc and script are specified, then doc is ignored. If you specify a scripted update, include the fields you want to update in the script.

文档是不可变的：他们不能被修改，只能被替换。 update API 必须遵循同样的规则。 从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， update API 简单使用与之前描述相同的 检索-修改-重建索引 的处理过程。区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。通过减少检索和重建索引步骤之间的时间，我们也减少了其他进程的变更带来冲突的可能性。

方法一：update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。 例如，我们增加字段 tags 和 views 到我们的博客文章，如下所示：
``` kibana
POST twitter/_update/1
{
  "doc": {
    "message": "Trying out Elasticsearch, so far so good? yes",
    "tags": ["java"],
    "views": 0
  }
}
```

方法二：使用脚本部分更新文档编辑
脚本可以在 update API中用来改变 _source 的字段内容，它在更新脚本中称为 ctx._source 。例如，我们可以使用脚本来增加博客文章中 views 的数量：

``` kibana
POST twitter/_update/1
{
  "script" : {
    "source": "ctx._source.views += params.views",
    "lang": "painless",
    "params" : {
      "views" : 4
    }
  }
}

POST twitter/_update/1
{
  "script": {
    "source": "ctx._source.tags.add(params.tag)",
    "lang": "painless",
    "params": {
      "tag": "lucene"
    }
  }
}

POST twitter/_update/1
{
  "script": {
    "source": "ctx._source.message=\"yes, you are right.\"",
    "lang": "painless"
  }
}

POST twitter/_update/1
{
  "script" : "ctx._source.new_field = 'value_of_new_field'"
}

POST twitter/_update/1
{
  "script" : "ctx._source.remove('new_field')"
}
```


### Delete API
Removes a JSON document from the specified index.
``` kibana
DELETE /<index>/_doc/<_id>
```

根据查询结果删除，注意这里发的是`POST`请求
``` kibana
POST /my-index-000001/_delete_by_query?refresh&slices=3&pretty=true
{
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}
```

在ES里面删除数据的时候要非常小心，如果全部都清空了，可能整个库的mapping都会有问题。这时，一些原先可以执行的语句可能会无法执行。


### Search API 搜索
Just for kicks, let's get all the documents stored (we should see the tweet from @elastic@ as well):
``` kibana
GET /twitter/_search?pretty=true
{
  "query": {
    "match_all": {}
  }
}
```

Let's find all the tweets that @kimchy@ posted:
``` kibana
GET /twitter/_search?pretty=true
{
  "query": {
    "match": {
      "user": "kimchy"
    }
  }
}
```

We can also do range search (the @post_date@ was automatically identified as date)
``` kibana
GET /twitter/_search?pretty=true
{
  "query": {
    "range": {
      "post_date": {
        "from": "2009-11-15T13:00:00",
        "to": "2009-11-15T14:15:00"
      }
    }
  }
}
```

在指定索引（所有索引、指定单个或多个索引）上查询：
``` kibana
PUT /my-index-000001/_create/1?pretty
{
  "user": "kimchy",
  "post_date": "2009-11-15T13:12:00",
  "message": "Trying out Elasticsearch, so far so good?"
}

GET /_search?pretty=true
{
  "query": {
    "match_all": {}
  }
}


GET /twitter,my-index-000001/_search?pretty=true
{
  "query": {
    "match_all": {}
  }
}


简单联合查询
GET /user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "scott"
          }
        }
      ],
      "should": [
        {
          "range": {
            "age": {
              "gte": 10,
              "lte": 20
            }
          }
        }
      ]
    }
  },
  "size": 20
}


term查询
GET /user/_search
{
  "query": {
    "term": {
      "name": {
        "value": "scott"
      }
    }
  },
  "size": 20
}
```


### 聚合查询
``` kibana
GET /user/_search
{
  "query": {
    "match": {
      "name": "scott"
    }
  },
  "aggs": {
    "age-histogram": {
      "histogram": {
        "field": "age",
        "interval": 22
      }
    }
  },
  "size": 0
}


GET /user/_search
{
  "aggs": {
    "name-term": {
      "terms": {
        "field": "name.keyword",
        "size": 10
      }
    },
    "age-term": {
      "terms": {
        "field": "age",
        "size": 10
      }
    }
  },
  "size": 0
}
```


### 查询返回某些指定的字段
如果查询的结果字段很多，而我们仅需要其中的某些字段的时候，我们可能通过`_source`来指定，比如我们只要user, message：
``` kibana
GET /twitter/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "user": "elastic"
          }
        }
      ]
    }
  },
  "_source": [
    "user",
    "message"
  ],
  "size": 2
}
```


#### 展示所有索引
``` kibana
GET /_cat/indices?v
```

health status index           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   twitter         yyRB0a9LRP63voi75TEGrw   1   1          2            0      5.3kb          5.3kb
yellow open   my-index-000001 Rc4yqBZWRCGqXeujM7Gq2Q   1   2          0            0       225b           225b


### 统计指定索引下的文档数：
``` kibana
GET /_cat/count/my-index-000002?v

epoch      timestamp count
1661084399 12:19:59  3
```


#### ES别名
别名的好处，就是更换索引的时候，系统业务代码无需修改。

查询所有别名：
``` kibana
GET _alias
```

查询某个索引的别名：
``` kibana
GET my-index-000001/_alias
```

查询某个别名对应的索引：
``` kibana
GET _alias/your_index_alia_name
```

添加别名：
``` kibana
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "my-index-000001",
        "alias": "my-idx-1"
      }
    }
  ]
}
```

删除别名：
``` kibana
POST _aliases
{
  "actions": [
    {
      "remove": {
        "index": "my-index-000001",
        "alias": "my-idx-1"
      }
    }
  ]
}
```

修改别名，ES没有修改别名的操作，只能先删除后添加：
``` kibana
POST _aliases
{
  "actions": [
    {
      "remove": {
        "index": "my-index-000001",
        "alias": "my-idx-1"
      }
    },
    {
      "add": {
        "index": "my-index-000001",
        "alias": "my-idx-001"
      }
    }
  ]
}
```


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

**cluster.name**
配置es的集群名称，默认是elasticsearch，不同的集群用名字来区分，es会自动发现在同一网段下的es，配置成相同集群名字的各个节点形成一个集群。如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群。

**http.port**
设置对外服务的http端口，默认为9200。不能相同，否则会冲突。


### 日期
Date formats can be customised, but if no format is specified then it uses the default:

        "strict_date_optional_time||epoch_millis"

if you set it like this:
``` kibana
PUT /my-index-000002

PUT /my-index-000002/_mapping
{
  "properties": {
    "date": {
      "type": "date",
      "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
    }
  }
}
```

you can use the below method to set date:
``` kibana
PUT /my-index-000002/_create/1?pretty
{
  "date": "2015-01-01 12:10:30"
}

PUT /my-index-000002/_create/2?pretty
{
  "date": "2015-01-02"
}

PUT /my-index-000002/_create/3?pretty
{
  "date": 1420070400001
}
```


### 分页查询
ES一次查询，最多返回10条，但hits会显示total一共有多少条，要使用from, size指定。


### 深度翻页问题

#### scroll search
https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results

We no longer recommend using the scroll API for deep pagination. If you need to preserve the index state while paging through more than 10,000 hits, use the search_after parameter with a point in time (PIT).

Scrolling is not intended for real time user requests, but rather for processing large amounts of data, e.g. in order to reindex the contents of one data stream or index into a new data stream or index with a different configuration.

The results that are returned from a scroll request reflect the state of the data stream or index at the time that the initial search request was made, like a snapshot in time. Subsequent changes to documents (index, update or delete) will only affect later search requests.

In order to use scrolling, the initial search request should specify the scroll parameter in the query string, which tells Elasticsearch how long it should keep the “search context” alive (see Keeping the search context alive), eg ?scroll=1m.

``` kibana
POST /my-index-000001/_search?scroll=1m
{
  "size": 3,
  "query": {
    "match": {
      "name": "scott"
    }
  },
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}
```


The result from the above request includes a _scroll_id, which should be passed to the scroll API in order to retrieve the next batch of results.
``` kibana
POST /_search/scroll
{
  "scroll" : "1m",
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ=="
}
```

* GET or POST can be used and the URL should not include the index name — this is specified in the original search request instead.
* The scroll parameter tells Elasticsearch to keep the search context open for another 1m.
* The scroll_id parameter


clear scroll
``` kibana
DELETE /_search/scroll
{
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ=="
}
```


ES默认的分页机制一个不足的地方是，比如有5010条数据，当你仅想取第5000到5010条数据的时候，ES也会将前5000条数据加载到内存当中。从价值观上来看，使用大量的CPU，内存和带宽，分类过程确实会变得非常重要。 为此，我们强烈建议不要进行深度分页。
``` kibana
{
  "query": {
    "bool": {
      "must": [],
      "must_not": [],
      "should": [
        {
          "wildcard": {
            "orgName": "*有限公司"
          }
        },
        {
          "wildcard": {
            "orgName": "*株式会社"
          }
        }
      ]
    }
  },
  "from": 1000000,
  "size": 100,
  "sort": [],
  "aggs": {}
}
```

    Result window is too large, from + size must be less than or equal to: [1000000] but was [1000100]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level setting.


要解决这个问题，可以使用下面的方式来改变ES默认深度分页的`index.max_result_window`最大窗口值
``` kibana
PUT /my_index/_settings
{
  "index": {
    "max_result_window": 500000
  }
}
```

其中my_index为要修改的index名，500000为要调整的新的窗口数

或者分页使用ES的scroll api实现：
``` java

private static void scrollSearch() throws IOException {
    String searchText = "scott";

    Time time = Time.of(t -> t.time("1m"));

    ResponseBody<User> response = elasticsearchClient.search(s -> s
                    .index(indexName)
                    .query(q -> q
                            .match(t -> t
                                    .field("name")
                                    .query(searchText)
                            )
                    )
                    .scroll(time)
                    .sort(SortOptions.of(so -> so.field(FieldSort.of(f -> f.field("age").order(SortOrder.Asc)))))
                    .size(3)
            , User.class
    );

    TotalHits total = response.hits().total();
    boolean isExactResult = total.relation() == TotalHitsRelation.Eq;

    if (isExactResult) {
        System.out.println("There are " + total.value() + " results");
    } else {
        System.out.println("There are more than " + total.value() + " results");
    }

    do {
        System.out.println("-----------------------------------");
        List<Hit<User>> hits = response.hits().hits();
        for (Hit<User> hit : hits) {
            User user = hit.source();
            System.out.println("Found user: " + user + ", score " + hit.score());
        }

        String scrollId = response.scrollId();
        System.out.println("scrollId: " + scrollId);

        response = elasticsearchClient.scroll(s -> s.scrollId(scrollId).scroll(time), User.class);
    } while (response.hits().hits().size() != 0);
}
```

#### search_after(pit search)
https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#search-after

Avoid using from and size to page too deeply or request too many results at once. Search requests usually span multiple shards. Each shard must load its requested hits and the hits for any previous pages into memory. For deep pages or large sets of results, these operations can significantly increase memory and CPU usage, resulting in degraded performance or node failures.

By default, you cannot use from and size to page through more than 10,000 hits. This limit is a safeguard set by the index.max_result_window index setting. If you need to page through more than 10,000 hits, use the search_after parameter instead.

You can use the search_after parameter to retrieve the next page of hits using a set of sort values from the previous page.

Using search_after requires multiple search requests with the same query and sort values. The first step is to run an initial request.

``` kibana
GET /my-index-000001/_search
{
  "size": 3,
  "query": {
    "match": {
      "name": "scott"
    }
  },
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}
```

To retrieve the next page of results, repeat the request, take the sort values from the last hit, and insert those into the search_after array:
``` kibana
GET /my-index-000001/_search
{
  "size": 3,
  "query": {
    "match": {
      "name": "scott"
    }
  },
  "search_after": [20],
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}
```

Repeat this process by updating the search_after array every time you retrieve a new page of results. If a refresh occurs between these requests, the order of your results may change, causing inconsistent results across pages. To prevent this, you can create a point in time (PIT) to preserve the current index state over your searches.

``` kibana
POST /my-index-000001/_pit?keep_alive=1m
```

The API returns a PIT ID.

{
  "id": "o93qAwEEdXNlchZfVWRURzY0UFJISzljbnZPTzhlaXhRABZWd1A3b3BIM1NpV1pTdHNsYmxENm5nAAAAAAAAAz-XFmNtX1lpSE9hVEx5T1Q2anRHQ1NZVWcAARZfVWRURzY0UFJISzljbnZPTzhlaXhRAAA="
}


To get the first page of results, submit a search request with a sort argument. If using a PIT, specify the PIT ID in the pit.id parameter and omit the target data stream or index from the request path.
``` kibana
GET /_search
{
  "size": 3,
  "query": {
    "match": {
      "name": "scott"
    }
  },
  "pit": {
    "id": "o93qAwEEdXNlchZfVWRURzY0UFJISzljbnZPTzhlaXhRABZWd1A3b3BIM1NpV1pTdHNsYmxENm5nAAAAAAAAAz-XFmNtX1lpSE9hVEx5T1Q2anRHQ1NZVWcAARZfVWRURzY0UFJISzljbnZPTzhlaXhRAAA=",
    "keep_alive": "1m"
  },
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}
```

To get the next page of results, rerun the previous search using the last hit’s sort values (including the tiebreaker) as the search_after argument. If using a PIT, use the latest PIT ID in the pit.id parameter. The search’s query and sort arguments must remain unchanged. If provided, the from argument must be 0 (default) or -1.
``` kibana
GET /_search
{
  "size": 3,
  "query": {
    "match": {
      "name": "scott"
    }
  },
  "pit": {
    "id": "o93qAwEEdXNlchZfVWRURzY0UFJISzljbnZPTzhlaXhRABZWd1A3b3BIM1NpV1pTdHNsYmxENm5nAAAAAAAAAz-XFmNtX1lpSE9hVEx5T1Q2anRHQ1NZVWcAARZfVWRURzY0UFJISzljbnZPTzhlaXhRAAA=",
    "keep_alive": "1m"
  },
  "search_after": [20, 7],
  "track_total_hits": false,
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}
```

When you’re finished, you should delete your PIT.
``` kibana
DELETE /_pit
{
    "id" : "o93qAwEEdXNlchZfVWRURzY0UFJISzljbnZPTzhlaXhRABZWd1A3b3BIM1NpV1pTdHNsYmxENm5nAAAAAAAAAz-XFmNtX1lpSE9hVEx5T1Q2anRHQ1NZVWcAARZfVWRURzY0UFJISzljbnZPTzhlaXhRAAA="
}
```


### pinyin分词的使用
1.1 使用pinyin分词器创建一个索引
``` kibana
PUT /my-index-000003/
{
  "settings": {
    "analysis": {
      "analyzer": {
        "pinyin_analyzer": {
          "tokenizer": "my_pinyin"
        }
      },
      "tokenizer": {
        "my_pinyin": {
          "type": "pinyin",
          "keep_separate_first_letter": false,
          "keep_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "lowercase": true,
          "remove_duplicated_term": true
        }
      }
    }
  }
}
```

1.2 测试分词器，分析一个中文名字，例如中：刘德华
``` kibana
GET /my-index-000003/_analyze
{
  "text": [
    "刘德华"
  ],
  "analyzer": "pinyin_analyzer"
}


{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "刘德华",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 2
    }
  ]
}
```

1.3 创建mapping
``` kibana
POST /my-index-000003/_mapping
{
  "properties": {
    "name": {
      "type": "keyword",
      "fields": {
        "pinyin": {
          "type": "text",
          "store": false,
          "term_vector": "with_offsets",
          "analyzer": "pinyin_analyzer"
        }
      }
    }
  }
}
```

1.4 索引数据
``` kibana
POST /my-index-000003/_create/andy
{
  "name": "刘德华"
}
```

1.5 查询全部数据
``` kibana
GET /my-index-000003/_search
{
  "query": {
    "match_all": {}
  }
}

{
  "took": 891,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "my-index-000003",
        "_id": "andy",
        "_score": 1,
        "_source": {
          "name": "刘德华"
        }
      }
    ]
  }
}
```

1.6 使用拼音分词搜索，下面的搜索方式，都能搜索到数据
``` kibana
GET /my-index-000003/_search
{
  "query": {
    "match": {
      "name": "刘德华"
    }
  }
}

GET /my-index-000003/_search
{
  "query": {
    "match": {
      "name.pinyin": "刘德华"
    }
  }
}

GET /my-index-000003/_search
{
  "query": {
    "match": {
      "name.pinyin": "ldh"
    }
  }
}

GET /my-index-000003/_search
{
  "query": {
    "match": {
      "name.pinyin": "liu"
    }
  }
}

GET /my-index-000003/_search
{
  "query": {
    "match": {
      "name.pinyin": "de hua"
    }
  }
}
```


2.1 使用pinyin Token Filter，首先创建一个索引
``` kibana
PUT /my-index-000004/
{
  "settings": {
    "analysis": {
      "analyzer": {
        "user_name_analyzer": {
          "tokenizer": "whitespace",
          "filter": "pinyin_first_letter_and_full_pinyin_filter"
        }
      },
      "filter": {
        "pinyin_first_letter_and_full_pinyin_filter": {
          "type": "pinyin",
          "keep_first_letter": true,
          "keep_full_pinyin": true,
          "keep_none_chinese": true,
          "keep_original": false,
          "limit_first_letter_length": 16,
          "lowercase": true,
          "trim_whitespace": true,
          "keep_none_chinese_in_first_letter": true
        }
      }
    }
  }
}
```

2.2 测试分词器，分析一个中文名字，例如中：刘德华 张学友 郭富城 黎明 四大天王
``` kibana
GET /my-index-000004/_analyze
{
  "text": [
    "刘德华 张学友 郭富城 黎明 四大天王"
  ],
  "analyzer": "user_name_analyzer"
}


{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 0
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "zhang",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 3
    },
    {
      "token": "xue",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 4
    },
    {
      "token": "you",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 5
    },
    {
      "token": "zxy",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 5
    },
    {
      "token": "guo",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 6
    },
    {
      "token": "fu",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 7
    },
    {
      "token": "cheng",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 8
    },
    {
      "token": "gfc",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 8
    },
    {
      "token": "li",
      "start_offset": 12,
      "end_offset": 14,
      "type": "word",
      "position": 9
    },
    {
      "token": "ming",
      "start_offset": 12,
      "end_offset": 14,
      "type": "word",
      "position": 10
    },
    {
      "token": "lm",
      "start_offset": 12,
      "end_offset": 14,
      "type": "word",
      "position": 10
    },
    {
      "token": "si",
      "start_offset": 15,
      "end_offset": 19,
      "type": "word",
      "position": 11
    },
    {
      "token": "da",
      "start_offset": 15,
      "end_offset": 19,
      "type": "word",
      "position": 12
    },
    {
      "token": "tian",
      "start_offset": 15,
      "end_offset": 19,
      "type": "word",
      "position": 13
    },
    {
      "token": "wang",
      "start_offset": 15,
      "end_offset": 19,
      "type": "word",
      "position": 14
    },
    {
      "token": "sdtw",
      "start_offset": 15,
      "end_offset": 19,
      "type": "word",
      "position": 14
    }
  ]
}
```


3.1 使用phrase query，首先创建一个索引
``` kibana
PUT /my-index-000005/
{
  "settings": {
    "analysis": {
      "analyzer": {
        "pinyin_analyzer": {
          "tokenizer": "my_pinyin"
        }
      },
      "tokenizer": {
        "my_pinyin": {
          "type": "pinyin",
          "keep_first_letter": false,
          "keep_separate_first_letter": false,
          "keep_full_pinyin": true,
          "keep_original": false,
          "limit_first_letter_length": 16,
          "lowercase": true
        }
      }
    }
  }
}
```

3.2 测试分词器，分析一个中文名字，例如中：刘德华
``` kibana
GET /my-index-000005/_analyze
{
  "text": [
    "刘德华"
  ],
  "analyzer": "pinyin_analyzer"
}

{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 2
    }
  ]
}
```

3.3 索引数据
``` kibana
POST /my-index-000005/_create/andy
{
  "name": "刘德华"
}
```

3.4 使用分词搜索，使用下面的搜索方式，是搜索不到数据的
``` kibana
GET /my-index-000005/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "刘德华"
    }
  }
}
```


4.1 另一种使用phrase query，首先创建一个索引
``` kibana
PUT /my-index-000006/
{
  "settings": {
    "analysis": {
      "analyzer": {
        "pinyin_analyzer": {
          "tokenizer": "my_pinyin"
        }
      },
      "tokenizer": {
        "my_pinyin": {
          "type": "pinyin",
          "keep_first_letter": true,
          "keep_separate_first_letter": true,
          "keep_full_pinyin": true,
          "keep_original": false,
          "limit_first_letter_length": 16,
          "lowercase": true
        }
      }
    }
  }
}
```

4.2 创建mapping
``` kibana
POST /my-index-000006/_mapping
{
  "properties": {
    "name": {
      "type": "keyword",
      "fields": {
        "pinyin": {
          "type": "text",
          "store": false,
          "term_vector": "with_offsets",
          "analyzer": "pinyin_analyzer"
        }
      }
    }
  }
}
```

4.3 测试分词器
``` kibana
GET /my-index-000006/_analyze
{
  "text": [
    "刘德华"
  ],
  "analyzer": "pinyin_analyzer"
}

{
  "tokens": [
    {
      "token": "l",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "d",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 1
    },
    {
      "token": "de",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 1
    },
    {
      "token": "h",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 2
    },
    {
      "token": "hua",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 2
    }
  ]
}
```

4.4 索引数据
``` kibana
POST /my-index-000006/_create/andy
{
  "name": "刘德华"
}
```

4.5 搜索数据，下面的搜索方式，都能搜索到数据
``` kibana
GET /my-index-000006/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "刘德h"
    }
  }
}

GET /my-index-000006/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "刘dh"
    }
  }
}

GET /my-index-000006/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "liudh"
    }
  }
}

GET /my-index-000006/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "liudeh"
    }
  }
}

GET /my-index-000006/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "liude华"
    }
  }
}
```


### ik分词的使用
https://github.com/medcl/elasticsearch-analysis-ik

ik_max_word 和 ik_smart 的区别
ik_max_word：会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query；
ik_smart：会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”，适合 Phrase 查询。


1.1 使用ik分词器创建一个索引
``` kibana
PUT /my-index-000007/
```

1.2 创建mapping
``` kibana
POST /my-index-000007/_mapping
{
  "properties": {
    "content": {
      "type": "text",
      "analyzer": "ik_max_word",
      "search_analyzer": "ik_smart"
    }
  }
}
```

1.3 测试分词器
``` kibana
GET /my-index-000007/_analyze
{
  "text": [
    "中华人民共和国MN",
    "刘德华"
  ],
  "tokenizer": "ik_max_word"
}

{
  "tokens": [
    {
      "token": "中华人民共和国",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "中华人民",
      "start_offset": 0,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "中华",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "华人",
      "start_offset": 1,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "人民共和国",
      "start_offset": 2,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 4
    },
    {
      "token": "人民",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 5
    },
    {
      "token": "共和国",
      "start_offset": 4,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 6
    },
    {
      "token": "共和",
      "start_offset": 4,
      "end_offset": 6,
      "type": "CN_WORD",
      "position": 7
    },
    {
      "token": "国",
      "start_offset": 6,
      "end_offset": 7,
      "type": "CN_CHAR",
      "position": 8
    },
    {
      "token": "mn",
      "start_offset": 7,
      "end_offset": 9,
      "type": "ENGLISH",
      "position": 9
    },
    {
      "token": "刘德华",
      "start_offset": 10,
      "end_offset": 13,
      "type": "CN_WORD",
      "position": 110
    }
  ]
}
```

1.4 索引数据
``` kibana
POST /my-index-000007/_create/1
{
  "content": "美国留给叙利亚的是个烂摊子吗"
}

POST /my-index-000007/_create/2
{
  "content": "公安厅：各地校车将享最高路权"
}

POST /my-index-000007/_create/3
{
  "content": "中日渔警冲突调查：日警平均每天扣1艘中国渔船"
}

POST /my-index-000007/_create/4
{
  "content": "中国驻纽约领事馆遭亚裔男子枪击 嫌犯已自首"
}
```

1.5 搜索数据
``` kibana
GET /my-index-000007/_search
{
  "query": {
    "match": {
      "content": "中国"
    }
  },
  "highlight": {
    "pre_tags": [
      "<tag1>",
      "<tag2>"
    ],
    "post_tags": [
      "</tag1>",
      "</tag2>"
    ],
    "fields": {
      "content": {}
    }
  }
}

{
  "took": 70,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.642793,
    "hits": [
      {
        "_index": "my-index-000007",
        "_id": "3",
        "_score": 0.642793,
        "_source": {
          "content": "中日渔警冲突调查：日警平均每天扣1艘中国渔船"
        },
        "highlight": {
          "content": [
            "中日渔警冲突调查：日警平均每天扣1艘<tag1>中国</tag1>渔船"
          ]
        }
      },
      {
        "_index": "my-index-000007",
        "_id": "4",
        "_score": 0.642793,
        "_source": {
          "content": "中国驻纽约领事馆遭亚裔男子枪击 嫌犯已自首"
        },
        "highlight": {
          "content": [
            "<tag1>中国</tag1>驻纽约领事馆遭亚裔男子枪击 嫌犯已自首"
          ]
        }
      }
    ]
  }
}
```


### pinyin分词、ik分词联合使用
1.1 首先创建一个索引
``` kibana
PUT /my-index-000008/
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ik_smart_pinyin": {
          "tokenizer": "ik_smart",
          "filter": "pinyin_first_letter_and_full_pinyin_filter"
        },
        "ik_max_pinyin": {
          "tokenizer": "ik_max_word",
          "filter": "pinyin_first_letter_and_full_pinyin_filter"
        }
      },
      "filter": {
        "pinyin_first_letter_and_full_pinyin_filter": {
          "type": "pinyin",
          "keep_separate_first_letter": false,
          "keep_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "lowercase": true,
          "remove_duplicated_term": true
        }
      }
    }
  }
}
```

1.2 创建mapping
``` kibana
POST /my-index-000008/_mapping
{
  "properties": {
    "content": {
      "type": "text",
      "analyzer": "ik_smart_pinyin",
      "search_analyzer": "ik_max_pinyin"
    }
  }
}
```

1.3 测试分词器
``` kibana
GET /my-index-000008/_analyze
{
  "text": [
    "中华人民共和国MN",
    "刘德华"
  ],
  "analyzer": "ik_smart_pinyin"
}

{
  "tokens": [
    {
      "token": "zhong",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "hua",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "ren",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "min",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "gong",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 4
    },
    {
      "token": "he",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 5
    },
    {
      "token": "guo",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 6
    },
    {
      "token": "中华人民共和国",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 6
    },
    {
      "token": "zhrmghg",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 6
    },
    {
      "token": "m",
      "start_offset": 7,
      "end_offset": 9,
      "type": "ENGLISH",
      "position": 7
    },
    {
      "token": "n",
      "start_offset": 7,
      "end_offset": 9,
      "type": "ENGLISH",
      "position": 8
    },
    {
      "token": "mn",
      "start_offset": 7,
      "end_offset": 9,
      "type": "ENGLISH",
      "position": 8
    },
    {
      "token": "liu",
      "start_offset": 10,
      "end_offset": 13,
      "type": "CN_WORD",
      "position": 109
    },
    {
      "token": "de",
      "start_offset": 10,
      "end_offset": 13,
      "type": "CN_WORD",
      "position": 110
    },
    {
      "token": "hua",
      "start_offset": 10,
      "end_offset": 13,
      "type": "CN_WORD",
      "position": 111
    },
    {
      "token": "刘德华",
      "start_offset": 10,
      "end_offset": 13,
      "type": "CN_WORD",
      "position": 111
    },
    {
      "token": "ldh",
      "start_offset": 10,
      "end_offset": 13,
      "type": "CN_WORD",
      "position": 111
    }
  ]
}
```

1.4 索引数据
``` kibana
POST /my-index-000008/_create/1
{
  "content": "香港的刘德华"
}

POST /my-index-000008/_create/2
{
  "content": "香港的郭富城"
}

POST /my-index-000008/_create/3
{
  "content": "香港的林忆莲"
}

POST /my-index-000008/_create/4
{
  "content": "香港的周星驰"
}
```

1.5 搜索数据，下面的搜索方式，都能搜索到数据
``` kibana
GET /my-index-000008/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "liu"
          }
        }
      ]
    }
  }
}

GET /my-index-000008/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "刘"
          }
        }
      ]
    }
  }
}
```


### 以下是一些常用查询：
``` kibana
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "_id": "http://www.abc.com"
          }
        },
        {
          "match": {
            "_id": "http://www.csdn.net/tag/scala"
          }
        }
      ]
    }
  }
}

	
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "*canton*"
          }
        },
        {
          "match": {
            "name": "Tim"
          }
        }
      ]
    }
  }
}


{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              {
                "match": {
                  "title": {
                    "minimum_should_match": "100%",
                    "query": "Air Quality"
                  }
                }
              },
              {
                "match": {
                  "body_text": {
                    "minimum_should_match": "100%",
                    "query": "Air Quality"
                  }
                }
              }
            ]
          }
        },
        {
          "wildcard": {
            "user_ids": "*760aa069-2ed2-40d6-89da-f62e83f82887*"
          }
        }
      ]
    }
  },
  "from": 0,
  "size": 20
}


{
  "query": {
    "bool": {
      "must_not": [],
      "should": [
        {
          "bool": {
            "should": [
              {
                "match_phrase": {
                  "app_type.title": {
                    "query": "china"
                  }
                }
              },
              {
                "match_phrase": {
                  "app_type.title": {
                    "query": "中国"
                  }
                }
              }
            ]
          }
        },
        {
          "bool": {
            "should": [
              {
                "match_phrase": {
                  "app_type.body_text": {
                    "query": "china"
                  }
                }
              },
              {
                "match_phrase": {
                  "app_type.body_text": {
                    "query": "中国"
                  }
                }
              }
            ]
          }
        }
      ],
      "must": [],
      "filter": {
        "bool": {
          "should": [],
          "must": [
            {
              "range": {
                "updatetime": {
                  "lte": 1474617163524
                }
              }
            },
            {
              "wildcard": {
                "user_ids": "*760aa069-2ed2-40d6-89da-f62e83f82887*"
              }
            }
          ]
        }
      }
    }
  },
  "from": 0,
  "size": 20,
  "sort": [
    {
      "updatetime_6h": {
        "order": "desc"
      }
    },
    {
      "_score": {
        "order": "desc"
      }
    }
  ]
}


{
  "query": {
    "bool":{
      "must":[{
        "match":{"birthYear":1989}}
        ]
    }
  },
  "aggregations" : {
    "CATEGORY" : {
      "terms" : {
        "field" : "deposit",
        "size" : 100,
        "order" : {
          "_count" : "desc"
        }
      }
    }
  },
  "size":0
}


{
  "aggs": {
    "NAME": {
      "terms": {
        "field": "username",
        "size": 100
      }
    }
  },
  "query": {
    "bool": {
      "must": [
        {
          "wildcard": {
            "username": {
              "value": "*文*"
            }
          }
        }
      ]
    }
  },
  "size":100
}


{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "city": "深圳市"
          }
        },
        {
          "bool": {
            "should": [
              {
                "match": {
                  "industry": "制造业"
                }
              },
              {
                "match": {
                  "industry": "通用设备制造业"
                }
              },
              {
                "match": {
                  "industry": "专用设备制造业"
                }
              }
            ]
          }
        },
        {
          "range": {
            "patentCount": {
              "gte": 5
            }
          }
        },
        {
          "range": {
            "regCapital": {
               "gte": 200,
              "lte": 5000
            }
          }
        },
        {
          "match": {
            "isGaoXin": 0
          }
        },
        {
          "range": {
            "regYear": {
              "gte": 2015,
              "lte": 2017
            }
          }
        }
      ]
    }
  },
  "size": 20
}


{
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "username"
          }
        }
      ]
    }
  }
}
```


未完待续……

[link_id_elasticsearch-install]: ../../../../2018/09/16/elasticsearch-install "elasticsearch 的安装"
[link_id_ElasticsearchUtil]: https://github.com/hewentian/study-lib/blob/main/codes/bigdata/elasticsearch/src/main/java/com/hewentian/elasticsearch/util/ElasticsearchUtil.java
[link_id_ElasticsearchDemo]: https://github.com/hewentian/study-lib/blob/main/codes/bigdata/elasticsearch/src/main/java/com/hewentian/elasticsearch/ElasticsearchDemo.java

