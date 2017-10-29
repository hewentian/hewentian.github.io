---
title: solr学习笔记
date: 2017-10-29 12:12:26
tags: solr
---

### HttpSolrClient
HttpSolrClient 使用 HTTPClient 和 solr 服务器进行通信。
``` java
import org.apache.solr.client.solrj.impl.HttpSolrClient;

String baseSolrUrl = "http://localhost:8983/solr/";

HttpSolrClient solrClient = new HttpSolrClient.Builder().withBaseSolrUrl(baseSolrUrl).build();
solrClient.setConnectionTimeout(1000);
solrClient.setDefaultMaxConnectionsPerHost(100);
solrClient.setMaxTotalConnections(100);
```
HttpSolrClient 是线程安全的，建议重复使用 HttpSolrClient 实例。

### EmbeddedSolrServer
EmbeddedSolrServer 提供和 HttpSolrClient 相同的接口，它不需要http连接。如果，你使用的是一个本地的 solrServer 的话，这是一个不错的选择。
``` java
import org.apache.solr.client.solrj.embedded.EmbeddedSolrServer;
import org.apache.solr.core.CoreContainer;

System.setProperty("solr.solr.home", "E:\\solr-6.5.0\\server\\solr");
		
CoreContainer coreContainer = new CoreContainer("E:\\solr-6.5.0\\server\\solr");
coreContainer.load(); // don't forge to invoke this method since solr 4.4.0
		
EmbeddedSolrServer embeddedSolrServer = new EmbeddedSolrServer(coreContainer, coreName);
```

通过 POJO 方式往 Solr 中添加 doc 时，要使用 注解`@org.apache.solr.client.solrj.beans.Field`，@Field 可以被用在域上，或者是setter方法上。如果一个域的名称跟bean的名称是不一样的，那么在java注释中填写别名，具体的，可以参照下面的例子：
``` java
public class User {
	/** 用户id */
	@Field
	private String id;

	/** 名字 */
	@Field
	private String name;

	/** 年龄 */
	@Field
	private Integer age;

	// getter setter method
}
```

当将一个 doc 往 solr 中添加之后，要记得调用 commit 方法，当然，如果在 solrconfig.xml 中有配置硬/软提交的，可以省略
``` java
SolrClient solrClient = createSolrClient(baseSolrUrl);
UpdateResponse response = solrClient.addBean(collection, user);

solrClient.commit(collection);
solrClient.close();
```

Group 分组划分结果，返回的是分组结果；
Facet 分组统计，侧重统计，返回的是分组后的数量；
下面分别说下

### Facet查询
进行Facet查询需要在请求参数中加入”facet=on”或者”facet=true”只有这样Facet组件才起作用.
搜索结果按照Facet的字段分组并统计
``` java
SolrQuery solrQuery = new SolrQuery();
solrQuery.setQuery("老师");
solrQuery.setFacet(true);
solrQuery.setFacetMinCount(1); // 统计结果大于 1 的才会返回
solrQuery.setFacetLimit(10); // 对于每一个 field 最多返回 10 个结果
solrQuery.addFacetField("age", "name");
solrQuery.setRows(0); // 设置返回结果条数，如果分组查询，就设置为0

QueryResponse queryResponse = SolrUtils.getQueryResponse(baseSolrUrl, collection, solrQuery);
List<FacetField> facetFields = queryResponse.getFacetFields();
facetFields.stream().forEach(ff -> {
	ff.getValues().stream().forEach(c -> {
		System.out.println(c.getName() + ", " + c.getCount());
	});
	System.out.println();
});

///~: output
50, 3
20, 2
40, 1

陈六, 2
张三, 1
李七, 1
李四, 1
王五, 1
```
facet的参数见solr官方wiki  http://wiki.apache.org/solr/SimpleFacetParameters

facet 参数字段要求:
1.字段必须被索引
2.facet=on 或 facet=true
 
1.facet.field  
分组的字段
2.facet.prefix 
表示Facet字段前缀
3.facet.limit 
Facet字段返回条数
4.facet.offict 
开始条数,偏移量,它与facet.limit配合使用可以达到分页的效果
5.facet.mincount 
Facet字段最小count,默认为0
6.facet.missing 
如果为on或true,那么将统计那些Facet字段值为null的记录
7.facet.method 
取值为enum或fc,默认为fc, fc表示Field Cache
8.facet.enum.cache.minDf 
当facet.method=enum时,参数起作用,文档内出现某个关键字的最少次数

### Group查询
分组统计查询不同于分组统计（Facet），facet只是简单统计记录数，并不能为每组数据返回实际的数据回来，solr提供的grouping查询能够解决这一问题，也就是说，他除了能分组外，还能把每组数据返回来。
``` java
SolrQuery solrQuery = new SolrQuery();
solrQuery.setQuery("*:*");
solrQuery.set(GroupParams.GROUP, true);
solrQuery.set(GroupParams.GROUP_FIELD, "age", "name");
solrQuery.set(GroupParams.GROUP_LIMIT, 3);
solrQuery.set(GroupParams.GROUP_FORMAT, "grouped");
solrQuery.set(GroupParams.GROUP_MAIN, false);
solrQuery.set(GroupParams.GROUP_TOTAL_COUNT, true);
solrQuery.set(GroupParams.GROUP_SORT, "age asc");

QueryResponse queryResponse = SolrUtils.getQueryResponse(baseSolrUrl, collection, solrQuery);
// 通过GroupResponse获取GroupCommand，进而通过GroupCommand获取Group，而Group又可以获得SolrDocumentList
GroupResponse groupResponse = queryResponse.getGroupResponse();
List<GroupCommand> groupCommands = groupResponse.getValues();
for (GroupCommand gc : groupCommands) {
	System.out.println("\n########## group name: " + gc.getName());
	// 只有设置 GroupParams.GROUP_TOTAL_COUNT 这里 getNGroups 才有效果
	System.out.println("########## Num of Groups Found: " + gc.getNGroups()); 
	System.out.println("########## Num of documents Found: " + gc.getMatches());

	List<Group> groups = gc.getValues();
	for (Group g : groups) {
		SolrDocumentList solrDocumentList = g.getResult();
		System.out.println("\n########## group name: " + gc.getName() + ", groupValue: " + g.getGroupValue()
				+ ", Num of Documents in this group: " + solrDocumentList.getNumFound());
		if (CollectionUtils.isNotEmpty(solrDocumentList)) {
			for (SolrDocument sd : solrDocumentList) {
				System.out.println(sd);
			}
		}
	}
}
```

## solr建立索引的过程和查询过程
### solr索引建立： 
第一步：solr会对句子进行空格分隔，比如对 JAVA is a kind of computer language 分词，分为JAVA，is，a，kind，of，computer，language;

第二步：solr会对上面分出来的词，去掉停用词，这里比如a，is;

第三部：solr会对上面分出来的英文都转化为小写 java,kind,of,computer,language。

### solr查询过程： 
第一步：同索引过程；

第二步：同索引过程；

第三部：同索引过程；

最后分词匹配成功，那么索引的那句话就会被查询出来，这里需要注意：索引和查询的分词器必须使用一样的！！！

## 下面说说Solr基础
详情请参阅：
http://blog.csdn.net/awj3584/article/details/16963525
https://lucene.apache.org/solr/guide/
以下部分为抄录内容
### types 部分
是一些常见的可重用定义，定义了 Solr（和 Lucene）如何处理 Field。也就是添加到索引中的xml文件属性中的类型，如int、text、date等.
``` xml
<fieldType name="string" class="solr.StrField" sortMissingLast="true"/>
<fieldType name="boolean" class="solr.BoolField" sortMissingLast="true"/>
<fieldType name="int" class="solr.TrieIntField" precisionStep="0" positionIncrementGap="0"/>
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
<analyzer type="index">
  <tokenizer class="solr.StandardTokenizerFactory"/>
  <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
  <filter class="solr.LowerCaseFilterFactory"/>
</analyzer>
<analyzer type="query">
  <tokenizer class="solr.StandardTokenizerFactory"/>
  <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
  <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
  <filter class="solr.LowerCaseFilterFactory"/>
</analyzer>
</fieldType>
```
参数说明:

| 属性 					| 描述 																							 |
| ------------- 		| :-------------																				 |
| name 					| 标识而已 																						 |
| class 				| 和其他属性决定了这个fieldType的实际行为。														 |
| sortMissingLast 		| 设置成true没有该field的数据排在有该field的数据之后，而不管请求时的排序规则, 默认是设置成false。|
| sortMissingFirst 		| 跟上面倒过来呗。 默认是设置成false 															 |
| analyzer 				| 字段类型指定的分词器 																			 |
| type 					| 当前分词用用于的操作.index代表生成索引时使用的分词器query代码在查询时使用的分词器 			 |
| tokenizer 			| 分词器类 																						 |
| filter 				| 分词后应用的过滤器  过滤器调用顺序和配置相同. 												 |

### fileds
是你添加到索引文件中出现的属性名称，而声明类型就需要用到上面的types
``` xml
<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false"/>
<field name="path" type="text_smartcn" indexed="false" stored="true" multiValued="false" termVector="true" />
<field name="content" type="text_smartcn" indexed="false" stored="true" multiValued="false" termVector="true"/>
<field name ="text" type ="text_ik" indexed ="true" stored ="false" multiValued ="true"/>
<field name ="pinyin" type ="text_pinyin" indexed ="true" stored ="false" multiValued ="false"/>
<field name="_version_" type="long" indexed="true" stored="true"/>
<dynamicField name="*_i" type="int" indexed="true" stored="true"/>
<dynamicField name="*_l" type="long" indexed="true" stored="true"/>
<dynamicField name="*_s" type="string" indexed="true" stored="true" />
```
field: 固定的字段设置
dynamicField: 动态的字段设置,用于后期自定义字段,*号通配符.例如: test_i就是int类型的动态字段.
还有一个特殊的字段copyField,一般用于检索时用的字段这样就只对这一个字段进行索引分词就行了copyField的dest字段如果有多个source一定要设置multiValued=true,否则会报错的
``` xml
<copyField source="content" dest="pinyin"/>
<copyField source="content" dest="text"/>
<copyField source="pinyin" dest="text"/>
```
字段属性说明:

| 属性 					| 描述 																															   | 
| ------------- 		| :-------------																				 								   |
| name 					| 字段类型名 																												       |
| class					| java类名																														   |
| indexed				| 缺省true。 说明这个数据应被搜索和排序，如果数据没有indexed，则stored应是true。												   |
| stored				| 缺省true。说明这个字段被包含在搜索结果中是合适的。如果数据没有stored,则indexed应是true。										   |
| omitNorms				| 字段的长度不影响得分和在索引时不做boost时，设置它为true。一般文本字段不设置为true。											   |
| termVectors			| 如果字段被用来做more like this 和highlight的特性时应设置为true。																   |
| compressed			| 字段是压缩的。这可能导致索引和搜索变慢，但会减少存储空间，只有StrField和TextField是可以压缩，这通常适合字段的长度超过200个字符。 |
| multiValued 			| 字段多于一个值的时候，可设置为true。																							   |
| positionIncrementGap	| 和multiValued一起使用，设置多个值之间的虚拟空白的数量																			   |

注意:_version_ 是一个特殊字段,不能删除,是记录当前索引版本号的.

uniqueKey: 唯一键，这里配置的是上面出现的fileds，一般是id、url等不重复的。在更新、删除的时候可以用到。
defaultSearchField: 默认搜索属性，如q=solr就是默认的搜索那个字段
solrQueryParser: 查询转换模式，是并且还是或者（AND/OR必须大写）

## solr配置solrconfig.xml
对于我的机器来说，是在以下目录：E:\solr-6.5.0\server\solr\mysqlCore\conf\solrconfig.xml

### 索引indexConfig
Solr 性能因素，来了解与各种更改相关的性能权衡。 表 1 概括了可控制 Solr 索引处理的各种因素：

| 属性 					| 描述 																															   | 
| ------------- 		| :-------------																				 								   |
| useCompoundFile 		| 通过将很多 Lucene 内部文件整合到一个文件来减少使用中的文件的数量。这可有助于减少 Solr 使用的文件句柄数目，代价是降低了性能。除非是应用程序用完了文件句柄，否则 false 的默认值应该就已经足够。 |
| ramBufferSizeMB		| 在添加或删除文档时，为了减少频繁的更些索引,Solr会选缓存在内存中,当内存中的文件大于设置的值,才会更新到索引库。较大的值可使索引时间变快但会牺牲较多的内存。 |
| maxBufferedDocs		| 同 ramBufferSizeMB，如两个值同时设置,满足一个就会进行刷新索引. |
| mergeFactor			| 决定低水平的 Lucene 段被合并的频率。较小的值（最小为 2）使用的内存较少但导致的索引时间也更慢。较大的值可使索引时间变快但会牺牲较多的内存。 |
| maxIndexingThreads	| 是indexWriter生成索引时使用的最大线程数 |
| unlockOnStartup		| unlockOnStartup 告知 Solr 忽略在多线程环境中用来保护索引的锁定机制。在某些情况下，索引可能会由于不正确的关机或其他错误而一直处于锁定，这就妨碍了添加和更新。将其设置为 true | 可以禁用启动锁定，进而允许进行添加和更新。 |
| lockType				| 1. single: 在只读索引或是没有其它进程修改索引时使用. 2. native: 使用操作系统本地文件锁,不能使用多个Solr在同一个JVM中共享一个索引. 3. simple :使用一个文本文件锁定索引. |



### 查询配置query

| 属性 						| 描述 																															   							   			|
| ------------- 			| :-------------																				 								   							   			|
| maxBooleanClauses			| 最大的BooleanQuery数量. 当值超出时，抛出 TooManyClausesException.注意这个是全局的,如果是多个SolrCore都会使用一个值,每个Core里设置不一样的化,会使用最后一个的.			|
| filterCache				| filterCache存储了无序的lucene document id集合，1.存储了filter queries(“fq”参数)得到的document id集合结果。2还可用于facet查询3. 3）如果配置了useFilterForSortedQuery，那么如果查询有filter，则使用filterCache。																													 |
| queryResultCache			| 缓存搜索结果,一个文档ID列表																																   			|
| documentCache				| 缓存Lucene的Document对象,不会自热																															   			|
| fieldValueCache			| 字段缓存使用文档ID进行快速访问。默认情况下创建fieldValueCache即使这里没有配置。																			   	 		|
| enableLazyFieldLoading	| 若应用程序预期只会检索 Document 上少数几个 Field，那么可以将属性设置为 true。延迟加载的一个常见场景大都发生在应用程序返回和显示一系列搜索结果的时候，用户常常会单击其中的一个来查看存储在此索引中的原始文档。初始的显示常常只需要显示很短的一段信息。若考虑到检索大型 Document 的代价，除非必需，否则就应该避免加载整个文档。																																						 |
| queryResultWindowSize		| 一次查询中存储最多的doc的id数目.																															   			|
| queryResultMaxDocsCached	| 查询结果doc的最大缓存数量, 例如要求每页显示10条,这里设置是20条,也就是说缓存里总会给你多出10条的数据.让你点示下一页时很快拿到数据.							   		    |
| listener					| 选项定义 newSearcher 和 firstSearcher 事件，您可以使用这些事件来指定实例化新搜索程序或第一个搜索程序时应该执行哪些查询。如果应用程序期望请求某些特定的查询，那么在创建新搜索程序或第一个搜索程序时就应该反注释这些部分并执行适当的查询。	|
| useColdSearcher			| 是否使用冷搜索,为false时使用自热后的searcher																															|
| maxWarmingSearchers		| 最大自热searcher数量																																				  	|

### 删除索引
删除索引可以通过两种方式操作,一种是通过文档ID进行删除,别一种是通过查询到的结果进行删除.
``` java
通过ID删除方式代码:
solrClient.deleteById(id);

//或是使用批量删除
solrClient.deleteById(ids);

通过查询删除方式代码:
solrClient.deleteByQuery("*:*"); // ”*.*”就查询所有内容的, 这样就删除了所有文档索引
```

### 优化索引
优化Lucene 的索引文件以改进搜索性能。索引完成后执行一下优化通常比较好。如果更新比较频繁，则应该在使用率较低的时候安排优化。一个索引无需优化也可以正常地运行。优化是一个耗时较多的过程。
``` java
solrClient.optimize(); //不要频繁的调用..尽量在无人使用时调用.
```

### 查询参数

名称	描述
q
查询字符串，必须的。
fq
filter query。使用Filter Query可以充分利用Filter Query Cache，提高检索性能。作用：在q查询符合结果中同时是fq查询符合的，例如：q=mm&fq=date_time:[20081001 TO 20091031]，找关键字mm，并且date_time是20081001到20091031之间的。
fl
field list。指定返回结果字段。以空格“ ”或逗号“,”分隔。
start
用于分页定义结果起始记录数，默认为0。
rows
用于分页定义结果每页返回记录数，默认为10。
sort
排序，格式:sort=<field name>+<desc|asc>[,<field name>+<desc|asc>]… 。示例：（inStock desc, price asc）表示先 “inStock” 降序, 再 “price” 升序，默认是相关性降序。
df
默认的查询字段，一般默认指定。
q.op
覆盖schema.xml的defaultOperator（有空格时用"AND"还是用"OR"操作逻辑），一般默认指定。必须大写
wt
writer type。指定查询输出结构格式，默认为“xml”。在solrconfig.xml中定义了查询输出格式：xml、json、python、ruby、php、phps、custom。
qt
query type，指定查询使用的Query Handler，默认为“standard”。
explainOther
设置当debugQuery=true时，显示其他的查询说明。
defType
设置查询解析器名称。
timeAllowed
设置查询超时时间。
omitHeader
设置是否忽略查询结果返回头信息，默认为“false”。
indent
返回的结果是否缩进，默认关闭，用 indent=true|on 开启，一般调试json,php,phps,ruby输出才有必要用这个参数。
version
查询语法的版本，建议不使用它，由服务器指定默认值。
debugQuery
设置返回结果是否显示Debug信息。


### 查询语法

1.匹配所有文档：*:*
2.强制、阻止和可选查询：
1)    Mandatory：查询结果中必须包括的(for example, only entry name containing the word make)
Solr/Lucene Statement：+make, +make +up ,+make +up +kiss
2)    prohibited：(for example, all documents except those with word believe)
Solr/Lucene Statement：+make +up -kiss
3)    optional：
Solr/Lucene Statement：+make +up kiss
3.布尔操作：AND、OR和NOT布尔操作（必须大写）与Mandatory、optional和prohibited相似。
1)       make AND up ＝ +make +up :AND左右两边的操作都是mandatory
2)       make || up ＝ make OR up＝make up :OR左右两边的操作都是optional
3)       +make +up NOT kiss ＝ +make +up –kiss
4)       make AND up OR french AND Kiss不可以达到期望的结果，因为AND两边的操作都是mandatory的。
4. 子表达式查询（子查询）：可以使用“()”构造子查询。
示例：(make AND up) OR (french AND Kiss)
5.子表达式查询中阻止查询的限制：
示例：make (-up):只能取得make的查询结果；要使用make (-up *:*)查询make或者不包括up的结果。
6.多字段fields查询：通过字段名加上分号的方式（fieldName:query）来进行查询
示例：entryNm:make AND entryId:3cdc86e8e0fb4da8ab17caed42f6760c
7.通配符查询（wildCard Query）：
1)       通配符？和*：“*”表示匹配任意字符；“？”表示匹配出现的位置。
示例：ma?*（ma后面的一个位置匹配），ma??*(ma后面两个位置都匹配)
2)       查询字符必须要小写:+Ma +be**可以搜索到结果；+Ma +Be**没有搜索结果.
3)       查询速度较慢，尤其是通配符在首位：主要原因一是需要迭代查询字段中的每个term，判断是否匹配；二是匹配上的term被加到内部的查询，当terms数量达到1024的时候，查询会失败。
4)       Solr中默认通配符不能出现在首位（可以修改QueryParser，设置
setAllowLeadingWildcard为true）
5)       set setAllowLeadingWildcard to true.
8.模糊查询、相似查询：不是精确的查询，通过对查询的字段进行重新插入、删除和转换来取得得分较高的查询解决（由Levenstein Distance Algorithm算法支持）。
1)       一般模糊查询：示例：make-believ~
2)       门槛模糊查询：对模糊查询可以设置查询门槛，门槛是0~1之间的数值，门槛越高表面相似度越高。示例：make-believ~0.5、make-believ~0.8、make-believ~0.9
9.范围查询（Range Query）：Lucene支持对数字、日期甚至文本的范围查询。结束的范围可以使用“*”通配符。
示例：
1)       日期范围（ISO-8601 时间GMT）：sa_type:2 AND a_begin_date:[1990-01-01T00:00:00.000Z TO 1999-12-31T24:59:99.999Z]
2)       数字：salary:[2000 TO *]
3)       文本：entryNm:[a TO a]
10.日期匹配：YEAR, MONTH, DAY, DATE (synonymous with DAY) HOUR, MINUTE, SECOND, MILLISECOND, and MILLI (synonymous with MILLISECOND)可以被标志成日期。
示例：
1)  r_event_date:[* TO NOW-2YEAR]：2年前的现在这个时间
2)  r_event_date:[* TO NOW/DAY-2YEAR]：2年前前一天的这个时间
