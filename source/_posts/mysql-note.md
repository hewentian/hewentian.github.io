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
SELECT CONCAT('Tim', ' ', 'Ho') FROM DUAL; -- Tim Ho
SELECT IFNULL(NULL, 'expr2') FROM DUAL;    -- expr2
SELECT IFNULL('expr1', 'expr2') FROM DUAL; -- expr1
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


### 创建数据库用户并授权
``` sql
CREATE USER 'bfg_user'@'%' IDENTIFIED BY 'gXk9IDpybrJPVMKq';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'bfg_user'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'bfg_user'@'%';
FLUSH PRIVILEGES;
```


### 修改用户密码
https://linuxize.com/post/how-to-change-mysql-user-password/

``` sql
mysql> ALTER USER 'user-name'@'localhost' IDENTIFIED BY 'NEW_USER_PASSWORD';
mysql> FLUSH PRIVILEGES;
```


### 创建数据库
在MYSQL中用`数据库管理员`用户创建了一个db，其他MYSQL用户是暂时看不到的，除非得到`数据库管理员`用户的授权
``` sql
CREATE DATABASE IF NOT EXISTS bfg_db COLLATE = 'utf8mb4_general_ci' CHARACTER SET = 'utf8mb4';

GRANT ALL ON bfg_db.* TO 'bfg_user'@'%' IDENTIFIED BY 'gXk9IDpybrJPVMKq';

GRANT ALL ON bfg_db.* TO 'bfg_user'@'localhost' IDENTIFIED BY 'gXk9IDpybrJPVMKq';

FLUSH PRIVILEGES;

mysql8.0之后要用上一节的方式创建用户和授权
```


### 查看用户的权限
``` sql
mysql> SHOW GRANTS;
+---------------------------------------------------------------------------+
| Grants for canal@%                                                        |
+---------------------------------------------------------------------------+
| GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%' |
+---------------------------------------------------------------------------+
1 row in set (0.00 sec)
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
    -- CALL proc_count_student(@cs)
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


### mysql中int型的取值范围
https://dev.mysql.com/doc/refman/8.0/en/integer-types.html

11.1.2 Integer Types (Exact Value) - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT
MySQL supports the SQL standard integer types INTEGER (or INT) and SMALLINT. As an extension to the standard, MySQL also supports the integer types TINYINT, MEDIUMINT, and BIGINT. The following table shows the required storage and range for each integer type.

Table 11.1 Required Storage and Range for Integer Types Supported by MySQL

+-----------+-----------------+----------------------+------------------------+----------------------+------------------------+
| Type      | Storage (Bytes) | Minimum Value Signed | Minimum Value Unsigned | Maximum Value Signed | Maximum Value Unsigned |
+-----------+-----------------+----------------------+------------------------+----------------------+------------------------+
| TINYINT   | 1	              | -128                 | 0                      | 127                  | 255                    |
| SMALLINT  | 2	              | -32768               | 0                      | 32767                | 65535                  |
| MEDIUMINT	| 3	              | -8388608             | 0                      | 8388607              | 16777215               |
| INT       | 4	              | -2147483648          | 0                      | 2147483647           | 4294967295             |
| BIGINT    | 8	              | -2^63                | 0                      | 2^63-1               | 2^64-1                 |
+-----------+-----------------+----------------------+------------------------+----------------------+------------------------+


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

-- 示例: sys_user表里的user_name字段，由原来长度是10个字符，修改改成40个字符
ALTER TABLE sys_user MODIFY COLUMN user_name VARCHAR(40);
```


### mysql 修改字段名
``` sql
ALTER TABLE [表名] CHANGE [旧字段名] [新字段名] [类型];

-- 示例: sys_user表里的user_name字段，修改改成user_name_new
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


### 日期类型函数
``` sql
mysql> SELECT NOW(), CURRENT_TIMESTAMP;
+---------------------+---------------------+
| NOW()               | CURRENT_TIMESTAMP   |
+---------------------+---------------------+
| 2022-07-11 14:18:13 | 2022-07-11 14:18:13 |
+---------------------+---------------------+

mysql> SELECT ADDDATE(NOW(), -3);
+---------------------+
| ADDDATE(NOW(), -3)  |
+---------------------+
| 2022-07-08 14:25:26 |
+---------------------+

mysql> SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %h:%i:%s'), DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s'), DATE_FORMAT(NOW(), '%Y-%m-%d %T');
+-----------------------------------------+-----------------------------------------+-----------------------------------+
| DATE_FORMAT(NOW(), '%Y-%m-%d %h:%i:%s') | DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s') | DATE_FORMAT(NOW(), '%Y-%m-%d %T') |
+-----------------------------------------+-----------------------------------------+-----------------------------------+
| 2022-07-11 02:20:09                     | 2022-07-11 14:20:09                     | 2022-07-11 14:20:09               |
+-----------------------------------------+-----------------------------------------+-----------------------------------+

mysql> SELECT STR_TO_DATE('2022-07-11', '%Y-%m-%d'), STR_TO_DATE('2022-07-11 00:02:10', '%Y-%m-%d %T');
+---------------------------------------+---------------------------------------------------+
| STR_TO_DATE('2022-07-11', '%Y-%m-%d') | STR_TO_DATE('2022-07-11 00:02:10', '%Y-%m-%d %T') |
+---------------------------------------+---------------------------------------------------+
| 2022-07-11                            | 2022-07-11 00:02:10                               |
+---------------------------------------+---------------------------------------------------+

mysql> SELECT TIMESTAMP(DATE_FORMAT(NOW(), '%Y-%m-%d')), TIMESTAMPADD(SECOND, 86399, DATE_FORMAT(NOW(),'%Y-%m-%d'));
+-------------------------------------------+------------------------------------------------------------+
| TIMESTAMP(DATE_FORMAT(NOW(), '%Y-%m-%d')) | TIMESTAMPADD(SECOND, 86399, DATE_FORMAT(NOW(),'%Y-%m-%d')) |
+-------------------------------------------+------------------------------------------------------------+
| 2022-07-11 00:00:00                       | 2022-07-11 23:59:59                                        |
+-------------------------------------------+------------------------------------------------------------+

mysql> SELECT STR_TO_DATE(DATE_FORMAT(NOW(), '%Y-%m-%d'), '%Y-%m-%d %T'), DATE_ADD(DATE_FORMAT(NOW(), '%Y-%m-%d'), INTERVAL 86399 SECOND);
+------------------------------------------------------------+-----------------------------------------------------------------+
| STR_TO_DATE(DATE_FORMAT(NOW(), '%Y-%m-%d'), '%Y-%m-%d %T') | DATE_ADD(DATE_FORMAT(NOW(), '%Y-%m-%d'), INTERVAL 86399 SECOND) |
+------------------------------------------------------------+-----------------------------------------------------------------+
| 2022-07-11 00:00:00                                        | 2022-07-11 23:59:59                                             |
+------------------------------------------------------------+-----------------------------------------------------------------+

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

如果执行过程中报`mysqldump: Couldn't execute 'SELECT COLUMN_NAME, JSON_EXTRACT(HISTOGRAM, '$."number-of-buckets-specified"')`，则加上这个参数即可解决：`--column-statistics=0`

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
mysql有一个表，有1000W条记录。在9:00的时候A用户对表进行查询，查询需10分钟才能完成，表没索引，FULL SCAN，表中某条记录的值为100。在9:05分的时候B对表进行了UPDATE操作，将记录的值修改为200。那么在9:10的时候，A获致到的是100还是200？结果是100。因为这是读一致性的问题，在A查的时候，它看到的是整个表在那一刻的快照的数据，返回的是那一刻的结果。不过，它有可能会抛出 snapshot too old 这个异常。


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


### 插入数据的时候，如果存在，则更新
``` sql
mysql> INSERT INTO t_user(name, age) VALUES('scott', 20), ('tiger', 30);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t_user;
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | scott |   20 |
|  2 | tiger |   30 |
+----+-------+------+
2 rows in set (0.01 sec)

mysql> INSERT INTO t_user(id, name, age) VALUE(1, 'ken', 40) ON DUPLICATE KEY UPDATE name = 'ken', age = 40;
Query OK, 2 rows affected (0.01 sec)

mysql> SELECT * FROM t_user;
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | ken   |   40 |
|  2 | tiger |   30 |
+----+-------+------+
2 rows in set (0.01 sec)
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


### 备份与恢复
Hot Backup 热备： 在数据库运行中直接备份，对运行中的数据库操作没有任何的影响；
Cold Backup 冷备： 在数据库停机状态下进行备份，一般备份数据库的物理文件；
Warm Backup 温备：也是在数据库运行中进行备份，但是会对当前的数据库操作有所影响，例如它会加一个全局读锁以保证备份数据的一致性。

对于mysqldump备份工具来说，可以通过添加`--single-transaction`选项获得InnoDB存储引擎的一致性备份，此时的备份是在一个执行时间很长的事务中完成的。请务必加上此选项。

mysqldump不能导出视图。免费好用的开源热备份工具有XtraBackup。


### 命令行连接mysql
        mysql -hip_address -uuser_name -Pport -ppassword db_name

如果`-p`后面的密码中有特殊字符，就要加个`\`来转义
        mysql -h192.168.1.100 -uroot -P3306 -pT3O\$8yKl%aRi user_db


示例：
``` sql
$ mysql -h192.168.1.100 -uscoot -p
$ mysql -h192.168.1.100 -uscoot -ptiger
```

如果连接之后，查询表数据发现是乱码，可以在连接的时候，指定编码：
        mysql -h192.168.1.100 -uscoot -ptiger --default-character-set=utf8


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
3. PRIMARY KEY：主键，指定该列的值可以唯一的表示每条记录，代表该字段的值不可重复且不能为null；
4. FOREIGN KEY：外键，指定该行记录从属于另一张表中的一条记录，主要用于保证参照完整性；
5. CHECK：检查，指定一个布尔表达式，用于指定对应列的值必须满足该表达式；
6. DEFAULT：默认约束，若该字段的值不手动插入时会有默认的值；
7. AUTO_INCREMENT：自增约束，描述的列必须是一个键列且是整型，一张表最多只有一个自增长列，一般是主键。


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


### left join, right join, inner join, join区别
有如下表：
``` sql
CREATE TABLE t_user (
  id INT NOT NULL,
  name VARCHAR(20)
);

CREATE TABLE t_contact (
  id INT NOT NULL COMMENT 't_user的主键 id',
  card VARCHAR(20)
);

INSERT INTO t_user VALUES (1, 'Scott'),(2, 'Tiger'),(3, 'Larry');
INSERT INTO t_contact VALUES (1, 'A'),(1, 'B'),(2, 'C'),(4, 'D');


MySQL [test]> select * from t_user;
+----+-------+
| id | name  |
+----+-------+
|  1 | Scott |
|  2 | Tiger |
|  3 | Larry |
+----+-------+
3 rows in set (0.00 sec)

MySQL [test]> select * from t_contact;
+----+------+
| id | card |
+----+------+
|  1 | A    |
|  1 | B    |
|  2 | C    |
|  4 | D    |
+----+------+
```

join 查询
``` sql
MySQL [test]> select * from t_user tu join t_contact tc on tu.id=tc.id;
+----+-------+----+------+
| id | name  | id | card |
+----+-------+----+------+
|  1 | Scott |  1 | A    |
|  1 | Scott |  1 | B    |
|  2 | Tiger |  2 | C    |
+----+-------+----+------+
```

inner join 查询
只返回两个表中联结字段相等的行
``` sql
MySQL [test]> select * from t_user tu inner join t_contact tc on tu.id=tc.id;
+----+-------+----+------+
| id | name  | id | card |
+----+-------+----+------+
|  1 | Scott |  1 | A    |
|  1 | Scott |  1 | B    |
|  2 | Tiger |  2 | C    |
+----+-------+----+------+
```

left join 查询
返回左表中的所有记录和右表中联结字段相等的记录
``` sql
MySQL [test]> select * from t_user tu left join t_contact tc on tu.id=tc.id;
+----+-------+------+------+
| id | name  | id   | card |
+----+-------+------+------+
|  1 | Scott |    1 | A    |
|  1 | Scott |    1 | B    |
|  2 | Tiger |    2 | C    |
|  3 | Larry | NULL | NULL |
+----+-------+------+------+
```

right join 查询
返回右表中的所有记录和左表中联结字段相等的记录
``` sql
MySQL [test]> select * from t_user tu right join t_contact tc on tu.id=tc.id;
+------+-------+----+------+
| id   | name  | id | card |
+------+-------+----+------+
|    1 | Scott |  1 | A    |
|    1 | Scott |  1 | B    |
|    2 | Tiger |  2 | C    |
| NULL | NULL  |  4 | D    |
+------+-------+----+------+
```


### Case Statement
Here is the syntax for MySQL Case statement.

        select
        case
            when condition1 then value1
            when condition2 then value2
            ...
        end,
        column2, column3, ...
        from table_name

示例
``` sql
mysql> select * from t_user;
+----+------+-----+
| id | name | age |
+----+------+-----+
|  1 | AAA  |  10 |
|  2 | BBB  |  20 |
|  3 | CCC  |  30 |
+----+------+-----+
3 rows in set (0.01 sec)

mysql> select id, name, case when age = 10 then '小孩' when age > 10 and age < 20 then '少年' when age >= 20 then '青年' end as age from t_user;
+----+------+--------+
| id | name | age    |
+----+------+--------+
|  1 | AAA  | 小孩   |
|  2 | BBB  | 青年   |
|  3 | CCC  | 青年   |
+----+------+--------+
3 rows in set (0.00 sec)
```


### 查询满足多个属性的产品列表
像淘宝中搜索满足多个条件的产品列表，准备数据如下：
``` sql
CREATE TABLE t_product (
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(20) NOT NULL COMMENT '产品名'
);

CREATE TABLE t_product_property (
id INT AUTO_INCREMENT PRIMARY KEY,
product_id INT NOT NULL COMMENT '产品id',
property_name VARCHAR(20) NOT NULL COMMENT '属性名'
);


INSERT INTO t_product(id, name) VALUES (1,'car'),(2,'plane');
INSERT INTO t_product_property(id, product_id, property_name)
  VALUES(1,1,'drive'),(2,1,'black'),(3,1,'benz'),(4,2,'fly'),(5,2,'white'),(6,2,'boeing');
```

匹配一个属性：drive
``` sql
SELECT * FROM t_product WHERE id IN
(
  SELECT p.product_id FROM (
    t_product_property p
  ) WHERE p.id = 1
);
```

匹配两个属性：drive, black
``` sql
SELECT * FROM t_product WHERE id IN
(
  SELECT p.product_id FROM (
    t_product_property p,
    (SELECT id, product_id FROM t_product_property) p2
  ) WHERE p.id = 1 AND p2.id = 2 AND p.product_id = p2.product_id
);
```

匹配三个属性:drive, black, benz
``` sql
SELECT * FROM t_product WHERE id IN
(
  SELECT p.product_id FROM (
    t_product_property p,
    (SELECT id, product_id FROM t_product_property) p2,
    (SELECT id, product_id FROM t_product_property) p3
  ) WHERE p.id = 1 AND p2.id = 2 AND p3.id = 3 AND p.product_id = p2.product_id AND p.product_id = p3.product_id
);
```


### mysql报错ERROR 1064 (42000)
原因是使用了mysql的保留字。如果表的字段使用了mysql的保留字，在查询的时候要用反引号将其引起来。

        SELECT * FROM t_user WHERE `key` = 'abc';


### mysql update
单表更新语法：
``` sql
UPDATE [LOW_PRIORITY] [IGNORE] table_name
SET
    column_name1 = expr1,
    column_name2 = expr2,
    ...
[WHERE
    condition];
```

也可以用从其他表SELECT出来的数据来SET值
``` sql
UPDATE table_name
SET
    column_name1 = (SELECT column_name1 FROM table_name2 WHERE column_name2 = 'h' ORDER BY RAND() LIMIT 1)
WHERE
    column_name1 IS NULL;
```

连表更新语法：
``` sql
UPDATE T1, T2
[INNER JOIN | LEFT JOIN] T1 ON T1.C1 = T2.C1
SET
    T1.C2 = T2.C2,
    T2.C3 = expr
WHERE condition
```

示例：
Suppose you want to adjust the salary of employees based on their performance.
``` sql
UPDATE employees e
INNER JOIN merits m ON e.performance = m.performance
SET
    e.salary = e.salary + e.salary * m.percentage;
```

The UPDATE LEFT JOIN statement basically updates a row in a table when it does not have a corresponding row in another table.

For example, you can increase the salary for a new hire by 1.5%  using the following statement:
``` sql
UPDATE employees e
LEFT JOIN merits m ON e.performance = m.performance
SET
    e.salary = e.salary + e.salary * 0.015
WHERE
    m.percentage IS NULL;
```

另一个例子：
``` sql
UPDATE t_user_info i
INNER JOIN t_user u ON i.user_id = u.id AND (LOWER(u.nick_name) = i.name OR UPPER(u.nick_name) = i.name)
SET i.remark = 'can update'
WHERE i.is_deleted = 0 AND u.is_deleted = 0;
```


### FOREIGN KEY引用导致无法删表
这时候，只要按如下方式删除，即可。
``` sql
SET FOREIGN_KEY_CHECKS=0;
DROP TABLE IF EXISTS table_name;
```


### 递归查询
https://stackoverflow.com/questions/20215744/how-to-create-a-mysql-hierarchical-recursive-query

``` sql
CREATE TABLE t_category (
id int primary key,
name varchar(100),
parent_id int
);

INSERT INTO t_category(id, name, parent_id) VALUES (1,'a',0), (2,'b',0), (3,'aa',1), (4,'ab',1), (5,'bb',2), (6,'aaa',3);


with recursive cte (id, name, parent_id) as (
  select     id,
             name,
             parent_id
  from       t_category
  where      id = 1
  union all
  select     c.id,
             c.name,
             c.parent_id
  from       t_category c
  inner join cte
          on c.parent_id = cte.id
)
select * from cte;
```


### jdbc连接参数说明
有jdbc连接如下：

        jdbc:mysql://mysql.hewentian.com:3306/seata?useUnicode=true&rewriteBatchedStatements=true


rewriteBatchedStatements：根据MySQL官网的说明，连接参数中的`rewriteBatchedStatements`为true时，在执行executeBatch，并且操作类型为insert时，jdbc驱动会把对应的SQL优化成`insert into () values (), ()`的形式来提升批量插入的性能。根据实际的测试，该参数设置为true后，对应的批量插入性能为原来的10倍多，因此在数据源为MySQL时，建议把该参数设置为true。


### MySQL JSON columns
MySQL 5.7+ InnoDB databases and PostgreSQL 9.2+ both directly support JSON document types in a single field.

Note that any database will accept JSON documents as a single string blob. However, MySQL and PostgreSQL support validated JSON data in real key/value pairs rather than a basic string.

JSON value fields can’t be indexed, so avoid using it on columns which are updated or searched regularly.

Note that JSON columns can’t be used as a primary key, be used as a foreign key, or have an index.

``` sql
CREATE TABLE `test` (
  `id` int  PRIMARY KEY AUTO_INCREMENT,
  `name` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `age` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

为表`test`增加一个JSON的列：

        ALTER TABLE `test` ADD COLUMN `address` JSON NOT NULL DEFAULT ( JSON_OBJECT() );  -- 带默认值
        ALTER TABLE `test` ADD COLUMN `address` JSON DEFAULT NULL;                        -- 默认为NULL


在建表的时候，设置JSON列
``` sql
CREATE TABLE `test` (
  `id` int  PRIMARY KEY AUTO_INCREMENT,
  `name` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `age` int DEFAULT NULL,
  `address` json NOT NULL DEFAULT (json_object())
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

一些操作
``` sql
mysql> SELECT * FROM test;
Empty set (0.02 sec)

mysql> INSERT INTO test (name, age) VALUES ('tim', 21);
Query OK, 1 row affected (0.02 sec)

mysql> INSERT INTO test (name, age, address) VALUES ('ho', 21, '{"province": "广东", "city": "广州"}');
Query OK, 1 row affected (0.02 sec)

mysql> INSERT INTO test (name, age, address) VALUES ('ho', 21, '["广东", "广州", "番禺"]');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM test;
+----+------+------+------------------------------------------+
| id | name | age  | address                                  |
+----+------+------+------------------------------------------+
|  1 | tim  |   21 | {}                                       |
|  2 | ho   |   21 | {"city": "广州", "province": "广东"}     |
|  3 | ho   |   21 | ["广东", "广州", "番禺"]                 |
+----+------+------+------------------------------------------+
3 rows in set (0.01 sec)
```

下面的函数，可以创建JSON
``` sql
mysql> SELECT JSON_ARRAY(1, 2, 'abc');
+-------------------------+
| JSON_ARRAY(1, 2, 'abc') |
+-------------------------+
| [1, 2, "abc"]           |
+-------------------------+


mysql> SELECT JSON_OBJECT('a', 1, 'b', 2);
+-----------------------------+
| JSON_OBJECT('a', 1, 'b', 2) |
+-----------------------------+
| {"a": 1, "b": 2}            |
+-----------------------------+


mysql> SELECT JSON_TYPE('[1, 2, "abc"]');
+----------------------------+
| JSON_TYPE('[1, 2, "abc"]') |
+----------------------------+
| ARRAY                      |
+----------------------------+


mysql> SELECT JSON_TYPE('{"a": 1, "b": 2}');
+-------------------------------+
| JSON_TYPE('{"a": 1, "b": 2}') |
+-------------------------------+
| OBJECT                        |
+-------------------------------+


mysql> SELECT JSON_TYPE('{"a": 1, "b": 2');
ERROR 3141 (22032): Invalid JSON text in argument 1 to function json_type: "Missing a comma or '}' after an object member." at position 15.
mysql>
```

函数`JSON_VALID()`可以验证一段字符串，是否为正确的JSON。返回1表示正确，返回0表示错误。
``` sql
mysql> SELECT JSON_VALID('[1, 2, "abc"]');
+-----------------------------+
| JSON_VALID('[1, 2, "abc"]') |
+-----------------------------+
|                           1 |
+-----------------------------+

mysql> SELECT JSON_VALID('{"a": 1, "b": 2}');
+--------------------------------+
| JSON_VALID('{"a": 1, "b": 2}') |
+--------------------------------+
|                              1 |
+--------------------------------+

mysql> SELECT JSON_VALID('{"a": 1, "b": 2');
+-------------------------------+
| JSON_VALID('{"a": 1, "b": 2') |
+-------------------------------+
|                             0 |
+-------------------------------+
```


一些搜索操作
``` sql
mysql> SELECT * FROM test WHERE JSON_CONTAINS(address, '["广州"]');
+----+------+------+--------------------------------+
| id | name | age  | address                        |
+----+------+------+--------------------------------+
|  3 | ho   |   21 | ["广东", "广州", "番禺"]       |
+----+------+------+--------------------------------+
1 row in set (0.03 sec)


mysql> SELECT * FROM test WHERE JSON_CONTAINS(address, '{"city": "广州"}');
+----+------+------+------------------------------------------+
| id | name | age  | address                                  |
+----+------+------+------------------------------------------+
|  2 | ho   |   21 | {"city": "广州", "province": "广东"}     |
+----+------+------+------------------------------------------+
1 row in set (0.01 sec)


mysql> SELECT * FROM test WHERE JSON_SEARCH(address, 'one', '广%') IS NOT NULL;
+----+------+------+------------------------------------------+
| id | name | age  | address                                  |
+----+------+------+------------------------------------------+
|  2 | ho   |   21 | {"city": "广州", "province": "广东"}     |
|  3 | ho   |   21 | ["广东", "广州", "番禺"]                 |
+----+------+------+------------------------------------------+
2 rows in set (0.01 sec)

mysql> SELECT * FROM test WHERE JSON_SEARCH(address, 'all', '广%') IS NOT NULL;
+----+------+------+------------------------------------------+
| id | name | age  | address                                  |
+----+------+------+------------------------------------------+
|  2 | ho   |   21 | {"city": "广州", "province": "广东"}     |
|  3 | ho   |   21 | ["广东", "广州", "番禺"]                 |
+----+------+------+------------------------------------------+
2 rows in set (0.01 sec)


mysql> SELECT JSON_EXTRACT('{"city": "广州", "province": "广东"}', '$.province');
+------------------------------------------------------------------------+
| JSON_EXTRACT('{"city": "广州", "province": "广东"}', '$.province')     |
+------------------------------------------------------------------------+
| "广东"                                                                 |
+------------------------------------------------------------------------+
1 row in set (0.01 sec)


{
  "a": 1,
  "b": 2,
  "c": [3, 4],
  "d": {
    "e": 5,
    "f": 6
  }
}

Example paths:
$.a returns 1
$.c returns [3, 4]
$.c[1] returns 4
$.d.e returns 5
$**.e returns [5]


mysql> SELECT id, name, age, address->"$[0]" FROM test ;
+----+------+------+------------------------------------------+
| id | name | age  | address->"$[0]"                          |
+----+------+------+------------------------------------------+
|  1 | tim  |   21 | {}                                       |
|  2 | ho   |   21 | {"city": "广州", "province": "广东"}     |
|  3 | ho   |   21 | "广东"                                   |    -- 注意这条数据
+----+------+------+------------------------------------------+
3 rows in set (0.01 sec)

mysql> SELECT id, name, age, address->"$.city" FROM test ;
+----+------+------+-------------------+
| id | name | age  | address->"$.city" |
+----+------+------+-------------------+
|  1 | tim  |   21 | NULL              |
|  2 | ho   |   21 | "广州"            |                           -- 注意这条数据
|  3 | ho   |   21 | NULL              |
+----+------+------+-------------------+
3 rows in set (0.00 sec)


mysql> SELECT * FROM test WHERE address->"$.city" IS NOT NULL;
+----+------+------+------------------------------------------+
| id | name | age  | address                                  |
+----+------+------+------------------------------------------+
|  2 | ho   |   21 | {"city": "广州", "province": "广东"}     |
+----+------+------+------------------------------------------+
1 row in set (0.01 sec)

```


### LIKE 查询
LIKE查询默认大小写不敏感，但是，我们可以强制它大小写敏感。
``` sql
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'ID',
  `name` varchar(100) NOT NULL COMMENT 'name'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO t_user(id, name) VALUE(1,'Tim');
INSERT INTO t_user(id, name) VALUE(2,'tim');
INSERT INTO t_user(id, name) VALUE(3,'Ho');

mysql> SELECT * FROM t_user WHERE name LIKE '%t%';
+----+------+
| id | name |
+----+------+
|  1 | Tim  |
|  2 | tim  |
+----+------+
2 rows in set (0.00 sec)

mysql> SELECT * FROM t_user WHERE name LIKE '%T%';
+----+------+
| id | name |
+----+------+
|  1 | Tim  |
|  2 | tim  |
+----+------+
2 rows in set (0.00 sec)

mysql> SELECT * FROM t_user WHERE name LIKE BINARY '%t%';
+----+------+
| id | name |
+----+------+
|  2 | tim  |
+----+------+
1 row in set (0.00 sec)

```


### mysql 时间 datetime 索引不生效问题
1、在查询数据条数约占总条数五分之一以下时能够使用到索引，但超过五分之一时，则使用全表扫描了；
2、查询条件有日期索引和其他条件的话，只有所有条件都有索引的情况下，才会走日期索引。


### 对MySQL数据库性能的测试工具
这里有2款比较好的工具：sysbench和mysql-tpcc

主要测试：
CPU性能
磁盘IO性能
调度程序性能
内存分配及传输速度
POSIX线程性能
数据库OLTP基准测试

