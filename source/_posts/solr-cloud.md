---
title: solr 集群的创建
date: 2018-03-03 16:38:59
tags: solr
categories: bigdata
---
solr 集群，即 solrcloud 的创建
下面说说solr集群的安装使用，同样是使用前面例子使用的 solr6.5.0 版本，我在一台机器上安装，所以这是伪集群，当修改为真集群的时候，只要将IP地址修改下即可，下面会说明。

首先，你得搭建 zookeeper 集群，可以参考 [zookeeper集群版安装方法][zookeeper-cluster]

### 下面开始创建solr集群：
创建一个目录用于存放集群使用到的所有实例和配置信息
``` bash
$ cd /home/hewentian/ProjectD
$ mkdir solrCloud		# 存放集群的所有文件，包括：三个solr实例、和实例共享的配置文件solr-configs
$ mkdir solrCloud/solr-configs	# 用于存放集群的core的配置信息，会被上传到zookeeper
```
将一个SOLR压缩包放到这个目录，我之前已在Downloads目录下载好了
``` bash
$ cp /home/hewentian/Downloads/solr-6.5.0.tgz ./
$ tar xzvf solr-6.5.0.tgz
$ mv solr-6.5.0 solr1	# 为方便起见，这里将其重命名为solr1，先将solr1配置好，后面会将其复制为solr2, solr3

$ ls
solr1  solr-6.5.0.tgz  solr-configs
```
我们仍然使用之前的例子，演示使用DIH方式将数据导入到SOLR，只不过，这次是集群的方式，所以需要将`data_driven_schema_configs/conf`的配置信息复制到`solr-configs`中，并将其命名为`mysqlCore`
``` bash
$ pwd
/home/hewentian/ProjectD/solrCloud

$ cp -r solr1/server/solr/configsets/data_driven_schema_configs/conf/ solr-configs/mysqlCore/
```
将mysql-connector-java-5.1.25.jar放到`solr1/server/lib/ext`目录下面，并将上一个例子，[solr的安装使用][solr-standalone]的那三个文件: solrconfig.xml、db-data-config.xml、managed-schema复制到`solr-configs/mysqlCore/`下，override存在的。

接下来还有2个文件需要修改：
修改文件一：solr.in.sh
``` bash
$ vi solr1/bin/solr.in.sh

# 修改以下内容即可
ZK_HOST="127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183"
ZK_CLIENT_TIMEOUT="15000"
SOLR_PORT=8983
```
修改文件二：solr.xml
``` bash
$ vi solr1/server/solr/solr.xml

# 修改下面的内容即可
<str name="host">${host:127.0.0.1}</str>   	# 当修改为真集群的时候，只修改这个IP地址即可
<int name="hostPort">${jetty.port:8983}</int>	# 这里端口为 8983
```

这样一个实例就配置好了，将其复制为solr2, solr3
``` bash
$ cp -r solr1 solr2
$ cp -r solr1 solr3
```
修改solr2, solr3的端口为8984, 8985
``` bash
$ vi solr2/bin/solr.in.sh

# 修改以下内容即可
SOLR_PORT=8984

$ vi solr3/bin/solr.in.sh

# 修改以下内容即可
SOLR_PORT=8985

$ vi solr2/server/solr/solr.xml

# 修改下面的内容即可
<str name="host">${host:127.0.0.1}</str>   	# 当修改为真集群的时候，只修改这个IP地址即可
<int name="hostPort">${jetty.port:8984}</int>	# 这里端口为 8984

$ vi solr3/server/solr/solr.xml

# 修改下面的内容即可
<str name="host">${host:127.0.0.1}</str>   	# 当修改为真集群的时候，只修改这个IP地址即可
<int name="hostPort">${jetty.port:8985}</int>	# 这里端口为 8985

```

这样，三个solr实例就已经配置好了，下面将其都启动
``` bash
$ pwd
/home/hewentian/ProjectD/solrCloud

$ solr1/bin/solr start
Waiting up to 180 seconds to see Solr running on port 8983 [\]  
Started Solr server on port 8983 (pid=15292). Happy searching!

$ solr2/bin/solr start
Waiting up to 180 seconds to see Solr running on port 8984 [\]  
Started Solr server on port 8984 (pid=15507). Happy searching!

$ solr3/bin/solr start
Waiting up to 180 seconds to see Solr running on port 8985 [\]  
Started Solr server on port 8985 (pid=15706). Happy searching!
```

在浏览器中可以如下访问：
http://localhost:8983
http://localhost:8984
http://localhost:8985

下面，我们将`solr-configs/mysqlCore`的内容上传到zookeeper，以便让三台solr都使用这个配置
``` bash
$ /home/hewentian/ProjectD/solrCloud/solr1/server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183 -cmd upconfig -confdir /home/hewentian/ProjectD/solrCloud/solr-configs/mysqlCore -confname mysqlCore
```
#### 说明：

1. -confdir：这个指的是 本地上传的文件位置    
2. -confname：上传后在zookeeper中的节点名称

也可以上传单个文件，单个文件上传先要删除，不然会报错：
``` bash
$ /home/hewentian/ProjectD/solrCloud/solr1/server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183 -cmd putfile /configs/mysqlCore/solrconfig.xml /home/hewentian/ProjectD/solrCloud/solr-configs/mysqlCore/solrconfig.xml
```
#### 说明: 
putfile 后第一个`/configs/mysqlCore/solrconfig.xml`指的是`zookeeper`中的配置文件，第二个`/home/hewentian/ProjectD/solrCloud/solr-configs/mysqlCore/solrconfig.xml`指的是本地文件路径


下面我们在solr1中创建一个core，并命名为 mysqlCore，在浏览器中输入如下url即可，它有3个分片，每个分片有2个副本：
http://localhost:8983/solr/admin/collections?action=CREATE&name=mysqlCore&numShards=3&replicationFactor=2&maxShardsPerNode=3&collection.configName=mysqlCore
执行成功后，你可能会看到如下输出：
``` xml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<response>
    <lst name="responseHeader">
        <int name="status">0</int>
        <int name="QTime">16020</int>
    </lst>
    <lst name="success">
        <lst name="127.0.0.1:8983_solr">
            <lst name="responseHeader">
                <int name="status">0</int>
                <int name="QTime">13930</int>
            </lst>
            <str name="core">mysqlCore_shard3_replica2</str>
        </lst>
        <lst name="127.0.0.1:8983_solr">
            <lst name="responseHeader">
                <int name="status">0</int>
                <int name="QTime">14273</int>
            </lst>
            <str name="core">mysqlCore_shard2_replica1</str>
        </lst>
        <lst name="127.0.0.1:8984_solr">
            <lst name="responseHeader">
                <int name="status">0</int>
                <int name="QTime">14479</int>
            </lst>
            <str name="core">mysqlCore_shard3_replica1</str>
        </lst>
        <lst name="127.0.0.1:8985_solr">
            <lst name="responseHeader">
                <int name="status">0</int>
                <int name="QTime">14597</int>
            </lst>
            <str name="core">mysqlCore_shard2_replica2</str>
        </lst>
        <lst name="127.0.0.1:8985_solr">
            <lst name="responseHeader">
                <int name="status">0</int>
                <int name="QTime">14748</int>
            </lst>
            <str name="core">mysqlCore_shard1_replica1</str>
        </lst>
        <lst name="127.0.0.1:8984_solr">
            <lst name="responseHeader">
                <int name="status">0</int>
                <int name="QTime">14778</int>
            </lst>
            <str name="core">mysqlCore_shard1_replica2</str>
        </lst>
    </lst>
</response>
```

也可以使用下面的方式，指定在哪台机器中创建哪个分片的哪个副本。
``` bash
$ curl "http://localhost:8983/solr/admin/cores?action=CREATE&name=mysqlCore_shard1_replica1&instanceDir=mysqlCore_shard1_replica1&collection=mysqlCore&shard=mysqlCore_shard1"

$ curl "http://localhost:8983/solr/admin/cores?action=CREATE&name=mysqlCore_shard3_replica2&instanceDir=mysqlCore_shard3_replica2&collection=mysqlCore&shard=mysqlCore_shard3"


$ curl "http://localhost:8984/solr/admin/cores?action=CREATE&name=mysqlCore_shard1_replica2&instanceDir=mysqlCore_shard1_replica2&collection=mysqlCore&shard=mysqlCore_shard1"

$ curl "http://localhost:8984/solr/admin/cores?action=CREATE&name=mysqlCore_shard2_replica1&instanceDir=mysqlCore_shard2_replica1&collection=mysqlCore&shard=mysqlCore_shard2"


$ curl "http://localhost:8985/solr/admin/cores?action=CREATE&name=mysqlCore_shard2_replica2&instanceDir=mysqlCore_shard2_replica2&collection=mysqlCore&shard=mysqlCore_shard2"

$ curl "http://localhost:8985/solr/admin/cores?action=CREATE&name=mysqlCore_shard3_replica1&instanceDir=mysqlCore_shard3_replica1&collection=mysqlCore&shard=mysqlCore_shard3"
```

你也可以查看分片是否创建成功：
``` bash
$ ls solr1/server/solr
configsets  mysqlCore_shard2_replica1  mysqlCore_shard3_replica2  README.txt  solr.xml  zoo.cfg

$ ls solr2/server/solr
configsets  mysqlCore_shard1_replica2  mysqlCore_shard3_replica1  README.txt  solr.xml  zoo.cfg

$ ls solr3/server/solr
configsets  mysqlCore_shard1_replica1  mysqlCore_shard2_replica2  README.txt  solr.xml  zoo.cfg
```

在[http://localhost:8983](http://localhost:8983)上面执行DIH将user索引进solr，成功之后你就可以在[http://localhost:8984](http://localhost:8984), [http://localhost:8985](http://localhost:8985)上面看到同样的结果。

当然，我们也可以在启动的时候，才指定zookeeper
``` bash
$ ./bin/solr start -c -z zk1:port,zk2:port,zk3:port
```
说明：

1. -c：以solr_cloud的方式启动；
2. -z:指定zookeeper集群的地址和端口，上面搭建zookeeper集群时的3台机器

### solr备份
在进行solr备份的时候，一定要先将solr服务停了，然后再备份。否则在还原的时候有可能会产生`write.lock`，在产生`write.lock`的时候也不要怕，你可以按下面的方法解决：

1. 将`write.lock`删掉，然后再新建一个，然后修改权限；
``` bash
$ rm write.lock
$ touch write.lock
$ chmod 666 write.lock
```

2. 重启solr服务；
3. 在浏览器登录到那个solr的cord，reload一下，然后查询，如果没报错的话，可能要等5分钟左右，查询结果就出来了。
[solr界面]->[collections]->[点击你的那个 collection]->[Reload]


JAVA示例代码如下：
``` java
CloudSolrClient cloudSolrClient = new CloudSolrClient.Builder()
				.withZkHost("127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183").build();
		cloudSolrClient.setDefaultCollection("mysqlCore");

		SolrQuery solrQuery = new SolrQuery();

		solrQuery.setQuery("schools:\"北京第一中学\" AND id:[1 TO 3]");

		// 分页
		solrQuery.setStart(0);
		solrQuery.setRows(10);

		SolrDocumentList results = cloudSolrClient.query(solrQuery).getResults();

		logger.info("numFound: " + results.getNumFound());
		if (CollectionUtils.isEmpty(results)) {
			return;
		}

		results.stream().forEach(logger::info);
	}
```

未完，待续……

[zookeeper-cluster]: ../../../../2017/12/06/zookeeper-cluster/
[solr-standalone]: ../../../../2017/10/28/solr-standalone/