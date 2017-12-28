---
title: mysql 笔记
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

flush privileges;
```
