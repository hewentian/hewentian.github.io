---
title: xxl-job 学习笔记
date: 2022-03-07 14:31:20
tags: job
categories: java
---

本篇将说说分布式任务调度平台[XXL-JOB][link_id_xxl_job]的安装使用。

### 下载
``` bash
$ cd /home/hewentian/ProjectD/gitHub/
$ git clone https://github.com/xuxueli/xxl-job.git
$ cd xxl-job/
$ git checkout -b 2.3.0 origin/2.3.0
$ ls
doc  LICENSE  NOTICE  pom.xml  README.md  xxl-job-admin  xxl-job-core  xxl-job-executor-samples
```

下载后，将代码导入IDEA，这样方便修改文件。


### 初始化数据库
数据库文件在`/home/hewentian/ProjectD/gitHub/xxl-job/doc/db/tables_xxl_job.sql`
``` sql
mysql> source /home/hewentian/ProjectD/gitHub/xxl-job/doc/db/tables_xxl_job.sql

mysql> show tables;
+--------------------+
| Tables_in_xxl_job  |
+--------------------+
| xxl_job_group      |
| xxl_job_info       |
| xxl_job_lock       |
| xxl_job_log        |
| xxl_job_log_report |
| xxl_job_logglue    |
| xxl_job_registry   |
| xxl_job_user       |
+--------------------+
8 rows in set (0.00 sec)
```


### 部署调度中心
调度中心项目：xxl-job-admin

配置日志存放路径，`/home/hewentian/ProjectD/gitHub/xxl-job/xxl-job-admin/src/main/resources/logback.xml`，仅配置下面这一项即可：

        <property name="log.path" value="/home/hewentian/logs/xxl-job/xxl-job-admin.log"/>

配置项目基本配置文件，`/home/hewentian/ProjectD/gitHub/xxl-job/xxl-job-admin/src/main/resources/application.properties`，修改项如下：
``` bash
spring.datasource.url=jdbc:mysql://mysql.hewentian.com:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

```

启动项目：启动类为`com.xxl.job.admin.XxlJobAdminApplication`

WEB访问地址：
http://localhost:8080/xxl-job-admin

默认用户名/密码：admin/123456


![](/img/xxl-job-1.png "")


### 部署执行器项目
执行器项目，我部署的是：xxl-job-executor-samples/xxl-job-executor-sample-springboot

配置日志存放路径，`/home/hewentian/ProjectD/gitHub/xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/resources/logback.xml`，仅配置下面这一项即可：

        <property name="log.path" value="/home/hewentian/logs/xxl-job/xxl-job-executor-sample-springboot.log"/>

配置项目基本配置文件，`/home/hewentian/ProjectD/gitHub/xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/resources/application.properties`，修改项如下：
``` bash
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
xxl.job.executor.logpath=/home/hewentian/logs/xxl-job/jobhandler
```

启动项目：启动类为`com.xxl.job.executor.XxlJobExecutorApplication`


### 在调度中心新建任务
登录下面的地址，进行配置：
http://localhost:8080/xxl-job-admin

项目中的代码如下：
![](/img/xxl-job-2.png "")

执行器管理：
![](/img/xxl-job-3.png "")

任务管理：
![](/img/xxl-job-4.png "")

调度日志：
![](/img/xxl-job-5.png "")


[link_id_xxl_job]: https://www.xuxueli.com/xxl-job/

