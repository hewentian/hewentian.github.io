---
title: solr配置IKAnalyzer的中文分词
date: 2018-01-08 16:07:17
tags: solr
categories: bigdata
---

配置IKAnalyzer的中文分词，基于`solr6.5.0`：

1、首先下载IKAnalyzer，这是最新的支持solr 6.5，
http://files.cnblogs.com/files/wander1129/ikanalyzer-solr5.zip

在[这里](https://github.com/hewentian/solr-demo/tree/master/docs)也有提供

解压后会有四个文件:
ext.dic						为扩展字典
stopword.dic				为停止词字典
IKAnalyzer.cfg.xml			为配置文件
ik-analyzer-solr5-5.x.jar	为分词jar包


2、将文件夹下的IKAnalyzer.cfg.xml, ext.dic和stopword.dic 三个文件复制到`/home/hewentian/ProjectD/solr-6.5.0/server/resources`目录下（视个人安装solr，而适当修改）
注意: 记得将stopword.dic，ext.dic的编码方式为UTF-8 无BOM的编码方式。 

并修改IKAnalyzer.cfg.xml（一般默认即可）
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
	<comment>IK Analyzer 扩展配置</comment>
	
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">ext.dic;</entry> 
	
	<!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">stopword.dic;</entry> 
</properties>
```

3、在ext.dic 里增加自己的扩展词典，例如，唯品会 聚美优品


4、复制`ik-analyzer-solr5-5.x.jar`到`/home/hewentian/ProjectD/solr-6.5.0/server/solr-webapp/webapp/WEB-INF/lib`目录下


5、在`/home/hewentian/ProjectD/solr-6.5.0/server/solr/mysqlCore/conf/managed-schema`文件的</schema>前增加如下配置
``` xml
<!-- 我添加的IK分词 -->
<fieldType name="text_ik" class="solr.TextField"> 
	<analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/> 
	<analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/> 
</fieldType>
```
重启solr:
``` bash
$ cd /home/hewentian/ProjectD/solr-6.5.0/bin
$ ./solr stop -all
$ ./solr start
```

在浏览器中打开：
http://localhost:8983/solr/#/mysqlCore/analysis

在Field Value (Index)中输入：中华人民共和国
在Analyse Fieldname / FieldType:选择 text_ik

点[Analyse Values]即可看到分词

至此，配置完成。



### 下面说说配置solr默认的中文分词：
1.首先将需要的`lucene-analyzers-smartcn-6.5.0.jar`复制到`WEB-INF/lib/`目录下
``` bash
$ cd /home/hewentian/ProjectD/solr-6.5.0
$ cp contrib/analysis-extras/lucene-libs/lucene-analyzers-smartcn-6.5.0.jar server/solr-webapp/webapp/WEB-INF/lib/
```

2、为mysqlCore添加对中文分词的支持，在`/home/hewentian/ProjectD/solr-6.5.0/server/solr/mysqlCore/conf/managed-schema`文件的</schema>前增加如下配置
``` xml
<fieldType name="text_smartcn" class="solr.TextField" positionIncrementGap="0">
	<analyzer type="index">
		<tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
	</analyzer>
	<analyzer type="query">
		<tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
	</analyzer>
</fieldType>
```
重启solr测试即可。可以与IK分词的结果作对比。
