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
  bindIp: 127.0.0.1,192.168.56.110 # 127.0.0.1只能从本机访问，将本机的IP配置到这里，这样其他机器才能通过该IP访问
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
> db.createUser({user:"admin",pwd:"admin",roles:["root"]})
Successfully added user: { "user" : "admin", "roles" : [ "root" ] }
> 
> show collections
Warning: unable to run listCollections, attempting to approximate collection names by parsing connectionStatus
> db.auth("admin","admin")
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


### 修改用户密码
修改用户密码使用`db.changeUserPassword(username, password)`，Updates a user’s password. Run the method in the database where the user is defined, i.e. the database you created the user.


### shell命令行方式连接到mongodb
连接到单机：
``` bash
$ mongo --host 192.168.1.111:27017 --authenticationDatabase user_database -u user_name -p user_password
```

连接到单机中的指定数据库：
``` bash
$ mongo "192.168.1.111:27017/user_database" --authenticationDatabase user_database -u user_name -p user_password
```

连接到副本集：
``` bash
$ mongo "192.168.1.111:27017,192.168.1.112:27017/user_database?replicaSet=mgset-9527&readPreference=secondaryPreferred" --authenticationDatabase user_database -u user_name -p user_password
```


### mongodb查询数组大小
mongodb查询数组大小使用`$size`，例如：我们有一个名为`person`的集合，其中有个字段为`childrenNames`，是数组类型，如果我们要查询`childrenNames`长度为2的数据，则查询语句为：

	db.getCollection('person').find({'childrenNames':{'$size':2}})

但是它不能限定数组的大小范围，只能查询指定的长度。要查询数组范围，我们使用`$exists`，例如：查询数组长度大于等于3的语句如下（检查数组第3个元素是否存了）

	db.getCollection('person').find({'childrenNames.2':{'$exists':1}})


### 数组查询
有如下数组，它的查询方式为：
        db.getCollection('userInfo').find({"sons": {$elemMatch: {"birthday": {$gte:1594252800000, $lte:1594339200000}}}})

``` javascript
{
    "_id" : ObjectId("5f0e26bcf0a06369a460b4c8"),
    "name" : "tony",
    "sons" : [ 
        {
            "name" : "h",
            "birthday" : NumberLong(1594762936000),
        },
		  {
            "name" : "w",
            "birthday" : NumberLong(1594762936000),
        }
    ]
}
```


### mongodb分组查询
如果我们要对集合`person`中的地区字段`area`来分组统计，语法如下：

	db.getCollection('person').aggregate([{'$group':{'_id':'$area','count':{'$sum':1}}}])

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


### 删除数据库
``` bash
> show dbs
> use monitor
> db.dropDatabase()
```


### 删除集合
``` bash
> show collections
> db.collection_name.drop()
```


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

如果导出的数据中包含查询条件，则要用下面这种方式导出：
``` bash
$ mongo "192.168.1.111:27017/user_database" --authenticationDatabase user_database -u user_name -p user_password --quiet --eval 'db.user.find({ _id: {$gt: ObjectId("5ee08d67e144cb56edf945da")}}).forEach(printjson);' > a.json
```

如果导出的查询条件中包含ObjectId，则要用`$oid`来代替它：
``` bash
mongoexport -h 127.0.0.1 --port 27017 -u bfg_user -p a12345678 --authenticationDatabase admin -d bfg -c user -o /home/hewentian/ProjectD/db/user.json -q '{"_id":{"$oid":"5ece9b4d8008d750e611010c"}}'
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


### MongoDB的主键类型修改
**主键类型的修改不能像其他字段一样直接修改**

将String类型的主键修改为ObjectId类型，在Mongodb中String类型对应的int值为2，
我们先增加一条主键为ObjectId记录，然后删除主键为String的记录。
``` javascript
db.getCollection('userInfo').find({"_id": {"$type": 2}}).forEach(function(doc) {
    doc._id = ObjectId(doc._id);
    db.getCollection('userInfo').save(doc);
})

db.getCollection('userInfo').remove({"_id": {"$type": 2}})
```

反之，将ObjectId类型的主键修改为String类型，在Mongodb中ObjectId类型对应的int值为7。
``` javascript
db.getCollection('userInfo').find({"_id": {"$type": 7}}).forEach(function(doc) {
    doc._id = doc._id.toJSON().$oid;
    db.getCollection('userInfo').save(doc);
})

db.getCollection('userInfo').remove({"_id": {"$type": 7}})
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


### mongodb模糊查询
``` java
db.getCollection('userInfo').find({"name":{"$regex":"t"}})
```

这样名字中包含"tom"和"scott"的记录都会查询出来。


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
var count = 0;
var totalCount = db.getCollection('user').count({});
var lastId = "000000000000000000000000";

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


### mongo执行一个js脚本文件
例如有一个js脚本文件，位于`/home/hewentian/Documents/updateMongo.js`，内容如下：
``` javascript
/*
* 处理字段中数据重复的方法：将字符串一分为二，如果前后两部分相同，则更新成其中一部分
*/

var handleCount = 0;
var updateCount = 0;
var totalCount = db.getCollection('userInfo').count({});
var lastId = ObjectId('000000000000000000000000');

while (handleCount < totalCount) {
    db.getCollection('userInfo').find({"_id": {"$gt": lastId}}).sort({"_id": 1}).limit(5).forEach((doc) =>{
        if (++handleCount % 10 == 0 || handleCount == totalCount) {
            print('handling: ' + handleCount + ' / ' + totalCount + ", updateCount = " + updateCount + ", lastId = " + lastId.toJSON().$oid + (handleCount == totalCount ? " end" : ""));
        }

        lastId = doc._id;

        var userName = doc.userName;

        if (userName) { // 这个字段必须要存在
            var len = userName.length;

            if (len % 2 == 0) { // 字符数必须为偶数倍
                var middleIndex = len / 2;
                var substrA = userName.substr(0, middleIndex);
                var substrB = userName.substr(middleIndex);

                if (substrA == substrB) { // 前后两半字符串相同时，才更新
                    db.getCollection('userInfo').update({"_id": doc._id}, {"$set": {"userName": substrA}})
                    updateCount++;
                }
            }
        }
    });
}
```

我们这样执行这个脚本：
``` bash
$ mongo "192.168.1.111:27017/user_database" --authenticationDatabase user_database -u user_name -p user_password /home/hewentian/Documents/updateMongo.js
```


### 使用typeof操作符
有时候，我们在使用javascript操作Mongodb的过程中，可能要根据数据类型进行相关操作：
``` javascript
/*
* 对集合进行清洗，保证字段 telephone 是字符串格式
*/

var handleCount = 0;
var updateCount = 0;
var totalCount = db.getCollection('userInfo').count({});
var lastId = "000000000000000000000000";

while (handleCount < totalCount) {
    db.getCollection('userInfo').find({"_id": {"$gt": lastId}}).sort({"_id": 1}).limit(100).forEach((doc) =>{
        if (++handleCount % 1000 == 0 || handleCount == totalCount) {
            print('handling: ' + handleCount + ' / ' + totalCount + ", updateCount = " + updateCount + ", lastId = " + lastId + (handleCount == totalCount ? " end" : ""));
        }

        lastId = doc._id;

        var telephone = doc.telephone;

        if (telephone && typeof(telephone) == 'object') { // 这个字段必须要存在，并且是内嵌类型文档
            var fixTelephone = telephone.phoneNumber;
            if (fixTelephone && typeof(fixTelephone) == 'string') {
                db.getCollection('userInfo').updateOne({"_id": doc._id}, {"$set": {"telephone": fixTelephone}});
                updateCount++;
            }
        }
    });
}


```


### MongoDB 去重(distinct)查询后求总数(count)
1. 直接使用distinct语句查询，这种查询会将所有查询出来的数据返回给用户，然后对查询出来的结果集求总数（耗内存，耗时一些）
        db.student.distinct("name", {"age" : 18}).length

使用这种方法查询时，查询的结果集大于16M时会查询失败：
        {“message” : “distinct failed: MongoError: distinct too big, 16mb cap”,”stack” : “script:1:20”}

2. 使用聚合函数，多次分组统计结果，最终将聚合的结果数返回给用户
        db.student.aggregate([
            {$match:{"age" : 18}},
            {$project:{"name":1}},
            {$group:{"_id":"$name","count":{$sum:1}}},
            {$sort:{"count":-1}}
        ])

这种查询数据量大时就不会出现如上查询失败的情况，而且这种查询不管是内存消耗还是时间消耗都优于上面一种查询。


### BSON ObjectID Specification
A BSON ObjectID is a 12-byte value consisting of a 4-byte timestamp (seconds since epoch), a 3-byte machine id, 
a 2-byte process id, and a 3-byte counter. Note that the timestamp and counter fields must be stored big endian 
unlike the rest of BSON. This is because they are compared byte-by-byte and we want to ensure a mostly increasing 
order. The format:

        0 1 2 3    4 5 6    7 8    9 10 11
        time       machine	pid	   inc

* TimeStamp: This is a unix style timestamp. It is a signed int representing the number of seconds before or after January 1st 1970 (UTC).
* Machine: This is the first three bytes of the (md5) hash of the machine host name, or of the mac/network address, or the virtual machine id.
* Pid: This is 2 bytes of the process id (or thread id) of the process generating the object id.
* Increment: This is an ever incrementing value, or a random number if a counter can't be used in the language/runtime.

BSON ObjectIds can be any 12 byte binary string that is unique; however, the server itself and almost all drivers use the format above.

ObjectId占用12字节的存储空间，由“时间戳” 、“机器名”、“PID号”和“计数器”组成。使用机器名的好处是在分布式环境中能够避免
单点计数的性能瓶颈。使用PID号的好处是支持同一机器内运行多个mongod实例。最终采用时间戳和计数器的组合来保证唯一性。

自动生成的主键objectId是一个24位的字符串，它是由一组十六进制的字符构成，每个字节两位的十六进制数字，总共用了12字节的存储空间。

* 时间戳
确保ObjectId唯一性依赖的是时间的顺序，不依赖时间的取值，因此集群节点的时间不必完全同步。既然ObjectId已经有了时间戳，
那么在文档中就可以省掉一个时间戳了。在使用ObjectID提取时间时，应注意到MongoDB允许各节点时间不一致这一细节。

* 机器名
机器名通过Md5加密后取前三个字节，应该还是有重复概率的，配置生产集群时检查一下总不会错。另外，我也注意到重启MongoDB后
MD5加密结果会发生变化，在利用ObjectID提取机器名信息时需格外注意。

* PID号
注意到每次重启mongod进程后PID号通常会发生变化就可以了。

* 计数器
计数器占3个字节，表示的取值范围就是`256*256*256-1=16777215`。不妨认为MongDB性能的极限是单台设备一秒钟插入一千万条记录。
以目前的水平看，单台设备一秒钟插入一万条就很不错了，因此ObjectID计数器的设计是够用的。

``` javascript
> db.user.findOne()._id
ObjectId("5eda936e8008d750e63ee723")

> db.user.findOne()._id.toString()
ObjectId("5eda936e8008d750e63ee723")

> db.user.findOne()._id.toJSON()
{ "$oid" : "5eda936e8008d750e63ee723" }

> db.user.findOne()._id.toJSON().$oid.substring(0, 8)
5eda936e

> db.user.findOne()._id.getTimestamp()
ISODate("2020-06-06T02:48:14Z")
```

* 从mongoDB的ObjectId中提取时间信息
取8个字符，得到的是这条数据创建时的时间戳（不带毫秒位数），在后面补上毫秒位数"000"。

java代码
``` java
// ObjectId("5eda936e8008d750e63ee723")
String id = "5eda936e8008d750e63ee723";

// 取前8位
long timestamp = Long.parseLong(Integer.parseInt(id.substring(0, 8), 16) + "000");

Date date = new Date(timestamp);
System.out.println(date); // Sat Jun 06 02:48:14 CST 2020
```

javascript代码
``` javascript
// ObjectId("5eda936e8008d750e63ee723")
var id = "5eda936e8008d750e63ee723";

// 取前8位
var timestamp = Number(parseInt(id.substring(0, 8), 16).toString() + "000");

var date = new Date(timestamp);

console.log(date); // Sat Jun 06 2020 02:48:14 GMT+0800 (China Standard Time)
```


### explain查询执行情况
explain的目的是将mongo的黑盒操作白盒化。比如查询很慢的时候想知道原因。explain有三种模式：
1. queryPlanner: 不会真正的执行查询，只是分析查询，选出winningPlan。
2. executionStats: 返回winningPlan的关键数据。
3. allPlansExecution: 执行所有的plans。

通过explain("executionStats")来选择模式，默认是第一种模式。一些返回字段的说明：
namespace: 本次所查询的集合
indexFilterSet: 是否使用partial index，比如只对某个集合中的部分文档进行index
parsedQuery: 本次执行的查询
executionTimeMillis: 该query查询的总体时间
indexName: 所使用的索引的名字
indexBounds: 索引查找时使用的范围
stage:
  COLLSCAN: 全表扫描
  IXSCAN: 索引扫描
  FETCH: 根据索引去检索指定document
  SHARD_MERGE: 将各个分片返回数据进行merge
  SORT: 表明在内存中进行了排序
  LIMIT: 使用limit限制返回数
  SKIP: 使用skip进行跳过
  IDHACK: 针对_id进行查询

通过这些信息就能判断查询时如何执行的了。示例如下，先插入3个文档：
``` javascript
db.getCollection("userInfo").insert({"age":"20", "name":"scott", "birthday":"2018-12-21"})
db.getCollection("userInfo").insert({"age":"21", "name":"tiger", "birthday":"2019-12-21"})
db.getCollection("userInfo").insert({"age":"23", "name":"tom", "birthday":"1919-11-21"})
```

没有建立索引的时候查询：
``` javascript
db.getCollection("userInfo").find({"name":"tom"}).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "bfg.userInfo",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"name" : {
				"$eq" : "tom"
			}
		},
		"queryHash" : "01AEE5EC",
		"planCacheKey" : "01AEE5EC",
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"name" : {
					"$eq" : "tom"
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "0d550468977f",
		"port" : 27017,
		"version" : "4.4.0",
		"gitVersion" : "563487e100c4215e2dce98d0af2a6a5a2d67c5cf"
	},
	"ok" : 1
}
```

从`"stage" : "COLLSCAN"`可知，没有使用到索引，是全表扫描的。我们建个索引，再查询。

        db.getCollection('userInfo').ensureIndex({"name":1}, {background:true})

``` javascript
db.getCollection("userInfo").find({"name":"tom"}).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "bfg.userInfo",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"name" : {
				"$eq" : "tom"
			}
		},
		"queryHash" : "01AEE5EC",
		"planCacheKey" : "4C5AEA2C",
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"name" : 1
				},
				"indexName" : "name_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"name" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"name" : [
						"[\"tom\", \"tom\"]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "0d550468977f",
		"port" : 27017,
		"version" : "4.4.0",
		"gitVersion" : "563487e100c4215e2dce98d0af2a6a5a2d67c5cf"
	},
	"ok" : 1
}
```

这次可以看到，它使用了索引。


### 监控
    mongodb可以通过profile来监控数据，进行优化。查看当前是否开启profile功能用命令：`db.getProfilingLevel()`
返回level等级，值为`0|1|2`，分别代表意思：0代表关闭，1代表记录慢命令，2代表全部。开启profile功能为
`db.setProfilingLevel(level)`，level为1的时候，慢命令默认值为100ms，更改为`db.setProfilingLevel(level, slowms)`
如`db.setProfilingLevel(1,50)`这样就更改为50毫秒。通过`db.system.profile.find()`查看当前的监控日志。通过执行
`db.system.profile.find({millis:{$gt:500}})`能够返回查询时间在500毫秒以上的查询命令。

这里值的含义
op: query，代表查询
ns: 代表查询的库与集合
command: 命令的内容
responseLength: 返回的结果集大小，byte数
nscanned: 扫描记录数量
filter: 后面是查询条件
nreturned: 返回记录数
ts: 命令执行的时刻
millis: 所花时间

如果发现时间比较长，那么就需要作优化。比如nscanned数很大，或者接近记录总数，那么可能没有用到索引查询。
responseLength很大，有可能返回没必要的字段。
nreturned很大，那么有可能查询的时候没有加限制。

mongo可以通过`db.serverStatus()`查看mongod的运行状态
mongo可以通过`db.currentOp()`可以查看当前正在执行的操作。这两个命令，必须用`admin`帐号才能操作。

如果出现如下错误提示，则是由于auth太多了，退出，并且以admin帐号登录admin库来执行
        mongo --host 192.168.1.100 --port 27017 --authenticationDatabase admin -u admin -p admin

``` javascript
db.currentOp()
{
	"ok" : 0,
	"errmsg" : "too many users are authenticated",
	"code" : 13,
	"codeName" : "Unauthorized"
}
```


