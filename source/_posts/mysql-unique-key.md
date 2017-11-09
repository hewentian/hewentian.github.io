---
title: mysql 添加唯一约束
date: 2017-11-07 10:36:58
categories: db
---
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
