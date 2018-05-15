---
title: mysql 学习笔记
date: 2017-12-07 10:26:13
tags: mysql
categories: db
---
### mysql 查询重复数据
``` sql
SELECT user_name, COUNT(*) AS c FROM user_table GROUP BY user_name HAVING c > 1;
```
当查询几个列的组合的时候，可以这样
``` sql
SELECT CONCAT(first_name, '_', last_name) AS user_name, COUNT(*) AS c 
FROM user_table 
GROUP BY user_name 
HAVING c > 1;
```
当使用`CONCAT(str1,str2,...)`函数的时候，只要有一个参数为`null`，则整个结果为`null`，而有时候这不是我们所想要的结果，这时候，我们可以使用`IFNULL(expr1,expr2)`函数来做一个转换，当`expr1`为`null`的时候，表达式的结果为`expr2`，否则为`expr1`，示例如下：
``` sql
SELECT CONCAT('Tim', ' ', 'Ho') FROM DUAL;	# Tim Ho
SELECT IFNULL(null, 'expr2') FROM DUAL;		# expr2
SELECT IFNULL('expr1', 'expr2') FROM DUAL;	# expr1
```

### mysql 添加唯一约束
表`t_user`结构如下：
``` sql
Field        Type          Collation        Null    Key     Default            Extra           Privileges                       Comment               
-----------  ------------  ---------------  ------  ------  -----------------  --------------  -------------------------------  ----------------------
id           int(11)       (NULL)           NO      PRI     (NULL)             auto_increment  select,insert,update,references  用户ID              
login_name   varchar(100)  utf8_general_ci  NO              (NULL)                             select,insert,update,references  登录名             
passwd       varchar(100)  utf8_general_ci  NO              (NULL)                             select,insert,update,references  登录密码          
nick_name    varchar(100)  utf8_general_ci  YES             (NULL)                             select,insert,update,references  别名                
gender       tinyint(4)    (NULL)           YES             1                                  select,insert,update,references  性别：1.男；2.女
phone        varchar(20)   utf8_general_ci  YES             (NULL)                             select,insert,update,references  手机号码          
address      varchar(255)  utf8_general_ci  YES             (NULL)                             select,insert,update,references  地址                
create_time  timestamp     (NULL)           YES             CURRENT_TIMESTAMP                  select,insert,update,references  创建时间          
update_time  timestamp     (NULL)           YES             (NULL)                             select,insert,update,references  修改时间
```
 
给 login_name 和 phone 添加联合唯一约束
``` sql
ALTER TABLE t_user ADD UNIQUE KEY (login_name, phone);
```

你也可以在创建唯一约束的时候，给唯一约束起名
``` sql
ALTER TABLE t_user ADD UNIQUE KEY `uk_login_name_phone` (login_name, phone);
```

查看结果：
``` sql
show create table t_user;

CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `login_name` varchar(100) NOT NULL COMMENT '登录名',
  `passwd` varchar(100) NOT NULL COMMENT '登录密码',
  `nick_name` varchar(100) DEFAULT NULL COMMENT '别名',
  `gender` tinyint(4) DEFAULT '1' COMMENT '性别：1.男；2.女',
  `phone` varchar(20) DEFAULT NULL COMMENT '手机号码',
  `address` varchar(255) DEFAULT NULL COMMENT '地址',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NULL DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_login_name_phone` (`login_name`,`phone`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8
```

删除唯一约束:
``` sql
ALTER TABLE t_user DROP INDEX uk_login_name_phone;
```
---
### MySQL中GROUP_CONCAT函数
参考：http://hchmsguo.iteye.com/blog/555543
完整的语法如下：
``` sql
GROUP_CONCAT([DISTINCT] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符'])
```

1. 基本查询
``` sql
select * from aa;  

+------+------+
| id| name |
+------+------+
|1 | 10|
|1 | 20|
|1 | 20|
|2 | 20|
|3 | 200 |
|3 | 500 |
+------+------+
6 rows in set (0.00 sec)
```

2. 以id分组，把name字段的值打印在一行，逗号分隔(默认)
``` sql
select id,group_concat(name) from aa group by id;  

+------+--------------------+
| id| group_concat(name) |
+------+--------------------+
|1 | 10,20,20|
|2 | 20 |
|3 | 200,500|
+------+--------------------+
3 rows in set (0.00 sec)
```

3. 以id分组，把name字段的值打印在一行，分号分隔
``` sql
select id,group_concat(name separator ';') from aa group by id;  

+------+----------------------------------+
| id| group_concat(name separator ';') |
+------+----------------------------------+
|1 | 10;20;20 |
|2 | 20|
|3 | 200;500 |
+------+----------------------------------+
3 rows in set (0.00 sec)
```

4. 以id分组，把去冗余的name字段的值打印在一行，逗号分隔
``` sql
select id,group_concat(distinct name) from aa group by id;  

+------+-----------------------------+
| id| group_concat(distinct name) |
+------+-----------------------------+
|1 | 10,20|
|2 | 20 |
|3 | 200,500 |
+------+-----------------------------+
3 rows in set (0.00 sec)
```

5. 以id分组，把name字段的值打印在一行，逗号分隔，以name排倒序
``` sql
select id,group_concat(name order by name desc) from aa group by id;  

+------+---------------------------------------+
| id| group_concat(name order by name desc) |
+------+---------------------------------------+
|1 | 20,20,10 |
|2 | 20|
|3 | 500,200|
+------+---------------------------------------+
3 rows in set (0.00 sec)
```
---
### GROUP_CONCAT使用及报错分析
函数GROUP_CONCAT，但在使用时，结果报了如下错误。

	ERROR 1260 (HY000): Row 17 was cut by GROUP_CONCAT()
原因：GROUP_CONCAT截断了结果，GROUP_CONCAT有个最大长度的限制，超过最大长度就会被截断掉。可以用下面命令查看其内存限制。
``` sql
SELECT @@global.group_concat_max_len;

+-------------------------------+
| @@global.group_concat_max_len |
+-------------------------------+
|                      1024     |
+-------------------------------+
```
1024这就是一般MySQL系统默认的最大长度，解决限制方法有二：
方法一：在MySQL配置文件中加上
group_concat_max_len = 102400 #你要的最大长度

方法二：可以简单一点，执行语句：
``` sql
SET GLOBAL group_concat_max_len=102400;
```
我是用方法一解决的

### 对连接要注意的事项
当用多个值连接起来产生一个MD5值、或其他用于确定唯一性的时候，几个值之间一定要加分隔符，否则不同记录可能会产生一样的MD5值。如下表：

	NAME	AGE	FAMLY_NUM
	张三	51	5
	李四	5	15
如果用AGE， FAMLY_NUM来产生一个MD5值来唯一确定这一条记录，在它们之间一定要加分隔符，否则连接结果都是515


### 创建数据库
在MYSQL中用`数据库管理员`用户创建了一个db，其他MYSQL用户是暂时看不到的，除非得到`数据库管理员`用户的授权
``` sql
CREATE DATABASE IF NOT EXISTS bfg_db COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';

GRANT ALL ON bfg_db.* TO 'bfg_user'@'%' IDENTIFIED BY 'iE1zNB?A91*YbQ9hK';

GRANT ALL ON bfg_db.* TO 'bfg_user'@'localhost' IDENTIFIED BY 'iE1zNB?A91*YbQ9hK';

FLUSH PRIVILEGES;
```

### mysql添加、删除主键
``` sql
添加自增长的主键id
ALTER TABLE tb ADD PRIMARY KEY(id);
ALTER TABLE tb CHANGE id id INT(10) NOT NULL AUTO_INCREMENT=1;

删除自增长的主键id，先删除自增长，再删除主键
ALTER TABLE tb CHANGE id id int(10); //删除自增长
ALTER TABLE tb DROP PRIMARY KEY; //删除主建
```


### mysql查找删除重复数据并只保留一条
有表数据如下：
``` sql
mysql> select * from t_user;
+------+-------+------+
| id   | name  | age  |
+------+-------+------+
|    1 | zhang |   21 |
|    2 | li    |   22 |
|    3 | zhang |   23 |
+------+-------+------+
3 rows in set (0.00 sec)
```
从上面可知，name字段有两条相同的记录zhang，我们要删除重复记录,保存id最小的一条
``` sql
DELETE 
FROM
	t_user 
WHERE
	name IN ( SELECT t1.name FROM ( SELECT name FROM t_user GROUP BY name HAVING COUNT( * ) > 1 ) t1 ) 
	AND 
	id NOT IN (SELECT t2.id FROM ( SELECT min( id ) id FROM t_user GROUP BY name HAVING count( * ) > 1 ) t2 );
	
然后再执行查询语句：
mysql> select * from t_user;
+------+-------+------+
| id   | name  | age  |
+------+-------+------+
|    1 | zhang |   21 |
|    2 | li    |   22 |
+------+-------+------+
2 rows in set (0.00 sec)
```
可知，id为3的记录已经被删掉了。


### char varchar nvarchar区别
参考：https://www.cnblogs.com/yelaiju/archive/2010/05/29/1746826.html
Unicode字符集就是为了解决字符集的不兼容的问题而产生的，它所有的字符都用两个字节表示，即英文字符也是用两个字节表示

varchar(n)
长度为 n 个字节的可变长度且非 Unicode 的字符数据。n 必须是一个介于 1 和 8,000 之间的数值。存储大小为输入数据的字节的实际长度，而不是 n 个字节。

nvarchar(n)
包含 n 个字符的可变长度 Unicode 字符数据。n 的值必须介于 1 与 4,000 之间。字节的存储大小是所输入字符个数的两倍。

两字段分别有字段值：我和coffee
那么varchar字段占2×2+6=10个字节的存储空间，而nvarchar字段占8×2=16个字节的存储空间。

一般来说，如果含有中文字符，用nchar/nvarchar，如果纯英文和数字，用char/varchar


联机帮助上的：

| 类型			| 长度	| 使用说明	| 长度说明		|
| -				| -		| -			 | -			|
| char(n)		| 定长	| 索引效率高 程序里面使用trim去除多余的空白 		| n 必须是一个介于 1 和 8,000 之间的数值,存储大小为 n 个字节	|
| varchar(n)	| 变长	| 效率没char高 灵活							| n 必须是一个介于 1 和 8,000 之间的数值。存储大小为输入数据的字节的实际长度，而不是 n 个字节	|
| text(n)		| 变长	| 非Unicode数据							  | 不用指定长度	|
| nchar(n)		| 定长	| 处理unicode数据类型(所有的字符使用两个字节表示)	| n 的值必须介于 1 与 4,000 之间。存储大小为 n 字节的两倍	|
| nvarchar(n)	| 变长	| 处理unicode数据类型(所有的字符使用两个字节表示)	| n 的值必须介于 1 与 4,000 之间。字节的存储大小是所输入字符个数的两倍。所输入的数据字符长度可以为零	|
| ntext(n)		| 变长	| 处理unicode数据类型(所有的字符使用两个字节表示)	| 不用指定长度	|


很多开发者进行数据库设计的时候往往并没有太多的考虑char， varchar类型，有的是根本就没注意，因为存储价格变得越来越便宜了，忘记了最开始的一些基本设计理论和原则，这点让我想到了现在的年轻人，大手一挥一把人民币就从他手里溜走了，其实我想不管是做人也好，做开发也好，细节的把握直接决定很多东西。当然还有一部分人是根本就没弄清楚他们的区别，也就随便选一个。在这里我想对他们做个简单的分析，当然如果有不对的地方希望大家指教。
1、CHAR。CHAR存储定长数据很方便，CHAR字段上的索引效率级高，比如定义char(10)，那么不论你存储的数据是否达到了10个字节，都要占去10个字节的空间,不足的自动用空格填充，所以在读取的时候可能要多次用到trim（）。

2、VARCHAR。存储变长数据，但存储效率没有CHAR高。如果一个字段可能的值是不固定长度的，我们只知道它不可能超过10个字符，把它定义为 VARCHAR(10)是最合算的。VARCHAR类型的实际长度是它的值的实际长度+1。为什么“+1”呢？这一个字节用于保存实际使用了多大的长度。从空间上考虑，用varchar合适；从效率上考虑，用char合适，关键是根据实际情况找到权衡点。

3、TEXT。text存储可变长度的非Unicode数据，最大长度为2^31-1(2,147,483,647)个字符。

4、NCHAR、NVARCHAR、NTEXT。这三种从名字上看比前面三种多了个“N”。它表示存储的是Unicode数据类型的字符。我们知道字符中，英文字符只需要一个字节存储就足够了，但汉字众多，需要两个字节存储，英文与汉字同时存在时容易造成混乱，Unicode字符集就是为了解决字符集这种不兼容的问题而产生的，它所有的字符都用两个字节表示，即英文字符也是用两个字节表示。nchar、nvarchar的长度是在1到4000之间。和char、varchar比较起来，nchar、nvarchar则最多存储4000个字符，不论是英文还是汉字；而char、varchar最多能存储8000个英文，4000个汉字。可以看出使用nchar、nvarchar数据类型时不用担心输入的字符是英文还是汉字，较为方便，但在存储英文时数量上有些损失。

所以一般来说，如果含有中文字符，用nchar/nvarchar，如果纯英文和数字，用char/varchar

它们的区别概括成：
CHAR，NCHAR 定长，速度快，占空间大，需处理
VARCHAR，NVARCHAR，TEXT 不定长，空间小，速度慢，无需处理
NCHAR、NVARCHAR、NTEXT处理Unicode码


### mysql 修改字段长度
``` sql
alter table [表名] modify column [字段名] [类型];

示例: sys_user表里的user_name字段，由原来长度是10个字符，修改改成40个字符
alter table sys_user modify column user_name varchar(40);
```


### mysql 增加列（字段）
``` sql
alter table [表名] add [列名] [列类型] [其他属性，如默认值];

示例: sys_user表增加一个地址列，长度为200个字符，默认值为null
alter table sys_user add address varchar(200) default null;
```


### mysql 按字段分组，取最小生日记录
有用户表如下：
``` sql
sql> select * from user;

id	name	birthday
1	a	2018-01-25 09:30:05
2	a	2018-01-25 09:30:14
3	a	2018-01-25 09:30:10
4	b	2018-01-25 09:30:29
5	b	2018-01-25 09:30:24
```
**要求：**以name分组，取出生日最小的记录，即id为2、4的记录
``` sql
sql> select * from (select * from user order by birthday desc) t group by t.name;

id	name	birthday
2	a	2018-01-25 09:30:14
4	b	2018-01-25 09:30:29
```

### mysql 仅保留 1000 条记录而删除其他记录
``` sql
求取总记录数
select count(*) from tb_name;

删除部分记录
delete from tb_name limit 总记录数-1000

例如，比如一个表有 10000 条记录，现想保留 1000 条数据
delete from tb_name limit 9000
```


### MySQL数据库备份和还原常用的命令
#### 一、备份命令
如果只需要导出表的结构,可以使用mysqldump的`-d`选项
``` sql
1、备份MySQL数据库的命令，备份指定的整个库
mysqldump -hhostname -uusername -ppassword databasename > ~/backupfile.sql

2、备份MySQL数据库为带删除表的格式，能够让该备份覆盖已有数据库而不需要手动删除原有数据库
mysqldump --add-drop-table -hhostname -uusername -ppassword databasename > ~/backupfile.sql

3、直接将MySQL数据库压缩备份
mysqldump -hhostname -uusername -ppassword databasename | gzip > ~/backupfile.sql.gz

4、备份MySQL数据库某个(些)表
mysqldump -hhostname -uusername -ppassword databasename specific_table1 [specific_table2] > ~/backupfile.sql

5、同时备份多个MySQL数据库
mysqldump -hhostname -uusername -ppassword -databases databasename1 databasename2 databasename3 > ~/multibackupfile.sql

6、仅仅备份数据库结构
mysqldump -no-data -databases databasename1 databasename2 databasename3 > ~/structurebackupfile.sql

7、备份服务器上所有数据库
mysqldump -all-databases > ~/allbackupfile.sql
```

#### 二、还原命令
``` sql
1、还原MySQL数据库的命令
mysql -hhostname -uusername -ppassword databasename < ~/backupfile.sql

2、还原压缩的MySQL数据库
gunzip < ~/backupfile.sql.gz | mysql -uusername -ppassword databasename

3、将数据库转移到新服务器
mysqldump -uusername -ppassword databasename | mysql -host=*.*.*.* -C databasename
```


### select into
除此之外，你还可以使用`select into`命令来导出数据，它与mysqldump的区别是：

1. mysqldump: 导出的文本包括了数据库的结构和记录；
2. select into: 导出的文本只有记录，并且，一般要数据库的管理员帐号，例如root才能执行；

select into 有两种使用方式：
``` sql
方式一：
mysql> select * from t_user into outfile 't_user.txt';
Query OK, 2 rows affected (0.16 sec)

方式二：
mysql> select * into outfile 't_user.txt' from t_user;
Query OK, 2 rows affected (0.00 sec)
```
导出的数据一般在数据库的安装目录下，你也可以使用`find / -name t_user.txt`来查找到


### ERROR 3 (HY000): Error writing file '/tmp/MYeaZGpS' (Errcode: 28 - No space left on device)
根据提示，是说mysql中/tmp使用的磁盘空间不足。
在MYSQL命令行下执行：
``` sql
mysql> show variables like 'tmpdir';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| tmpdir        | /tmp  |
+---------------+-------+
1 row in set (0.01 sec)
```
可以看到MYSQL的tmpdir是使用/tmp这个临时目录的，使用`df -h`看下/tmp的空间，确实不足。我们修改tmpdir所指定的目录，我们在空间比较大的分区上面建一个目录。
``` bash
$ cd /opt/data
$ mkdir mysqltmp
$ chmod a+w mysqltmp
```
然后修改my.cnf配置文件在其中修改tmpdir 
``` bash
$ whereis my.cnf
$ cd {my.cnf所在的目录}
$ vi my.cnf
# 修改的内容如下
tmpdir = /opt/data/mysqltmp
```

重启mysql
``` bash
$ /etc/init.d/mysqld restart	# 注意linux系统，命令可能不同
```
在MYSQL命令行中查看下是否生效
``` sql
mysql> show variables like 'tmpdir';
+---------------+--------------------+
| Variable_name | Value              |
+---------------+--------------------+
| tmpdir        | /opt/data/mysqltmp |
+---------------+--------------------+
1 row in set (0.00 sec)
```
问题解决。


读一致性：ORACLE有一个表，有1000W条记录。在9：00的时候A用户对表进行查询，查询需10分钟才能完成，表没索引，FULL SCAN，表中某条记录的值为100。在9：05分的时候B对表进行了UPDATE操作，将记录的值修改为200。那么在9:10的时候，A获致到的是100还是200？结果是100.因为这是读一致性的问题，在A查的时候，它看到的是整个表在那一刻的快照的数据，返回的是那一刻的结果。不过，它有可能会抛出 snapshot too old 这个异常。


### mysql 修改表名
``` mysql
ALTER TABLE table_name RENAME TO new_table_name;
```


### MySQL 百万级分页优化(Mysql千万级快速分页)
转自：http://www.jb51.net/article/31868.htm

一般刚开始学SQL的时候，会这样写，代码如下:
``` mysql
SELECT * FROM table ORDER BY id LIMIT 1000, 10; 
```
但在数据达到百万级的时候，这样写会慢死，代码如下:
``` mysql
SELECT * FROM table ORDER BY id LIMIT 1000000, 10; 
```
也许耗费几十秒 

网上很多优化的方法是这样的，代码如下:
``` mysql
SELECT * FROM table WHERE id >= (SELECT id FROM table LIMIT 1000000, 1) LIMIT 10; 
```
是的，速度提升到0.x秒了，看样子还行了 
可是，还不是完美的！ 

以下这句才是完美的！代码如下:
``` mysql
SELECT * FROM table WHERE id BETWEEN 1000000 AND 1000010; 
```
比上面那句，还要再快5至10倍 

另外，如果需要查询 id 不是连续的一段，最佳的方法就是先找出 id ，然后用 in 查询，代码如下:
``` mysql
SELECT * FROM table WHERE id IN(10000, 100000, 1000000...); 
```

再分享一点 
查询字段一较长字符串的时候，表设计时要为该字段多加一个字段,如，存储网址的字段，查询的时候，不要直接查询字符串，效率低下，应该查诡该字串的crc32或md5 


### mysql 查询一个字符串字段 最长的一条记录
参考：http://stackoverflow.com/questions/2357620/maxlengthfield-in-mysql
``` mysql
select name, length( name )
from my_table
where length( name ) = ( select max( length( name ) ) from my_table );
```


