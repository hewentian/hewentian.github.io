---
title: solr配置拼音检索
date: 2018-01-08 14:52:08
tags: solr
categories: bigdata
---

配置拼音检索，基于`solr6.5.0`：

1、前期准备，需要用到pinyin4j-2.5.0.jar、pinyinAnalyzer4.3.1.jar这两个jar包,下载地址：
http://files.cnblogs.com/files/wander1129/pinyin.zip

在[这里](https://github.com/hewentian/solr-demo/tree/master/docs)也有提供pinyin.zip

解压后会有2个文件:
pinyin4j-2.5.0.jar
pinyinAnalyzer4.3.1.jar

2、将上面的两个文件复制到`/home/hewentian/ProjectD/solr-6.5.0/server/solr-webapp/webapp/WEB-INF/lib`目录下，还有执行如下命令，
``` bash
$ cd /home/hewentian/ProjectD/solr-6.5.0
$ cp contrib/analysis-extras/lucene-libs/lucene-analyzers-smartcn-6.5.0.jar server/solr-webapp/webapp/WEB-INF/lib/
```
将需要的`lucene-analyzers-smartcn-6.5.0.jar`复制到`WEB-INF/lib/`目录下

3、在`/home/hewentian/ProjectD/solr-6.5.0/server/solr/mysqlCore/conf/managed-schema`文件的</schema>前增加如下配置
``` xml
<fieldType name="text_pinyin" class="solr.TextField" positionIncrementGap="0">
    <analyzer type="index">
        <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory" />
        <filter class="com.shentong.search.analyzers.PinyinTransformTokenFilterFactory" minTermLenght="2" />
        <filter class="com.shentong.search.analyzers.PinyinNGramTokenFilterFactory" minGram="1" maxGram="20" />
    </analyzer>
    <analyzer type="query">
        <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory" />
        <filter class="com.shentong.search.analyzers.PinyinTransformTokenFilterFactory" minTermLenght="2" />
        <filter class="com.shentong.search.analyzers.PinyinNGramTokenFilterFactory" minGram="1" maxGram="20" />
    </analyzer>
</fieldType>
```

重启solr:
``` bash
$ cd /home/hewentian/ProjectD/solr-6.5.0/bin
$ ./solr stop -all
$ ./solr start
```

4、在浏览器中打开：
http://localhost:8983/solr/#/mysqlCore/analysis

在Field Value (Index)中输入：中国
在Analyse Fieldname / FieldType:选择 text_pinyin

点[Analyse Values]即可看到分词

至此，配置完成。
