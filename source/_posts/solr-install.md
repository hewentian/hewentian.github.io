---
title: solr的安装使用
date: 2017-10-28 10:57:27
tags: solr
categories: solr
---

下面说说我的solr学习笔记之 solr6.5.0 + jetty + mysql 使用 DIH 搭建 solr 服务

参考：https://wiki.apache.org/solr/DIHQuickStart

首先下载solr，下载地址为：http://lucene.apache.org/solr/

截至写该笔记时，solr的版本为6.5.0，下载后，将其解压到任意一个目录。当然，目录路径不要有中文。

我解压到`/home/hewentian/ProjectD/solr-6.5.0`，目录结构如下：
bin
contrib
dist
docs
example
licenses
server
CHANGES.txt
LICENSE.txt
LUCENE_CHANGES.txt
NOTICE.txt
README.txt

solr的软件包，默认使用jetty部署，SOLR6要求JDK8，启动solr的命令很简单：
在DOS窗口中命令如下：
``` bash
$ /home/hewentian/ProjectD/solr-6.5.0/bin/solr start
```
然后在浏览器中可以访问：
http://localhost:8983/solr


在server/solr目录下，有一个README.txt的文件，其中说明了如何建立solr core。
最简单的建立是直接复制solr中为我们提供好的例子，打开`server/solr/configsets`目录会发现里面已经有三个例子，如下：
basic_configs
data_driven_schema_configs
sample_techproducts_configs

因为我们是要从数据库导入数据，所以选择以 “data_driven_schema_configs” 这个例子为模板创建我们的core: mysqlCore。
在DOS窗口中命令如下：
``` bash
$ /home/hewentian/ProjectD/solr-6.5.0/bin/solr create -c mysqlCore -d data_driven_schema_configs
```

说明：
-c: 指定创建的core为mysqlCore
-d：指定以data_driven_schema_configs为模板

执行上面的命令后在`/home/hewentian/ProjectD/solr-6.5.0/server/solr`目录下会有一个文件夹mysqlCore

然后在浏览器中可以访问：
http://localhost:8983/solr

你会在[Core Selector]中看到mysqlCore，但是目前还不能使用DIH导入数据，需下面的设置。


建数据表，并插入3条记录，如下：
``` sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/`test` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `test`;

/*Table structure for table `user` */

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` varchar(255) NOT NULL COMMENT '用户id',
  `name_` varchar(255) DEFAULT NULL COMMENT '名字',
  `age_` int(11) DEFAULT NULL COMMENT '年龄',
  `ability_` varchar(100) DEFAULT NULL COMMENT '资格；等级',
  `address_` varchar(255) DEFAULT NULL COMMENT '详细地址',
  `school1_` varchar(255) DEFAULT NULL COMMENT '学校1',
  `school2_` varchar(255) DEFAULT NULL COMMENT '学校2',
  `school3_` varchar(255) DEFAULT NULL COMMENT '学校3',
  `update_time_` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表';

/*Data for the table `user` */

insert  into `user`(`id`,`name_`,`age_`,`ability_`,`address_`,`school1_`,`school2_`,`school3_`,`update_time_`) 
values 
('1','张三',20,'初级工程师','广东',NULL,NULL,NULL,'2017-10-28 11:39:52'),
('2','李四',20,'中级工程师','湖南','湖南实验小学',NULL,'湖南大学','2017-10-28 11:39:51'),
('3','王五',40,'高级工程师','北京','北京实验小学','北京第一中学','北京大学','2017-10-28 11:39:53');
```

为了使用mysql导入数据，我们还需要导入两个重要的jar包。由于mysql的jar包并没有在项目中，于是我复制了一份
放到了`server/lib/ext`目录下面了。我加的是mysql-connector-java-5.1.25.jar，如果你的是另外的版本，要修改下
面的lib中正则的部分。另外两个需要的jar包就是dist目录下带有“dataimport”标识的jar包。
然后打开mysqlCore/conf/solrconfig.xml，引用上面提到的jar包，如下。
``` xml
<lib dir="${solr.install.dir:../../../..}/contrib/extraction/lib" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-cell-\d.*\.jar" />

<lib dir="${solr.install.dir:../../../..}/contrib/clustering/lib/" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-clustering-\d.*\.jar" />

<lib dir="${solr.install.dir:../../../..}/contrib/langid/lib/" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-langid-\d.*\.jar" />

<lib dir="${solr.install.dir:../../../..}/contrib/velocity/lib" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-velocity-\d.*\.jar" />
  
<!-- 上面的lib为原本有的，下面的lib为添加的 -->
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />
<lib dir="${solr.install.dir:../../../..}/server/lib/ext/" regex="mysql-connector-java-5.1.25.jar" />
```

添加requestHandler
在mysqlCore/conf/solrconfig.xml，搜索`<requestHandler name="/select" class="solr.SearchHandler">`
然后在该行之上添加如下代码。
``` xml
<requestHandler name="/dataimport" class="solr.DataImportHandler">
	<lst name="defaults">
      <str name="config">db-data-config.xml</str>
    </lst>
</requestHandler>
```

在mysqlCore/conf目录下新建db-data-config.xml, 内容如下。
``` xml
<dataConfig>
	<dataSource name="ds" type="JdbcDataSource" driver="com.mysql.jdbc.Driver"
		url="jdbc:mysql://localhost:3306/test" user="root" password="root" />
	<document>
		<entity name="user" pk="id" dataSource="ds" query="select * from user"
			deltaQuery="select id from user where update_time_>'${dataimporter.last_index_time}'"
			deltaImportQuery="select t.* from user t where t.id='${dataimporter.delta.id}'">
			<field name="id" column="id" />
			<field name="name" column="name_" />
			<field name="age" column="age_" />
			<field name="ability" column="ability_" />
			<field name="address" column="address_" />

			<!-- 将DB中的多个列，在solr中以数组形式存储到一个field中 -->
			<field name="schools" column="school1_" />
			<field name="schools" column="school2_" />
			<field name="schools" column="school3_" />

			<field name="update_time" column="update_time_" />
		</entity>
	</document>
</dataConfig>
```
说明：
	dataSource是数据库数据源。Entity就是一张表对应的实体，pk是主键，query是查询语句。
	Field对应一个字段，column是数据库里的column名，后面的name属性对应着Solr的Filed的名字。
	其中test是数据库名，user是表名。
  
在mysqlCore/conf/managed-schema配置field信息，其中id的field本身已有。默认的 filed 不要删。
``` xml
<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />
<field name="name" type="string" indexed="true" stored="true"/>
<field name="age" type="int" indexed="true" stored="true"/>
<field name="ability" type="string" indexed="true" stored="true"/>
<field name="address" type="string" indexed="true" stored="true"/>
<field name="schools" type="text_general" multiValued="true" indexed="true" stored="true"/>
<field name="update_time" type="date" multiValued="false" indexed="true" stored="true"/>
```
由于修改了配置文件，所以要重启solr，命令如下：
``` bash
$ /home/hewentian/ProjectD/solr-6.5.0/bin/solr stop -all
$ /home/hewentian/ProjectD/solr-6.5.0/bin/solr start
```
然后在浏览器中可以访问：
http://localhost:8983/solr

点击[Core Selector]->mysqlCore->Dataimport在新的页面中按如下选择：
Command: full-import
Entity:user
Auto-Refresh Status
然后点[Execute]

成功后，点[Query]即可查询。
