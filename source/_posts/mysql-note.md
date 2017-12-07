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

