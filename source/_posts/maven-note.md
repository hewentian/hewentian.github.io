---
title: maven笔记
date: 2017-11-10 00:48:44
categories: other
---
在`eclipse`中，遇到Missing artifact jdk.tools:jdk.tools:jar:1.8

原因：tools.jar包是JDK的，而tools.jar并未在仓库`maven repository`中

解决方案：在pom文件中添加如下代码即可
``` xml
<dependency>
	<groupId>jdk.tools</groupId>
	<artifactId>jdk.tools</artifactId>
	<version>1.8</version>
	<scope>system</scope>
	<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
</dependency>
```

在 pom.xml文件中不要写如下这样的配置：
``` xml
<properties>
	<java.home>/usr/local/java/jdk1.8.0_102</java.home>
</properties>
	
	<dependencies>
		<dependency>
			<groupId>jre</groupId>
			<artifactId>jre</artifactId>
			<version>8.0</version>
			<scope>system</scope>
			<systemPath>${java.home}/lib/rt.jar</systemPath>
		</dependency>
</dependencies>
```
去掉`<java.home>/usr/local/java/jdk1.8.0_102</java.home>`配置，这时`java.home`变量继承自`eclipse`的`java.home`配置。从pom.xml文件的Effective POM可以查看到java.home变量被替换了，eclipse的java.home路径在`help->about eclipse->Installation Details->configuration`页可以找到。

### ik-analyzer的使用
如果项目中要使用到`ik-analyzer`分词，可以到如下地址下载，并安装：
https://github.com/wks/ik-analyzer
