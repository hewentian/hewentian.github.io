---
title: solr提交索引的三种方式
date: 2017-10-29 10:59:11
tags: solr
categories: solr
---

## 1.commit
通过api直接commit,这样性能比较差

``` java
import org.apache.solr.client.solrj.SolrServerException;
import org.apache.solr.client.solrj.impl.HttpSolrClient;
import org.apache.solr.common.SolrInputDocument;

String baseSolrUrl = "http://localhost:8983/solr/";
HttpSolrClient solrClient = new HttpSolrClient.Builder().withBaseSolrUrl(baseSolrUrl).build();

SolrInputDocument solrInputDocument = new SolrInputDocument();
solrInputDocument.addField("id", "1");
solrInputDocument.addField("name", "Tim Ho");
solrInputDocument.addField("age", "23");

solrClient.add("mysqlCore", solrInputDocument);
solrClient.commit();
```

## 2.commitWithinMs
简单的说就是告诉solr在多少毫秒内提交，比如我指定commitWithinMs=10000，将会告诉solr在10s内提交我的document。
``` java
用法：
1.可以在add方法设置参数，比如：solrClient.add("mysqlCore", solrInputDocument, 10000);

2.UpdateRequest updateRequest = new UpdateRequest();
  updateRequest.add(solrInputDocument);
  updateRequest.setCommitWithin(10000);
  updateRequest.process(solrClient, "mysqlCore");
```
参考：https://lucene.apache.org/solr/guide/6_6/updatehandlers-in-solrconfig.html

## 3.AutoCommit
参考：http://wiki.apache.org/solr/SolrConfigXml
例如，我的配置文件在 E:\solr-6.5.0\server\solr\mysqlCore\conf\solrconfig.xml 中已有默认配置，如下：
``` xml
<!-- The default high-performance update handler -->
  <updateHandler class="solr.DirectUpdateHandler2">

    <!-- Enables a transaction log, used for real-time get, durability, and
         and solr cloud replica recovery.  The log can grow as big as
         uncommitted changes to the index, so use of a hard autoCommit
         is recommended (see below).
         "dir" - the target directory for transaction logs, defaults to the
                solr data directory.
         "numVersionBuckets" - sets the number of buckets used to keep
                track of max version values when checking for re-ordered
                updates; increase this value to reduce the cost of
                synchronizing access to version buckets during high-volume
                indexing, this requires 8 bytes (long) * numVersionBuckets
                of heap space per Solr core.
    -->
    <updateLog>
      <str name="dir">${solr.ulog.dir:}</str>
      <int name="numVersionBuckets">${solr.ulog.numVersionBuckets:65536}</int>
    </updateLog>

    <!-- AutoCommit

         Perform a hard commit automatically under certain conditions.
         Instead of enabling autoCommit, consider using "commitWithin"
         when adding documents. 

         http://wiki.apache.org/solr/UpdateXmlMessages

         maxDocs - Maximum number of documents to add since the last
                   commit before automatically triggering a new commit.

         maxTime - Maximum amount of time in ms that is allowed to pass
                   since a document was added before automatically
                   triggering a new commit. 
         openSearcher - if false, the commit causes recent index changes
           to be flushed to stable storage, but does not cause a new
           searcher to be opened to make those changes visible.

         If the updateLog is enabled, then it's highly recommended to
         have some sort of hard autoCommit to limit the log size.
      -->
    <autoCommit>
      <maxTime>${solr.autoCommit.maxTime:15000}</maxTime>
      <openSearcher>false</openSearcher>
    </autoCommit>

    <!-- softAutoCommit is like autoCommit except it causes a
         'soft' commit which only ensures that changes are visible
         but does not ensure that data is synced to disk.  This is
         faster and more near-realtime friendly than a hard commit.
      -->

    <autoSoftCommit>
      <maxTime>${solr.autoSoftCommit.maxTime:-1}</maxTime>
    </autoSoftCommit>

    <!-- Update Related Event Listeners
         
         Various IndexWriter related events can trigger Listeners to
         take actions.

         postCommit - fired after every commit or optimize command
         postOptimize - fired after every optimize command
      -->
    <!-- The RunExecutableListener executes an external command from a
         hook such as postCommit or postOptimize.
         
         exe - the name of the executable to run
         dir - dir to use as the current working directory. (default=".")
         wait - the calling thread waits until the executable returns. 
                (default="true")
         args - the arguments to pass to the program.  (default is none)
         env - environment variables to set.  (default is none)
      -->
    <!-- This example shows how RunExecutableListener could be used
         with the script based replication...
         http://wiki.apache.org/solr/CollectionDistribution
      -->
    <!--
       <listener event="postCommit" class="solr.RunExecutableListener">
         <str name="exe">solr/bin/snapshooter</str>
         <str name="dir">.</str>
         <bool name="wait">true</bool>
         <arr name="args"> <str>arg1</str> <str>arg2</str> </arr>
         <arr name="env"> <str>MYVAR=val1</str> </arr>
       </listener>
      -->

  </updateHandler>
```

### 1.硬提交是提交数据持久化到磁盘里面。不过，比较耗费机器资源，这样即使jvm崩溃或者宕机，也不影响这部分索引。
openSearcher: true  索引在searcher中可见, 但是也需要重新打开searcher才行。
openSearcher: false 索引在searcher中不可见。

按照下面的配置就会使用solr的自动硬提交功能，就不用我们自己手动提交。
``` xml
<autoCommit>
    <!--表示软提交达到1万条的时候会自动进行一次硬提交-->
    <maxDocs>10000</maxDocs>

    <!--表示软提交10秒之后会执行一次硬提交-->
    <maxTime>10000</maxTime>

    <!--true表示每一次硬提交都开启一个新的Searcher-->
    <openSearcher>false</openSearcher>
</autoCommit>
```

### 2.软提交是提交数据到内存里面，并没有持久化到磁盘，但是他会把提交的记录写到tlog的日志文件里面所以如果jvm崩溃，那这部分索引就没了

下面就是自动软提交的配置，不需要自己维护提交（默认没有开启）：
``` xml
<autoSoftCommit> 
  <!--5秒执行一次软提交-->
  <maxTime>5000</maxTime> 
</autoSoftCommit>
```
