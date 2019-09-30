---
title: maven 学习笔记
date: 2017-11-10 00:48:44
tags: maven
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


### 解决SecurityException问题
在使用maven打JAR包，在运行的时候可能会抛如下异常：

    Exception in thread “main” java.lang.SecurityException: Invalid signature file digest for Manifest main attributes

原因：由于重复引用某些依赖，导致在maven打包之后生成了一些.SF等文件，运行jar时会抛出。在META-INF下会有多余的以SF、RSA结尾的文件，删除后不会出现次问题

如无法重新打包，则解决方法如下：

    zip -d yourjar.jar 'META-INF/.SF' 'META-INF/.RSA' 'META-INF/*SF'

最好的方法是在pom.xml中配置打包的时候忽略这些文件：
``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>1.4</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
            </configuration>
        </execution>
    </executions>
</plugin>
```


### maven跳过单元测试的方式
1. `-DskipTests`： 编译测试用例类生成相应的class文件至target/test-classes下，但不执行测试用例，当然，也可以直接在POM文件中写上：
``` bash
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M3</version>
    <configuration>
        <skipTests>true</skipTests>
    </configuration>
</plugin>
```

2. `-Dmaven.test.skip=true`： 不编译测试用例类，也不执行测试用例；


打包的时候跳过单元测试：

    mvn package -Dmaven.test.skip=true


### maven打包并指定main方法
pom.xml配置如下：
``` xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>1.4</version>
            <configuration>
                <createDependencyReducedPom>true</createDependencyReducedPom>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <!--指定main方法-->
                                <mainClass>com.hewentian.jdk.App</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```


