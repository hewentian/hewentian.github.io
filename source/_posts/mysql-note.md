---
title: mysql 学习笔记
date: 2017-12-07 10:26:13
tags: mysql
categories: db
---

### mysql 查询重复数据
``` sql
SELECT user_name, COUNT(*) AS c 
FROM user_table 
GROUP BY user_name 
HAVING c > 1;
```

当查询几个列的组合的时候，可以这样
``` sql
SELECT CONCAT(first_name, '_', last_name) AS user_name, COUNT(*) AS c 
FROM user_table 
GROUP BY user_name 
HAVING c > 1;
```

当使用`CONCAT(str1,str2,...)`函数的时候，只要有一个参数为`NULL`，则整个结果为`NULL`，而有时候这不是我们所想要的结果，这时候，我们可以使用`IFNULL(expr1,expr2)`函数来做一个转换，当`expr1`为`NULL`的时候，表达式的结果为`expr2`，否则为`expr1`，示例如下：
``` sql
SELECT CONCAT('Tim', ' ', 'Ho') FROM DUAL; # Tim Ho
SELECT IFNULL(NULL, 'expr2') FROM DUAL;    # expr2
SELECT IFNULL('expr1', 'expr2') FROM DUAL; # expr1
```


### 索引
显示索引信息：
``` sql
SHOW INDEX FROM tableName;
```

创建索引，语法一：
``` sql
ALTER TABLE tableName ADD INDEX indexName(columnName);

eg: ALTER TABLE t_user ADD INDEX index_name(name);
```

创建索引，语法二：
``` sql
CREATE INDEX indexName ON tableName(columnName(length));

eg: CREATE INDEX index_sex ON t_user(sex);
```
如果是CHAR，VARCHAR类型，length可以小于字段实际长度或者省略；如果是BLOB和TEXT类型，必须指定length。

创建索引，语法三：
创建表的时候直接指定
``` sql
CREATE TABLE tableName (
ID INT NOT NULL,
columnName VARCHAR(16) NOT NULL,
INDEX [indexName] (columnName(length))
);
```

删除索引：
``` sql
DROP INDEX [indexName] ON tableName;
```


### 唯一索引
创建索引，语法一：
``` sql
ALTER TABLE tableName ADD UNIQUE [indexName] (columnName)

eg: ALTER TABLE t_user ADD UNIQUE index_name (name);
```

创建索引，语法二：
``` sql
CREATE UNIQUE INDEX indexName ON tableName(columnName(length))

eg: CREATE UNIQUE INDEX index_name ON t_user(name);
```


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
SHOW CREATE TABLE t_user;

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


### LIMIT 语句
取得某一范围的记录集。如前5条、第5到第10条，数据分页等。LIMIT写在查询语句的最后位置上。

    语法：
    SELECT * FROM tableName LIMIT 长度;
    SELECT * FROM tableName LIMIT 起始位置, 长度;

``` sql
SELECT * FROM t_user WHERE id > 2 LIMIT 5;     // 前5条记录
SELECT * FROM t_user WHERE id > 2 LIMIT 5, 10; // 第5条记录之后的前10条记录，不包括第5条记录
```


### MySQL中GROUP_CONCAT函数
https://www.mysqltutorial.org/mysql-group_concat/

完整的语法如下：
``` sql
GROUP_CONCAT(
    DISTINCT expression
    ORDER BY expression
    SEPARATOR sep
);
```

1. 基本查询
``` sql
SELECT * FROM aa;

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
SELECT id, GROUP_CONCAT(name) FROM aa GROUP BY id;

+------+--------------------+
| id| GROUP_CONCAT(name) |
+------+--------------------+
|1 | 10,20,20|
|2 | 20 |
|3 | 200,500|
+------+--------------------+
3 rows in set (0.00 sec)
```

3. 以id分组，把name字段的值打印在一行，分号分隔
``` sql
SELECT id, GROUP_CONCAT(name separator ';') FROM aa GROUP BY id;

+------+----------------------------------+
| id| GROUP_CONCAT(name separator ';') |
+------+----------------------------------+
|1 | 10;20;20 |
|2 | 20|
|3 | 200;500 |
+------+----------------------------------+
3 rows in set (0.00 sec)
```

4. 以id分组，把去冗余的name字段的值打印在一行，逗号分隔
``` sql
SELECT id, GROUP_CONCAT(distinct name) FROM aa GROUP BY id;

+------+-----------------------------+
| id| GROUP_CONCAT(distinct name) |
+------+-----------------------------+
|1 | 10,20|
|2 | 20 |
|3 | 200,500 |
+------+-----------------------------+
3 rows in set (0.00 sec)
```

5. 以id分组，把name字段的值打印在一行，逗号分隔，以name排倒序
``` sql
SELECT id, GROUP_CONCAT(name ORDER BY name DESC) FROM aa GROUP BY id;

+------+---------------------------------------+
| id| GROUP_CONCAT(name ORDER BY name DESC) |
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

    NAME    AGE    FAMLY_NUM
    张三    51     5
    李四    5      15
如果用AGE， FAMLY_NUM来产生一个MD5值来唯一确定这一条记录，在它们之间一定要加分隔符，否则连接结果都是515


### mysql禁用root远程登录
``` bash
mysql> DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
mysql> FLUSH PRIVILEGES;
```


### 创建数据库
在MYSQL中用`数据库管理员`用户创建了一个db，其他MYSQL用户是暂时看不到的，除非得到`数据库管理员`用户的授权
``` sql
CREATE DATABASE IF NOT EXISTS bfg_db COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8';

GRANT ALL ON bfg_db.* TO 'bfg_user'@'%' IDENTIFIED BY 'gXk9IDpybrJPVMKq';

GRANT ALL ON bfg_db.* TO 'bfg_user'@'localhost' IDENTIFIED BY 'gXk9IDpybrJPVMKq';

FLUSH PRIVILEGES;
```


### 数据库基本操作
删除数据库
``` sql
DROP DATABASE databaseName;
```

显示所有数据库
``` sql
SHOW DATABASES;
```

进入某一个数据库
``` sql
USE databaseName;
```

显示所有表
``` sql
SHOW TABLES;
```

查看指定表的结构
``` sql
DESC tableName;
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


### 存储过程
创建插入记录的存储过程：
``` mysql
DELIMITER $$

DROP PROCEDURE IF EXISTS `proc_insert_student`$$

CREATE PROCEDURE `proc_insert_student`(_sname VARCHAR(20), _sex TINYINT(4), _age INT)
BEGIN
    INSERT INTO student(sname, sex, age) VALUES(_sname, _sex, _age);
END$$

DELIMITER ;
```

创建统计数据的存储过程：
``` mysql
DELIMITER $$

DROP PROCEDURE IF EXISTS `proc_count_student`$$

CREATE PROCEDURE `proc_count_student`(OUT cs INT)
BEGIN
    # CALL proc_count_student(@cs)
    SELECT COUNT(*) INTO cs FROM student;
END$$

DELIMITER ;
```


查看数据库中的存储过程
``` sql
mysql> SHOW PROCEDURE STATUS;
+--------+---------------------+-----------+------------+---------------------+---------------------+---------------+---------+----------------------+----------------------+--------------------+
| Db     | Name                | Type      | Definer    | Modified            | Created             | Security_type | Comment | character_set_client | collation_connection | Database Collation |
+--------+---------------------+-----------+------------+---------------------+---------------------+---------------+---------+----------------------+----------------------+--------------------+
| bfg_db | proc_count_student  | PROCEDURE | bfg_user@% | 2019-10-21 17:33:15 | 2019-10-21 17:33:15 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
| bfg_db | proc_insert_student | PROCEDURE | bfg_user@% | 2019-10-21 17:21:38 | 2019-10-21 17:21:38 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
+--------+---------------------+-----------+------------+---------------------+---------------------+---------------+---------+----------------------+----------------------+--------------------+
2 rows in set (0.02 sec)
```


### mysql 删除记录
``` sql
DELETE FROM t_user;
DELETE FROM t_user WHERE id > 1;
```


### mysql 仅保留 1000 条记录而删除其他记录
``` sql
求取总记录数
SELECT COUNT(*) FROM tb_name;

删除部分记录
DELETE FROM tb_name LIMIT 总记录数-1000

例如，比如一个表有 10000 条记录，现想保留 1000 条数据
DELETE FROM tb_name LIMIT 9000
```


### mysql查找删除重复数据并只保留一条
有表数据如下：
``` sql
mysql> SELECT * FROM t_user;
+------+-------+------+
| id   | name  | age  |
+------+-------+------+
|    1 | zhang |   21 |
|    2 | li    |   22 |
|    3 | zhang |   23 |
+------+-------+------+
3 rows in set (0.00 sec)
```
从上面可知，name字段有两条相同的记录zhang，我们要删除重复记录，保存id最小的一条
``` sql
DELETE 
FROM
    t_user 
WHERE
    name IN (SELECT t1.name FROM (SELECT name FROM t_user GROUP BY name HAVING COUNT(*) > 1) t1 ) 
    AND 
    id NOT IN (SELECT t2.id FROM (SELECT MIN(id) id FROM t_user GROUP BY name HAVING COUNT(*) > 1) t2 );

然后再执行查询语句：
mysql> SELECT * FROM t_user;
+------+-------+------+
| id   | name  | age  |
+------+-------+------+
|    1 | zhang |   21 |
|    2 | li    |   22 |
+------+-------+------+
2 rows in set (0.00 sec)
```
可知，id为3的记录已经被删掉了。


### MySQL中CHAR和VARCHAR的区别
| CHAR | VARCHAR |
| :-----| :----- |
| 全称是 CHARACTER | 全称是 VARIABLE CHARACTER |
| 存储固定长度的数据，如果不足，会用空格补全，读取的时候可能会调用trim() | 按数据的实际长度存储，不会用空格补全 |
| 占空间大 | 占空间小 |
| 最大能存储255个字符 | 最大能存储65535个字符 |
| 静态内存分配 | 动态内存分配 |
| 索引效率高 | 索引效率没CHAR高 |

**VARCHAR存储的实际长度是它的值的长度+1，多出的这一个用于保存它实际使用了多大的长度。**


### 复制表
语法一，会复制完整的表结构（包括索引、主键等），但不包括数据：
``` sql
CREATE TABLE tableName_new LIKE tableName_old;
```

语法二，只会复制简单的表结构（不包括索引、主键等），但可以包括数据（根据FROM后的WHERE条件）：
``` sql
CREATE TABLE tableName_new AS SELECT * FROM tableName_old;
```


### mysql 修改字段定义
下面的关键字`COLUMN`是可以省略的：
``` sql
ALTER TABLE [表名] MODIFY COLUMN [字段名] [类型];

示例: sys_user表里的user_name字段，由原来长度是10个字符，修改改成40个字符
ALTER TABLE sys_user MODIFY COLUMN user_name VARCHAR(40);
```


### mysql 修改字段名
``` sql
ALTER TABLE [表名] CHANGE [旧字段名] [新字段名] [类型];

示例: sys_user表里的user_name字段，修改改成user_name_new
ALTER TABLE sys_user CHANGE user_name user_name_new VARCHAR(40);
```


### mysql 增加列（字段）
``` sql
ALTER TABLE [表名] ADD [列名] [列类型] [其他属性，如默认值];

示例: sys_user表增加一个地址列，长度为200个字符，默认值为null
ALTER TABLE sys_user ADD address VARCHAR(200) DEFAULT NULL;
ALTER TABLE sys_user ADD address VARCHAR(200) DEFAULT NULL AFTER age;

可以一次增加多个列：
ALTER TABLE sys_user ADD (
  sex VARCHAR(2) DEFAULT '男',
  tel VARCHAR(11)
);

如果新增加的列有默认值，则SQL执行后，该新增的列会自动获得该值。
```


### mysql 删除列（字段）
语法： 
``` sql
ALTER TABLE tableName DROP columnName;
```


### mysql 删除表
语法： 
``` sql
DROP TABLE tableName;
```


### mysql 分组统计
表结构和数据如下：
``` sql
CREATE TABLE t_user (
id INT PRIMARY KEY, 
name VARCHAR(20), 
salary INT,
birthday TIMESTAMP
);


sql> INSERT INTO t_user(id, name, salary, birthday) VALUES (1, "a", 4000, STR_TO_DATE("2018-01-25 09:30:05","%Y-%m-%d %h:%i:%s"));
sql> INSERT INTO t_user(id, name, salary, birthday) VALUES (2, "a", 2000, STR_TO_DATE("2018-01-25 09:30:14","%Y-%m-%d %h:%i:%s"));
sql> INSERT INTO t_user(id, name, salary, birthday) VALUES (3, "a", 3000, STR_TO_DATE("2018-01-25 09:30:10","%Y-%m-%d %h:%i:%s"));
sql> INSERT INTO t_user(id, name, salary, birthday) VALUES (4, "b", 6000, STR_TO_DATE("2018-01-25 09:30:29","%Y-%m-%d %h:%i:%s"));
sql> INSERT INTO t_user(id, name, salary, birthday) VALUES (5, "b", 5000, STR_TO_DATE("2018-01-25 09:30:24","%Y-%m-%d %h:%i:%s"));


sql> SELECT * FROM t_user;
+----+------+--------+---------------------+
| id | name | salary | birthday            |
+----+------+--------+---------------------+
|  1 | a    |   4000 | 2018-01-25 09:30:05 |
|  2 | a    |   2000 | 2018-01-25 09:30:14 |
|  3 | a    |   3000 | 2018-01-25 09:30:10 |
|  4 | b    |   6000 | 2018-01-25 09:30:29 |
|  5 | b    |   5000 | 2018-01-25 09:30:24 |
+----+------+--------+---------------------+
```

1. 按用户名分组，查询salary最大的记录
``` sql
sql> SELECT id, name, MAX(salary) FROM t_user GROUP BY name;

+----+------+-------------+
| id | name | MAX(salary) |
+----+------+-------------+
|  1 | a    |        4000 |
|  4 | b    |        6000 |
+----+------+-------------+


或者
sql > SELECT * FROM (SELECT * FROM t_user ORDER BY salary DESC) t GROUP BY t.name;

+----+------+--------+---------------------+
| id | name | salary | birthday            |
+----+------+--------+---------------------+
|  1 | a    |   4000 | 2018-01-25 09:30:05 |
|  4 | b    |   6000 | 2018-01-25 09:30:29 |
+----+------+--------+---------------------+
```

2. 按用户名分组，取生日最小的记录
``` sql
sql> SELECT * FROM (SELECT * FROM t_user ORDER BY birthday DESC) t GROUP BY t.name;

+----+------+--------+---------------------+
| id | name | salary | birthday            |
+----+------+--------+---------------------+
|  2 | a    |   2000 | 2018-01-25 09:30:14 |
|  4 | b    |   6000 | 2018-01-25 09:30:29 |
+----+------+--------+---------------------+
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
mysqldump -hhostname -uusername -ppassword --databases databasename1 databasename2 databasename3 > ~/multibackupfile.sql

6、仅仅备份数据库结构
mysqldump -hhostname -uusername -ppassword -d --databases databasename1 databasename2 databasename3 > ~/structurebackupfile.sql

7、备份服务器上所有数据库
mysqldump -hhostname -uusername -ppassword --all-databases > ~/allbackupfile.sql

8、备份MySQL数据库某个表中指定条件的数据
mysqldump -hhostname -uusername -ppassword databasename specific_table1 --where='age > 10' > ~/backupfile.sql
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


### SELECT INTO
除此之外，你还可以使用`SELECT INTO`命令来导出数据，它与mysqldump的区别是：

1. mysqldump: 导出的文本包括了数据库的结构和记录；
2. SELECT INTO: 导出的文本只有记录，并且，一般要数据库的管理员帐号，例如root才能执行；

SELECT INTO 有两种使用方式：
``` sql
方式一：
mysql> SELECT * FROM t_user INTO OUTFILE 't_user.txt';
Query OK, 2 rows affected (0.16 sec)

方式二：
mysql> SELECT * INTO OUTFILE 't_user.txt' FROM t_user;
Query OK, 2 rows affected (0.00 sec)
```
导出的数据一般在数据库的安装目录下，你也可以使用`find / -name t_user.txt`来查找到


### ERROR 3 (HY000): Error writing file '/tmp/MYeaZGpS' (Errcode: 28 - No space left on device)
根据提示，是说mysql中/tmp使用的磁盘空间不足。
在MYSQL命令行下执行：
``` sql
mysql> SHOW VARIABLES LIKE 'tmpdir';
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

修改的内容如下
tmpdir = /opt/data/mysqltmp
```

重启mysql
``` bash
$ /etc/init.d/mysqld restart    # 不同Linux系统，命令可能不同
```
在MYSQL命令行中查看下是否生效
``` sql
mysql> SHOW VARIABLES LIKE 'tmpdir';
+---------------+--------------------+
| Variable_name | Value              |
+---------------+--------------------+
| tmpdir        | /opt/data/mysqltmp |
+---------------+--------------------+
1 row in set (0.00 sec)
```
问题解决。


### 读一致性
mysql有一个表，有1000W条记录。在9：00的时候A用户对表进行查询，查询需10分钟才能完成，表没索引，FULL SCAN，表中某条记录的值为100。在9：05分的时候B对表进行了UPDATE操作，将记录的值修改为200。那么在9:10的时候，A获致到的是100还是200？结果是100。因为这是读一致性的问题，在A查的时候，它看到的是整个表在那一刻的快照的数据，返回的是那一刻的结果。不过，它有可能会抛出 snapshot too old 这个异常。


### mysql 修改表名
``` mysql
ALTER TABLE table_name RENAME TO new_table_name;
```


### INSERT INTO插入数据
如果不指定插入的列名，则VALUES的值必须与表的所有列名一一对应
``` sql
CREATE TABLE t_user (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(20),
  age INT
);

INSERT INTO t_user (name) VALUES ('Scott');
INSERT INTO t_user VALUES (NULL, 'Scott', 20);
INSERT INTO t_user VALUES (NULL, 'Scott', 20), (NULL, 'tiger', 21);
```


### MYSQL插入数据时忽略重复数据的方法
使用`IGNORE`关键字，示例：
``` mysql
INSERT IGNORE INTO tableName(column_list)
VALUES (value_list),
       (value_list),
       ...
```


### 将一个数据库中的表导入到另一个数据库中的表
将数据库A中的表a的数据导入到数据库B中的表b中。
``` mysql
INSERT [IGNORE] INTO 数据库B.`表名b` SELECT * FROM 数据库A.`表名a`;
INSERT [IGNORE] INTO 数据库B.`表名b` SELECT col1, col2, col3 FROM 数据库A.`表名a`;
INSERT [IGNORE] INTO 数据库B.`表名b`(col1, col2, col3) SELECT col1, col2, col3 FROM 数据库A.`表名a`;
```


### MySQL千万级数据快速分页
数据量少的时候，可以这样写：
``` mysql
SELECT * FROM tableName ORDER BY id LIMIT 1000, 10; 
```

但在数据量达到百万级，甚至千万级的时候，这样写会慢死：
``` mysql
SELECT * FROM tableName ORDER BY id LIMIT 1000000, 10; 
```

可能耗费几十秒，网上有很多优化的方法是这样的：
``` mysql
SELECT * FROM tableName WHERE id >= (SELECT id FROM tableName LIMIT 1000000, 1) LIMIT 10; 
```

是的，速度提升到0.x秒了，看样子还行。但，还不是完美的！下面这句才是完美的：
``` mysql
SELECT * FROM tableName WHERE id BETWEEN 1000000 AND 1000010; 
```
比上面那句，还要再快5至10倍

另外，如果需要查询`id`不是连续的一段，最佳的方法就是先找出`id`，然后用`IN`查询，代码如下：
``` mysql
SELECT * FROM tableName WHERE id IN (10000, 100000, 1000000, ...); 
```

如果查询的字段是一较长字符串的时候，表设计时要为该字段多加一个字段。如，存储网址的字段，查询的时候，不要直接查询该字符串，这样效率低下，应该查询该字符串的CRC32或MD5。


### mysql 查询一个字符串字段最长的一条记录
参考：http://stackoverflow.com/questions/2357620/maxlengthfield-in-mysql
``` mysql
SELECT name, LENGTH(name)
FROM my_table
WHERE LENGTH(name) = (SELECT MAX(LENGTH(name)) FROM my_table);
```


### skip-grant-tables 参数的使用
这是个非常有用的mysql启动参数

在my.cnf文件中增加一行：
``` xml
[mysqld]
skip-grant-tables
```

或者以命令行参数启动mysql：

        /usr/bin/mysqld_safe --skip-grant-tables &

登陆mysql修改管理员密码：
``` mysql
USE mysql;
UPDATE user SET password=password('root') WHERE user='root';
FLUSH PRIVILEGES;
EXIT;
```

重启mysql


### DELETE和TRUNCATE的区别
TRUNCATE TABLE 属于DDL，它虽然和对整张表执行DELETE操作产生的效果差不多，但是它是不能回滚的。


### 三大范式
第一范式(1NF)：在关系模式R中的每一个具体关系r，必须要有主键，并且每个属性值都是不可再分的最小数据单位，则称R是第一范式的关系；
第二范式(2NF)：如果关系模式R中的所有非主属性都完全依赖于主关键字，则称关系R是属于第二范式的；
第三范式(3NF)：关系模式R中的非主关键字不能依赖于其他非主关键字。即非主关键字之间不能有函数（传递）依赖关系，则称关系R是属于第三范式的。


### VARCHAR类型的说明
在官方手册中，VARCHAR类型最大支持65535，单位是字节，但实际上。在创建表的时候VARCHAR(N)中的N，指的是字符的长度。但是，如果表的字符集不同，能支持的最大长度也不同。

1. 表的CHARSET=latin1时，能使用VARCHAR(65532)；
2. 表的CHARSET=GBK时，能使用VARCHAR(32767)；
3. 表的CHARSET=UTF8，能使用VARCHAR(21845)；
4. 如果表的SQL_MODE为非严格模式，则表的CHARSET=latin1时，或者能使用VARCHAR(65535)成功创建表，但是可能会有warning。warning可能是数据库将那列自动转换为TEXT类型了；
5. 还有一个需要注意的地方是，65535长度是指表中所有VARCHAR列的长度总和。如果列的长度总和超出这个限制，依然无法创建表。


### CHAR类型的说明
CHAR(N)中的N指的是字符，对于多字节字符编码的CHAR数据类型，在InnoDB存储引擎中，会将其视为变长类型。对于未能占满长度的字符，还是填充`0X20`，也即是空格。

在多字节字符集的情况下，CHAR和VARCHAR的实际行存储基本是没有区别的。


### 强制使用索引
在查询的过程中可以使用`FORCE INDEX`来强制查询优化器使用指定的索引，例如表`t_user`有索引`idx_name`，则我们可以这样查询：
``` mysql
SELECT * FROM t_user FORCE INDEX(idx_name) WHERE name='Tim' AND age=22;
```


### 索引提示
在查询的过程中可以使用`USE INDEX`来显式地告诉查询优化器使用哪个索引，例如表`t_user`有索引`idx_name`，则我们可以这样查询：
``` mysql
SELECT * FROM t_user USE INDEX(idx_name) WHERE name='Tim' AND age=22;
```

主要在以下2种情况下要使用索引提示：
1. MYSQL数据库的优化器错误地选择了某个索引，导致SQL语句运行得很慢；
2. 某SQL语句可以选择的索引非常多，优化器选择执行计划时间的开销大于SQL语句本身。

`USE INDEX`只是告诉优化器可以选择该索引，但实际上优化器可能还是会再根据自已的判断进行选择，除非使用`FORCE INDEX`。


对数据库执行了一个删除操作，实际上并不会删除索引中的数据，相反还会在对应的DELETED表中插入记录，因此随着应用程序的运行，索引会变得非常大。即使索引中的有些数据已经被删除，查询也不会选择这类记录。InnoDb存储引擎，可以在必要时手动将已经删除的记录在索引中删除，命令是 OPTIMIZE TABLE。它还会对Cardinality进行统计，因此可以关闭它的统计。
``` mysql
mysql> SET GLOBAL innodb_optimize_fulltext_only=1;
mysql> OPTIMIZE TABLE table_name;
```

### 锁问题
锁问题有三个：
1. 脏读：首先了解一下脏数据，脏数据是指事务对缓冲池中行记录的修改，并且还没有被提交。如果读到了脏数据，即一个事务可以读到另外一个事务中未提交的数据，这显然违反了数据库的隔离性。而对脏页的读取，是非常正常的。脏页是因为数据库实例内存和磁盘的异步造成的，这并不影响数据的一致性，因为内存中的数据最终是要刷新回磁盘的。

2. 不可重复读：在同一个事务中多次读取同一个表，在多次读取之间，还有另外一个事务对该表进行了DML，从而导致多次读取到的数据可能不一致。与脏读的区别是：脏读读取到的是未提交的数据，而不可重复读读到的是已经提交的数据。这违反了数据库事务的一致性。

3. 丢失更新：就是一个事务的更新操作，会被另一个事务的更新操作所覆盖，从而导致数据的不一致。可以将事务的处理模式变成串行的，这样来避免。在更新数据的时候，特别是敏感数据的时候，一定要先加一个排他X锁，这样也方便对帐户余额进行先检查，再更新。而不是仅仅学会了简单的SELECT、UPDATE操作就开始处理数据。
``` mysql
SELECT cash FROM account WHERE user='user_name' FOR UPDATE;
```


### 事务的ACID特性
    事务是作为一个逻辑单元执行的一系列操作，一个逻辑工作单元必须有四个属性，
称为 ACID（原子性、一致性、隔离性和持久性）属性，只有这样才能成为一个事务。

原子性（Atomicity）
整个事务中的所有操作，要么全部成功执行，要么全部失败。

一致性（Consistency）
事务提交前后，表的所有约束条件都保持完好。对表的修改符合所有的Keys、数据类型、Checks和触发器的约束，没有约束被破坏。

隔离性（Isolation）
一个事务，无法访问到另一个事务未提交的数据。

持久性（Durability）
事务成功执行后，对数据的修改会持久保存在数据库中，不会被回滚，就算系统死机。


### 事务的隔离级别
对于InnoDB存储引擎而言，它默认的事务隔离级别为READ REPEATABLE，完全遵循和满足事务的ACID特性。

SQL标准定义了4个隔离级别：
READ UNCOMMITTED：会出现脏读、不可重复读、幻读（隔离级别最低，并发性能高）
READ COMMITTED：会出现不可重复读、幻读问题（锁定正在读取的行）
REPEATABLE READ：会出幻读（锁定所读取的所有行）
SERIALIZABLE：保证所有的情况不会发生（锁表）

可以在数据库启动的时候设置它的默认隔离级别：
``` mysql
[mysqld]
transaction-isolation=READ-COMMITTED
```

查看当前会话的事务隔离级别：
``` mysql
mysql> SELECT @@tx_isolation;
```


### 分布式事务
InnoDB存储引擎提供了对XA事务的支持，并通过XA事务来支持分布式事务的实现。分布式事务指的是允许多个独立的事务资源参与到一个全局的事务中。事务资源通常是关系型数据库系统，但也可以是其他类型的资源。全局事务要求在其中的所有参与事务要么都提交，要么都ROLLBACK，这对于事务原有的ACID要求又有了提高。另外，在使用分布式事务时，InnoDB存储引擎的事务隔离级别必须设置为SERIALIZABLE。
XA事务允许不同数据库之间的分布式事务，如一台服务器是MySQL数据库，另一台是Oracle数据库，只要参与在全局事务中的每个节点都支持XA事务。分布式事务可能在银行系统的转帐中比较常见。可以通过变量查看数据库是否启用了XA事务支持（默认为ON）：
``` mysql
mysql> SHOW VARIABLES LIKE 'innodb_support_xa';
```

MySQL数据库是自动提交的。当用户显式的使用命令`START TRANSACTION`或`BEGIN`来开启一个事务时，MySQL会自动地执行`SET AUTOCOMMIT=0`命令，并在`COMMIT`或`ROLLBACK`结束一个事务后，再执行`SET AUTOCOMMIT=1`。


对长事务的处理，是将长事务分解为多个小事务来分批处理。在应用程序中，最好是将事务的START TRANSACTION、COMMIT、ROLLBACK操作交给程序端来控制，而不是在存储过程中。


### 备份与恢复
Hot Backup 热备： 在数据库运行中直接备份，对运行中的数据库操作没有任何的影响；
Cold Backup 冷备： 在数据库停机状态下进行备份，一般备份数据库的物理文件；
Warm Backup 温备：也是在数据库运行中进行备份，但是会对当前的数据库操作有所影响，例如它会加一个全局读锁以保证备份数据的一致性。

对于mysqldump备份工具来说，可以通过添加`--single-transaction`选项获得InnoDB存储引擎的一致性备份，此时的备份是在一个执行时间很长的事务中完成的。请务必加上此选项。

mysqldump不能导出视图。免费好用的开源热备份工具有XtraBackup。



### 连接到mysql
``` sql
$ mysql -h192.168.1.100 -uscoot -p
$ mysql -h192.168.1.100 -uscoot -ptiger
```


### 不登录MYSQL来执行查询
可以在操作系统命令行下通过`-e`参数实现，例如查询test库下的表t_user：
``` mysql
$ mysql -h192.168.1.100 -uscoot -ptiger test -e "SELECT * FROM t_user"

+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | zhang |   21 |
|  2 | zhang |   23 |
|  4 | zhang |   21 |
|  5 | zhang |   21 |
+----+-------+------+
```

说明：

    -e, --execute=name  Execute command and quit. (Disables --force and history file.)


### 约束
概述：
1. 通过约束可以更好的保证数据表里数据的完整性；
2. 约束是在表上强制执行的数据校验规则，约束主要保证数据的完整性；
3. 当表中的数据存在相互依赖时，可以通过约束保护相关的数据不被删除。

MYSQL中支持以下五类约束：
1. NOT NULL：非空约束，指定某列不能为空；
2. UNIQUE：唯一约束，指定某列或者几列组合不能重复，允许空；
3. PRIMARY KEY：主键，指定该列的值可以唯一的表示每条记录；
4. FOREIGN KEY：外键，指定该行记录从属于主表中的一条记录，主要用于保证参照完整性；
5. CHECK：检查，指定一个布尔表达式，用于指定对应列的值必须满足该表达式。


### NOT NULL约束
如果我们向NOT NULL的字段插入一下NULL值，MySQL数据库会将其更改为0再进行插入。我们可以通过设置SQL_MODE来严格审核输入参数。
所有数据类型值都可以为null，如：int，float等。空字符串不等于null，0也不等于null。


### UNIQUE约束
UNIQUE约束用于保证指定列或指定列的组合不允许出现重复，但可以允许出现多个null值。同一张表内可以建多个UNIQUE约束。
``` sql
CREATE TABLE t_user (
  id INT NOT NULL,
  userName VARCHAR(20) UNIQUE
);

CREATE TABLE t_user (
  id INT NOT NULL,
  userName VARCHAR(20),
  CONSTRAINT index_uk UNIQUE(id, userName)
);
```


### PRIMARY KEY约束
主键约束相当于唯一约束和非空约束，每个表中最多允许一个主键。
``` sql
CREATE TABLE t_user (
  id INT PRIMARY KEY,
  userName VARCHAR(20)
);

CREATE TABLE t_user (
  id INT NOT NULL,
  userName VARCHAR(20),
  CONSTRAINT index_pk PRIMARY KEY(id)
);
```


### FOREIGN KEY约束
概述：
1. 外键约束主要用于保证一个或两个数据表之间的参照完整性，外键构建于一个表的两个字段或者两个表的两个字段之间；
2. 建立外键时mysql也会为该列建立索引；
3. 子表外键列的值必须在主表被参照列值的范围之内，或者为空；
4. 为了保证子表参照的主表存在，通常应先建主表；
5. 使用列级约束语法建立外键约束直接使用`REFERENCES`关键字。指定该列参照的哪个主表，以及参照主表的哪一列。

``` sql
CREATE TABLE teacher (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(20)
);

CREATE TABLE student (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(20),
  teacher_id INT,
  FOREIGN KEY(teacher_id) REFERENCES teacher(id)
);
```


### CHECK约束
CHECK约束在建表的列定义后增加逻辑表达式即可。
``` sql
CREATE TABLE t_user (
  id INT AUTO_INCREMENT PRIMARY KEY,
  userName VARCHAR(20),
  age INT,
  CHECK(age > 0)
);
```

目前所有MYSQL存储引擎并没有实现CHECK约束，也就是说，上面的建表语句正确，但约束无效。
https://dev.mysql.com/doc/refman/5.7/en/create-table.html#create-table-indexes-keys

    The CHECK clause is parsed but ignored by all storage engines.


### COUNT(1)、COUNT(*)、COUNT(pk)的区别
下面两句产生的结果是一样的:
COUNT(*) 统计表中有多少行，包括字段为NULL的行
COUNT(1) 统计表中有多少行，包括字段为NULL的行

当pk为主键的时候，因为主键不允许为NULL，所以
COUNT(pk) 同样是，统计表中有多少行，包括字段为NULL的行

但是，如果pk代表的字段允许为NULL，那么将产生不同的结果
COUNT(pk) 统计表中有多少行，不包括pk为NULL的行

``` sql
sql> SELECT * FROM t_user;
+----+------+--------+---------------------+
| id | name | salary | birthday            |
+----+------+--------+---------------------+
|  1 | a    |   4000 | 2018-01-25 09:30:05 |
|  2 | a    |   2000 | 2018-01-25 09:30:14 |
|  3 | a    |   3000 | 2018-01-25 09:30:10 |
|  4 | b    |   6000 | 2018-01-25 09:30:29 |
|  5 | b    |   5000 | 2018-01-25 09:30:24 |
|  6 | NULL |   6000 | 2018-01-25 09:30:29 |
+----+------+--------+---------------------+
6 rows in set (0.00 sec)

sql> SELECT COUNT(*), COUNT(1), COUNT(name) FROM t_user;
+----------+----------+-------------+
| COUNT(*) | COUNT(1) | COUNT(name) |
+----------+----------+-------------+
|        6 |        6 |           5 |
+----------+----------+-------------+
```

通常，建议使用COUNT(*)，因为它是SQL推荐的，SQL会自动优化到一个字段。

执行效率：
1. 当pk为主键时，COUNT(pk)比COUNT(1)快，否则COUNT(1)比COUNT(pk)快；
2. 当表有多个列时，如果没有主键，则COUNT(1)比COUNT(*)快； 如果有主键，则COUNT(pk)的效率是最优的；
3. 当表只有一个列时，COUNT(*)最优。


### 对MySQL数据库性能的测试工具
这里有2款比较好的工具：sysbench和mysql-tpcc

主要测试：
CPU性能
磁盘IO性能
调度程序性能
内存分配及传输速度
POSIX线程性能
数据库OLTP基准测试

