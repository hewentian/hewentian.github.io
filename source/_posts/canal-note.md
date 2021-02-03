---
title: canal 学习笔记
date: 2021-01-22 10:16:21
tags: mysql
categories: java
---

本篇将说说阿里巴巴开源的mysql数据库同步工具[canal][link_id_canal_github]的使用，详细说明可以参考[wiki][link_id_canal_wiki]。主要用途是基于mysql数据库增量日志解析，提供增量数据订阅和消费。

基于日志增量订阅和消费的业务包括
* 数据库镜像
* 数据库实时备份
* 索引构建和实时维护（拆分异构索引、倒排索引等）
* 业务 cache 刷新
* 带业务逻辑的增量数据处理

当前的 canal 支持源端 mysql 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x


### 工作原理
mysql主备复制原理

![](/img/mysql-1.png "mysql主备复制原理")

* mysql master 将数据变更写入二进制日志（binary log，其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看）
* mysql slave 将 master 的 binary log events 复制到它的中继日志(relay log)
* mysql slave 重放 relay log 中事件，将数据变更反映它自己的数据


canal工作原理

![](/img/canal-1.png "canal工作原理")

* canal 模拟 mysql slave 的交互协议，伪装自己为 mysql slave，向 mysql master 发送 dump 请求
* mysql master 收到 dump 请求，开始推送 binary log 给 slave （即 canal）
* canal 解析 binary log 对象（原始为 byte 流）


### 配置mysql服务器
在文件`/etc/mysql/conf.d/mysql.cnf`中增加如下配置信息：
``` mysql
[mysqld]
log_bin=mysql-bin    # 开启 binlog，其中 mysql-bin 是日志名称前缀
binlog_format=ROW    # 选择 ROW 模式
server_id=1          # 默认值是0，如果使用默认值则不能和从节点通信，这个值的区间是：1到(2^32)-1。注意不要和 canal 的 slaveId 重复
```

配置后，重启mysql服务器，验证是否配置成功：

        SHOW VARIABLES LIKE 'server_id';
        SHOW VARIABLES LIKE 'log_%';
        SHOW VARIABLES LIKE 'binlog_format';

创建 canal 链接 mysql 的账号，并分配作为 mysql slave 的权限，如果已有账户则可直接 grant

        CREATE USER canal IDENTIFIED BY 'canal';
        GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
        -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%';
        FLUSH PRIVILEGES;


### 安装canal
下载，可以在[releases][link_id_canal_releases]页面进行下载
``` bash
$ wget https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.deployer-1.1.4.tar.gz

$ mkdir ~/canal
$ tar xf canal.deployer-1.1.4.tar.gz -C ~/canal

$ cd ~/canal
$ ls
bin  conf  lib  logs
```

配置canal server
配置端口、用户名和访问密码
``` bash
$ vi ~/canal/conf/canal.properties

canal.port = 11111
canal.user = canal
canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458    # 这个是加密后的密码，未加密前是canal
canal.destinations = example                               # 当前默认开启了一个名为example的instance，多个之间用逗号(,)分隔
```

配置canal instance
``` bash
$ vi ~/canal/conf/example/instance.properties

canal.instance.mysql.slaveId=1234    # mysql serverId , v1.0.26+ will autoGen
canal.instance.master.address=127.0.0.1:3306
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
canal.instance.filter.regex=.*\\..*
```

启动canal
``` bash
$ cd ~/canal
$ sh bin/startup.sh
```

查看 server 日志
``` bash
$ more logs/canal/canal.log

2021-01-22 20:12:11.735 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
2021-01-22 20:12:11.797 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
2021-01-22 20:12:11.813 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2021-01-22 20:12:11.869 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[172.18.0.1(172.18.0.1):11111]
2021-01-22 20:12:13.779 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......
```

查看 instance 的日志
``` bash
$ more logs/example/example.log

2021-01-22 20:12:12.428 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2021-01-22 20:12:12.436 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
2021-01-22 20:12:12.722 [main] WARN  o.s.beans.GenericTypeAwarePropertyDescriptor - Invalid JavaBean property 'connectionCharset' being accessed! Ambiguous write methods found next to actually used [public void com.alibaba.otter.canal.parse.inbound.mysql.AbstractMysqlEventParser.setConnectionCharset(java.nio.charset.Charset)]: [public void com.alibaba.otter.canal.parse.inbound.mysql.AbstractMysqlEventParser.setConnectionCharset(java.lang.String)]
2021-01-22 20:12:12.803 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2021-01-22 20:12:12.803 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
2021-01-22 20:12:13.524 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example
2021-01-22 20:12:13.536 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^.*\..*$
2021-01-22 20:12:13.536 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter :
2021-01-22 20:12:13.570 [main] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - start successful....
2021-01-22 20:12:13.801 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
2021-01-22 20:12:13.801 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just show master status
2021-01-22 20:12:15.240 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> find start position successfully, EntryPosition[included=false,journalName=mysql-bin.000003,position=4,serverId=1,gtid=<null>,timestamp=1591628633000] cost : 1408ms , the next step is binlog dump
```

关闭
``` bash
$ cd ~/canal
$ sh bin/stop.sh
```


### JAVA使用示例
在JAVA项目中的pom.xml加入依赖
``` xml
<dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.client</artifactId>
    <version>1.1.4</version>
</dependency>
```

JAVA代码如下
``` java
package com.hewentian.canal;

import java.net.InetSocketAddress;
import java.util.Date;
import java.util.List;

import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.protocol.Message;
import com.alibaba.otter.canal.protocol.CanalEntry.Column;
import com.alibaba.otter.canal.protocol.CanalEntry.Entry;
import com.alibaba.otter.canal.protocol.CanalEntry.EntryType;
import com.alibaba.otter.canal.protocol.CanalEntry.EventType;
import com.alibaba.otter.canal.protocol.CanalEntry.RowChange;
import com.alibaba.otter.canal.protocol.CanalEntry.RowData;

public class SimpleCanalClientExample {

    public static void main(String args[]) {
        // 创建链接
        CanalConnector connector = CanalConnectors.newSingleConnector(
                new InetSocketAddress("192.168.56.113", 11111),
                "example", "canal", "canal");

        int batchSize = 1000;
        int emptyCount = 0;

        try {
            connector.connect();

            //订阅 监控的 数据库.表
//            connector.subscribe("test.t_user");
            connector.subscribe(".*\\..*");
            connector.rollback();
            int totalEmptyCount = 100;

            while (emptyCount < totalEmptyCount) {
                Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据
                long batchId = message.getId();
                int size = message.getEntries().size();
                System.out.println("batchId: " + batchId);

                if (batchId == -1 || size == 0) {
                    emptyCount++;
                    System.out.println("empty count: " + emptyCount);
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                    }
                } else {
                    emptyCount = 0;
                    // System.out.printf("message[batchId=%s,size=%s] \n", batchId, size);
                    printEntry(message.getEntries());
                }

                connector.ack(batchId); // 提交确认
                // connector.rollback(batchId); // 处理失败, 回滚数据
            }

            System.out.println("empty too many times, exit");
        } finally {
            connector.disconnect();
        }
    }

    private static void printEntry(List<Entry> entrys) {
        for (Entry entry : entrys) {
            if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {
                continue;
            }

            RowChange rowChage;
            try {
                rowChage = RowChange.parseFrom(entry.getStoreValue());
            } catch (Exception e) {
                throw new RuntimeException("ERROR ## parser of eromanga-event has an error, data:" + entry.toString(), e);
            }

            EventType eventType = rowChage.getEventType();
            long delayTime = new Date().getTime() - entry.getHeader().getExecuteTime();
            System.out.println(String.format("================ binlog[%s:%s], name[%s,%s], eventType: %s, delayTime: %s",
                    entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                    entry.getHeader().getSchemaName(), entry.getHeader().getTableName(),
                    eventType, delayTime));

            // DDL数据，打印SQL
            if (eventType == EventType.QUERY || rowChage.getIsDdl()) {
                System.out.println("sql -----> " + rowChage.getSql());
            }

            // DML数据，打印字段信息
            for (RowData rowData : rowChage.getRowDatasList()) {
                if (eventType == EventType.DELETE) {
                    printColumn(rowData.getBeforeColumnsList());
                } else if (eventType == EventType.INSERT) {
                    printColumn(rowData.getAfterColumnsList());
                } else {
                    System.out.println("---------- before");
                    printColumn(rowData.getBeforeColumnsList());
                    System.out.println("---------- after");
                    printColumn(rowData.getAfterColumnsList());
                }
            }
        }
    }

    private static void printColumn(List<Column> columns) {
        for (Column column : columns) {
            System.out.println(column.getName() + " : " + column.getValue() + ", update = " + column.getUpdated());
        }
    }

}
```

运行上面的JAVA代码后，可以从控制台看到类似消息：
``` java
empty count: 1
empty count: 2
empty count: 3
empty count: 4
```

此时代表当前数据库无变更数据


触发数据库变更
``` sql
use test;

CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(20) DEFAULT NULL COMMENT '用户名',
 PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC COMMENT='用户表';

INSERT INTO t_user(id,name) VALUE(1,'Scott');
UPDATE t_user SET name = 'Tiger' WHERE id = 1;
DELETE FROM t_user WHERE id = 1;
```

可以从控制台中看到：
``` java
empty count: 1
empty count: 2
empty count: 3
empty count: 4
empty count: 5
empty count: 6
================ binlog[mysql-bin.000003:2626], name[test,t_user], eventType: INSERT
id : 1, update = true
name : Scott, update = true
empty count: 1
empty count: 2
empty count: 3
empty count: 4
empty count: 5
empty count: 6
empty count: 7
empty count: 8
================ binlog[mysql-bin.000003:2892], name[test,t_user], eventType: UPDATE
---------- before
id : 1, update = false
name : Scott, update = false
---------- after
id : 1, update = false
name : Tiger, update = true
empty count: 1
empty count: 2
empty count: 3
empty count: 4
empty count: 5
empty count: 6
================ binlog[mysql-bin.000003:3170], name[test,t_user], eventType: DELETE
id : 1, update = false
name : Tiger, update = false
empty count: 1
empty count: 2
empty count: 3
empty count: 4
empty count: 5
empty count: 6
empty count: 7
empty count: 8
```


完整代码在[这里][link_id_canal_github_my]。

[link_id_canal_github]: https://github.com/alibaba/canal
[link_id_canal_wiki]: https://github.com/alibaba/canal/wiki
[link_id_canal_releases]: https://github.com/alibaba/canal/releases
[link_id_canal_github_my]: https://github.com/hewentian/study-lib/tree/main/codes/canal

