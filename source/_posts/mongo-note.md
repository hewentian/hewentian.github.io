---
title: mongo 学习笔记
date: 2018-03-07 14:39:42
tags: mongo
categories: db
---
### mongodb查询数组大小
mongodb查询数组大小使用`$size`，例如：我们有一个名为`person`的集合，其中有个字段为`childrenNames`，是数组类型，如果我们要查询`childrenNames`长度为2的数据，则查询语句为：

	db.getCollection('person').find({'childrenNames':{'$size':2}})

但是它不能限定数组的大小范围，只能查询指定的长度。要查询数组范围，我们使用`$exists`，例如：查询数组长度大于等于3的语句如下（检查数组第3个元素是否存了）

	db.getCollection('person').find({'childrenNames.2':{'$exists':1}})


### mongodb分组查询
如果我们要对集合`person`中的地区字段`area`来分组统计，语法如下：

	db.getCollection('person').aggregate({'$group':{'_id':'$area','count':{'$sum':1}}})

如果我们还想将分组统计结果，按数量倒序输出显示：

	db.getCollection('person').aggregate([{'$group':{'_id':'$area','count':{'$sum':1}}},{'$sort':{'count':-1}}])

对应的JAVA代码如下：
``` java
MongoDatabase mongoDatabase = MongoUtil.getMongoDatabase(); // 这个是我自已写的工具类
MongoCollection<Document> personCollection = mongoDatabase.getCollection("person");

List<BasicDBObject> pipeline = new ArrayList<BasicDBObject>();
pipeline.add(new BasicDBObject("$group", new BasicDBObject("_id", "$area").append("count", new BasicDBObject("$sum", 1))));
pipeline.add(new BasicDBObject("$sort", new BasicDBObject("count", -1))); // 如果不要排序，则注释这一行

AggregateIterable<Document> aggregate = personCollection.aggregate(pipeline);
MongoCursor<Document> iterator = aggregate.iterator();
```

### mongodb更新部分字段
例如我们要将`person`中`_id`为`123`的数据的`address`修改为：广东，语句如下：

	db.getCollection('person').update({'_id':'123'},{$set:{'address':'广东'}})




### 导出、导入数据库
#### 我们使用`mongoexport`来导出指定的`collection`
该命令位于`{MONGO_HOME}/bin/`目录下，可以把一个`collection`导出成JSON或CSV格式的文件
``` bash
语法：
$ mongoexport -h {SERVER_ADDRESS} --port {SERVER_PORT} -u {USER_NAME} -p {PASSWORD} --authenticationDatabase {AUTH_DB} -d {DB_NAME} -c {COLLECTION_NAME} -o {EXPORT_FILE_PATH} --type json/csv -f fields

参数说明：

	-h: MONGODB服务器的地址
	--port: MONGODB服务器的端口
	-u: 用户名
	-p: 密码
	--authenticationDatabase: 验证数据库
	-d: 数据库名
	-c: collection名
	-o: 输出的文件名
	--type: 输出的格式，默认为json
	-f: 输出的字段，如果-type为csv，则需要加上-f "字段名"

示例：
$ mongoexport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user -o /home/hewentian/ProjectD/db/user.json
$ mongoexport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user -o /home/hewentian/ProjectD/db/user.json --type json -f  "_id,name"
$ mongoexport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user -o /home/hewentian/ProjectD/db/user.csv --type csv -f  "_id,name"
```

#### 我们使用`mongoimport`来导入指定的`collection`
该命令位于`{MONGO_HOME}/bin/`目录下，可以把一个json/csv文件导入到指定的`collection`中
``` bash
语法：
$ mongoimport -h {SERVER_ADDRESS} --port {SERVER_PORT} -u {USER_NAME} -p {PASSWORD} --authenticationDatabase {AUTH_DB} -d {DB_NAME} -c {COLLECTION_NAME} --file {IMPORT_FILE_PATH} --headerline --type json/csv -f fields

参数说明：

	-h: MONGODB服务器的地址
	--port: MONGODB服务器的端口
	-u: 用户名
	-p: 密码
	--authenticationDatabase: 验证数据库
	-d: 数据库名
	-c: collection名
	--file: 要导入的文件
	--type: input format to import: json, csv, or tsv (defaults to 'json') (default: json)
	--headerline: use first line in input source as the field list (CSV and TSV only)
	-f: 导入的字段名，CSV或TSV的时候可用
 
示例：
$ mongoimport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user --file /home/hewentian/ProjectD/db/user.json

注意：--headerline 和 -f 不能同时使用

	--headerline: 使用CSV文件中首列指定的列名作为导入的name
	-f: 会将CSV文件的所有列，作为数据进行导入，包括第一列的列名（如果文件中有指定列名）

$ mongoimport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user --file /home/hewentian/ProjectD/db/user.csv --type csv --headerline
$ mongoimport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user --file /home/hewentian/ProjectD/db/user.csv --type csv -f "_id,name,age"
```

#### 我们使用`mongodump`来导出指定的`database`
该命令位于`{MONGO_HOME}/bin/`目录下，可以把一个`database`导出成指定目录下的JSON、BSON的文件集
``` bash
语法：
$ mongodump -h {SERVER_ADDRESS} --port {SERVER_PORT} -u {USER_NAME} -p {PASSWORD} --authenticationDatabase {AUTH_DB} -d {DB_NAME} -o {DUMP_FILE_PATH}

参数说明：

	-h: MONGODB服务器的地址
	--port: MONGODB服务器的端口
	-u: 用户名
	-p: 密码
	--authenticationDatabase: 验证数据库
	-d: 数据库名
	-o: 输出的文件路径，要事先创建该目录，在该目录下会自动创建要导出的数据库作为子目录

示例：
$ mongodump -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -o /home/hewentian/ProjectD/db/
```

#### 我们使用`mongorestore`来恢复指定的`database`
该命令位于`{MONGO_HOME}/bin/`目录下，可以把指定目录下的JSON、BSON的文件集恢复成`database`
``` bash
语法：
$ mongorestore -h {SERVER_ADDRESS} --port {SERVER_PORT} -u {USER_NAME} -p {PASSWORD} --authenticationDatabase {AUTH_DB} -d {DB_NAME} --dir {RESTORE_FILE_PATH}

参数说明：

	-h: MONGODB服务器的地址
	--port: MONGODB服务器的端口
	-u: 用户名
	-p: 密码
	--authenticationDatabase: 验证数据库
	-d: 数据库名
	--dir: 要恢复的数据库的位置，注意与上面备份的 -o 不同，这里要指定到数据库的目录

示例：
$ mongorestore -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg --dir /home/hewentian/ProjectD/db/bfg/
```
