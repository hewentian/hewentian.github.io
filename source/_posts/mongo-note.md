---
title: mongo 学习笔记
date: 2018-03-07 14:39:42
tags: mongo
categories: db
---

### 安装mongodb数据库
首先，我们必须安装mongodb，这样才能使用。我们下载相应版本的mongodb，因为我的笔记本电脑是ubuntu，所以我下载mongodb的tgz版本，下载地址如下：

    https://www.mongodb.com/download-center/community

下载之后，得到如下两个文件：
mongodb-linux-x86_64-ubuntu1804-4.0.6.tgz
mongodb-linux-x86_64-ubuntu1804-4.0.6.tgz.md5

mongodb依赖libcurl4、openssl，我们必须先安装这两个依赖包：
``` bash
如已安装，则可跳过，验证命令如下
$ dpkg -s libcurl4
$ dpkg -s openssl

安装命令
$ sudo apt-get install libcurl4 openssl
```

我们将压缩包下载到`/home/hewentian/ProjectD`目录，然后解压：
``` bash
$ cd /home/hewentian/ProjectD
$ tar xf mongodb-linux-x86_64-ubuntu1804-4.0.6.tgz
$
$ ls mongodb-linux-x86_64-ubuntu1804-4.0.6/
bin  LICENSE-Community.txt  MPL-2  README  THIRD-PARTY-NOTICES
```

创建data目录和log目录
``` bash
$ cd /home/hewentian/ProjectD
$ mkdir -p db/mongodb/data
$ mkdir -p db/mongodb/log
```

创建mongod.conf配置文件
``` bash
$ cd /home/hewentian/ProjectD/mongodb-linux-x86_64-ubuntu1804-4.0.6/
$ vi mongod.conf
```

并在其中输入如下内容：
``` bash
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /home/hewentian/ProjectD/db/mongodb/data
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/hewentian/ProjectD/db/mongodb/log/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1
#  bindIp: 0.0.0.0  


# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /home/hewentian/ProjectD/db/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

#security:
#  authorization: enabled

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:
```


### 启动mongodb
``` bash
$ cd /home/hewentian/ProjectD/mongodb-linux-x86_64-ubuntu1804-4.0.6/bin
$ ./mongod -f ../mongod.conf

about to fork child process, waiting until server is ready for connections.
forked process: 24049
child process started successfully, parent exiting
```


### 停止mongodb
不要直接通过kill命令杀掉mongodb的进程，而应通过它官方提供的关闭脚本，如下：
``` bash
$ cd /home/hewentian/ProjectD/mongodb-linux-x86_64-ubuntu1804-4.0.6/bin
$ ./mongod -f ../mongod.conf --shutdown

killing process with pid: 24049
```


### 配置登录用户名和密码
要开启密码登录，首先要将mongod.conf中的以下选项打开：

    security:
      authorization: enabled

然后重启mongodb服务。

1. 添加管理员
使用mongo命令进入命令行交互模式，创建第一个用户admin，该用户需要有用户管理权限，其角色为root。
``` bash
$ cd /home/hewentian/ProjectD/mongodb-linux-x86_64-ubuntu1804-4.0.6/bin
$ ./bin/mongo --host 127.0.0.1
MongoDB shell version v4.0.6
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("cbd0445f-667b-4d2a-92a8-66f8ad1d07ef") }
MongoDB server version: 4.0.6
> use admin
switched to db admin
> db.createUser({user:"admin",pwd:"12345",roles:["root"]})
Successfully added user: { "user" : "admin", "roles" : [ "root" ] }
> 
> show collections
Warning: unable to run listCollections, attempting to approximate collection names by parsing connectionStatus
> db.auth("admin","12345")
1
> show collections
```

2. 添加数据库用户
为数据库添加用户，添加用户前需要切换到该数据库，这里简单设置其角色为dbOwner
``` bash
> use bfg
switched to db bfg
> db.createUser({user: "bfg", pwd: "bfg100", roles: [{ role: "dbOwner", db: "bfg" }]})
Successfully added user: {
	"user" : "bfg",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "bfg"
		}
	]
}
```

这样，bfg的用户就只能访问bfg这个库了，登录方式如下：
``` bash
$ ./bin/mongo --host 127.0.0.1
MongoDB shell version v4.0.6
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("873114cf-fb7e-436a-b939-7e765dbecee9") }
MongoDB server version: 4.0.6
> use bfg
switched to db bfg
> db.auth("bfg","bfg100")
1
> show dbs
bfg  0.000GB
```

至此，数据库安装完毕（此安装过程2019年初才补上）。


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


### 删除数据
删除数据的时候，要注意条件为null的情况，这在组成json条件的时候，会变成{}，这会将整个库的数据都删掉，这个要避免。
``` java
MongoCollection<Document> mongoCollection = null; // 这里不建连接，仅演示
JSONArray userNames = new JSONArray();
for (String userName : Arrays.asList("", null, "xiao ming")) {
    JSONObject obj = new JSONObject();
    obj.put("userName", userName);
    userNames.add(obj);
}

JSONObject condition = new JSONObject();
condition.put("$or", userNames);

// 删除原有的数据
log.info("删除的语句为：" + condition); // {"$or":[{"userName":""},{},{"userName":"xiao ming"}]}
DeleteResult deleteResult = mongoCollection.deleteMany(BsonDocument.parse(condition.toJSONString()));

log.info("======mongo删除原有的数据条数：======" + deleteResult.getDeletedCount());
```



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


### mongodb类型转换
如下所示，将字符串类型的数据转换为int, double, date类型：
``` java
先插入一条数据，都是字符串类型：
db.getCollection("userInfo").insert({"age":"20", "salary":"30", "birthday":"2018-12-21"})

{
    "_id" : ObjectId("5c45b7099a14ab9807edaa75"),
    "age" : "20",
    "salary" : "30",
    "birthday" : "2018-12-21"
}

改变字段类型：
db.getCollection("userInfo").find({}).forEach(function(doc) {
    db.getCollection('userInfo').updateOne({_id: doc._id}, {$set: {"age": NumberInt(doc.age), "salary": parseInt(doc.salary), "birthday": new ISODate(doc.birthday)}})
})

{
    "_id" : ObjectId("5c45b7099a14ab9807edaa75"),
    "age" : 20,
    "salary" : 30.0,
    "birthday" : ISODate("2018-12-21T00:00:00.000Z")
}
```

如果要将集合中已有文档的`ISODate`类型转换成`String`类型的日期，例如：将`updateTime`由`ISODate("2019-04-04T07:12:37.295Z")`转换为`2019-04-04 07:12:37`类型：
``` javascript
db.getCollection('userInfo').find({"updateTime":{"$type":9}}).forEach(function(doc) {
    var mydate = doc.updateTime;
    if (mydate) {
        mydate = mydate.toJSON();
        if (mydate.length > 19) {
            mydate = mydate.substr(0, 19)
            mydate = mydate.replace('T',' ');
         }
    }

   db.getCollection('userInfo').update({"_id":doc._id}, {"$set":{"updateTime":mydate}})
})


验证结果：
db.getCollection('userInfo').find({"_id":ObjectId("5abc7ffc00a32c2b045e598c")})
```


### mongodb默认插入类型
插入的int32整数会默认转为Double类型，若需插入为整数，需指定NumberInt：
``` javascript
db.getCollection("userInfo").insert({"age":20, "salary":NumberInt(30), "birthday":"2018-12-21"})

db.getCollection("userInfo").find({})
{
    "_id" : ObjectId("5d3815a2a7de52e9fed7bbcf"),
    "age" : 20.0,
    "salary" : 30,
    "birthday" : "2018-12-21"
}
```


### mongodb日期查询
日期的类型不同，查询的方式也不同：
``` java
如果日期的类型为String：
db.getCollection('userInfo').find({"insertTime":{$regex:"2019-01-07 *"}}) // *号前有个空格
db.getCollection('userInfo').find({"insertTime":{$gte:"2019-02-01 00:00:00",$lte:"2019-02-11 23:59:59"}})

如果日期的类型为ISODate：
db.getCollection('userInfo').find({"insertTime":{$gte:ISODate("2019-01-01T00:00:00Z"),$lte:ISODate("2019-02-11T23:59:59Z")}})
```

### 复制集合
在复制集合的时候，修改某些值，可以使用javascript实现（建议在mongo shell下执行，这样不容易超时），示例：
``` java
var count=0;
var totalCount = db.getCollection('user').count({});
var lastId='';

while(count < totalCount) {
    db.getCollection('user').find({"_id":{"$gt":lastId}}).sort({"_id":1}).limit(100).forEach((doc)=> {
        if (++count % 1000 == 0) {
            print('handling: ' + count + ' / ' + totalCount)
        }
        
        lastId = doc._id;
        doc._id=doc.regId;
       db.getCollection('user_new').save(doc);
    });
}
```


### 正则表达式的使用
查询用户名`name`前后有空格的记录：
``` java
db.getCollection('userInfo').find({"name":/^\s+|\s+$/})
db.getCollection('userInfo').find({"$or":[{"name":/^\s+|\s+$/}, {"postCode":/^\s+|\s+$/}]})
```


### 删除字段
The following update() operation uses the `$unset` operator to remove the fields `quantity` and `instock` from the first document in the `products` collection where the field `sku` has a value of `unknown`.
``` bash
db.products.update(
   { sku: "unknown" },
   { $unset: { quantity: "", instock: "" } }
)
```

if you want to update all matched, use `updateMany` instead.


