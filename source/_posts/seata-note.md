---
title: seata 学习笔记
date: 2021-11-28 17:45:55
tags: seata
categories: java
---

### 安装
安装过程参考
https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html

Server端存储模式（store.mode）现有file、db、redis三种（后续将引入raft,mongodb），file模式无需改动，直接启动即可，下面专门讲下db模式启动步骤。
注： 
1. file模式为单机模式，全局事务会话信息在内存中读写并持久化到本地文件root.data，性能较高;
2. db模式为高可用模式，全局事务会话信息通过db共享，相应性能差些;
3. redis模式Seata-Server 1.3及以上版本支持，性能较高，存在事务信息丢失风险，请提前配置合适当前场景的redis持久化配置。


#### 第一步：首先，到如下地址，下载最新版本，截止目前，最新版本为`seata-server-1.4.2.zip`
https://seata.io/en-us/blog/download.html

解压

        unzip -O CP936 seata-server-1.4.2.zip


#### 第二步：建表（仅db模式）
建表语句，可以在下面的地址找到（注意版本号）
https://github.com/seata/seata/blob/1.4.2/script/server/db/mysql.sql

全局事务会话信息由3块内容构成，全局事务-->分支事务-->全局锁，对应表global_table、branch_table、lock_table


#### 第三步，方式一：注册到nacos，配置信息存放在file
打开文件`{SEATA_HOME}/conf/file.conf`，主要修改部分，如下：
``` conf
service {
  # transaction service group mapping
  vgroupMapping.hwt_tx_group = "default" # 修改事务组名称为：hwt_tx_group，和客户端自定义的名称对应
  # only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  # degrade, current not support
  enableDegrade = false
  # disable seata
  disableGlobalTransaction = false
}

store {
  mode = "db"

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.cj.jdbc.Driver"
    ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
    url = "jdbc:mysql://127.0.0.1:3306/seata?rewriteBatchedStatements=true"
    user = "mysql"
    password = "mysql"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
}
```

打开文件`{SEATA_HOME}/conf/registry.conf`，主要修改部分，如下：
``` conf
registry {
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "DEFAULT_GROUP"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
}

config {
  type = "file"

  file {
    name = "file.conf"
  }
}
```


#### 第三步，方式二：注册到nacos，配置信息存放在nacos
打开文件`{SEATA_HOME}/conf/registry.conf`，主要修改部分，如下：
``` conf
registry {
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "DEFAULT_GROUP"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
}

config {
  type = "nacos"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    group = "DEFAULT_GROUP"
    username = "nacos"
    password = "nacos"
    dataId = "seataServer.properties"
  }
}
```


接着，在nacos新建配置
Data Id: seataServer.properties
Group: DEFAULT_GROUP
配置格式： Properties

``` properties
transport.type=TCP
transport.server=NIO
transport.heartbeat=true
transport.enableClientBatchSendRequest=false
transport.threadFactory.bossThreadPrefix=NettyBoss
transport.threadFactory.workerThreadPrefix=NettyServerNIOWorker
transport.threadFactory.serverExecutorThreadPrefix=NettyServerBizHandler
transport.threadFactory.shareBossWorker=false
transport.threadFactory.clientSelectorThreadPrefix=NettyClientSelector
transport.threadFactory.clientSelectorThreadSize=1
transport.threadFactory.clientWorkerThreadPrefix=NettyClientWorkerThread
transport.threadFactory.bossThreadSize=1
transport.threadFactory.workerThreadSize=default
transport.shutdown.wait=3
service.vgroupMapping.hwt_tx_group=default
service.default.grouplist=127.0.0.1:8091
service.enableDegrade=false
service.disableGlobalTransaction=false
client.rm.asyncCommitBufferLimit=10000
client.rm.lock.retryInterval=10
client.rm.lock.retryTimes=30
client.rm.lock.retryPolicyBranchRollbackOnConflict=true
client.rm.reportRetryCount=5
client.rm.tableMetaCheckEnable=false
client.rm.tableMetaCheckerInterval=60000
client.rm.sqlParserType=druid
client.rm.reportSuccessEnable=false
client.rm.sagaBranchRegisterEnable=false
client.tm.commitRetryCount=5
client.tm.rollbackRetryCount=5
client.tm.defaultGlobalTransactionTimeout=60000
client.tm.degradeCheck=false
client.tm.degradeCheckAllowTimes=10
client.tm.degradeCheckPeriod=2000
store.mode=db
store.publicKey=
store.file.dir=file_store/data
store.file.maxBranchSessionSize=16384
store.file.maxGlobalSessionSize=512
store.file.fileWriteBufferCacheSize=16384
store.file.flushDiskMode=async
store.file.sessionReloadReadSize=100
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://mysql.hewentian.com:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=123456
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
store.redis.mode=single
store.redis.single.host=127.0.0.1
store.redis.single.port=6379
store.redis.sentinel.masterName=
store.redis.sentinel.sentinelHosts=
store.redis.maxConn=10
store.redis.minConn=1
store.redis.maxTotal=100
store.redis.database=0
store.redis.password=
store.redis.queryLimit=100
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
client.undo.dataValidation=true
client.undo.logSerialization=jackson
client.undo.onlyCareUpdateColumns=true
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
client.undo.logTable=undo_log
client.undo.compress.enable=true
client.undo.compress.type=zip
client.undo.compress.threshold=64k
log.exceptionRate=100
transport.serialization=seata
transport.compressor=none
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```

配置的内容可以从下面的文件获取（注意版本号）
https://github.com/seata/seata/blob/1.4.2/script/config-center/config.txt

主要修改点，如下：
``` properties
service.vgroupMapping.hwt_tx_group=default
store.mode=db
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://mysql.hewentian.com:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=123456
```


#### 第四步：启动
启动命令如下：

        bash {SEATA_HOME}/bin/seata-server.sh -h 127.0.0.1 -p 8091 -m db -n 1


参数说明：

        -h: 注册到注册中心的ip
        -p: Server rpc 监听端口
        -m: 全局事务会话信息存储模式，file、db、redis，优先读取启动参数 (Seata-Server 1.3及以上版本支持redis)
        -n: Server node，多个Server时，需区分各自节点，用于生成不同区间的transactionId，以免冲突


