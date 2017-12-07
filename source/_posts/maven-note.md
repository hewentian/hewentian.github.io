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
