---
title: maven 学习笔记
date: 2017-11-10 00:48:44
tags: maven
categories: other
---

### 在ubuntu上面安装 maven
``` bash
$ su root
$ cd /home/hewentian/Downloads
$ tar xzvf apache-maven-3.3.9-bin.tar.gz
$ cd /usr/local
$ mv /home/hewentian/Downloads/apache-maven-3.3.9 ./

$ vi /etc/profile
# 在打开的文件中添加如下代码
# add maven
export M2_HOME=/usr/local/apache-maven-3.3.9
export PATH=$M2_HOME/bin:$PATH

$ source /etc/profile
$ mvn -version

Maven home: /usr/local/apache-maven-3.3.9
Java version: 1.8.0_102, vendor: Oracle Corporation
Java home: /usr/local/java/jdk1.8.0_102/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.10.0-28-generic", arch: "amd64", family: "unix"
```


maven的学习有个好网站：https://howtodoinjava.com/maven/

### Maven Dependency Scopes
Maven dependency scope attribute is used to specify the visibility of a dependency, relative to the different lifecycle phases (build, test, runtime etc). Maven provides six scopes i.e. compile, provided, runtime, test, system, and import.
1. Compile Scope
2. Provided Scope
3. Runtime Scope
4. Test Scope
5. System Scope
6. Import Scope

详情：https://howtodoinjava.com/maven/maven-dependency-scopes/


### External Dependency
Some times, you will have to refer jar files which are not in maven repository (neither local, central or remote repository). You can use these jars by placing them in project’s lib folder and configure the external dependency like this:
``` xml
<dependency>
    <groupId>extDependency</groupId>
    <artifactId>extDependency</artifactId>
    <scope>system</scope>
    <version>1.0</version>
    <systemPath>${basedir}\war\WEB-INF\lib\extDependency.jar</systemPath>
</dependency>
```
* The groupId and artifactId are both set to the name of the dependency.
* The scope element value is set to system.
* The systemPath element refer to the location of the JAR file.


### Maven Dependency Tree
Using maven’s dependency:tree command, you can view list of all dependencies into your project – transitively. Transitive dependency means that if A depends on B and B depends on C, then A depends on both B and C.

Transitivity brings a very serious problem when different versions of the same artifacts are included by different dependencies. It may cause version mismatch issue in runtime. In this case, dependency:tree command is be very useful in dealing with conflicts of JARs.

        $ mvn dependency:tree


### Maven Dependency Exclusion
Apart from version mismatch issue caused with transitive dependency, there can be version mismatch between project artifacts and artifacts from the platform of deployment, such as Tomcat or another server.

To resolve such version mismatch issues, maven provides <exclusion> tag, in order to break the transitive dependency.

For example, when you have JUnit 4.12 in classpath and including DBUnit dependency, then you will need to remove JUnit 3.8.2 dependency. It can be done with exclusion tag.
``` xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.dbunit</groupId>
    <artifactId>dbunit</artifactId>
    <version>${dbunit.version}</version>
    <scope>test</scope>
    <exclusions>
        <!--Exclude transitive dependency to JUnit-3.8.2 -->
        <exclusion>
            <artifactId>junit</artifactId>
            <groupId>junit</groupId>
         </exclusion>
    </exclusions>
</dependency>
```


### maven常用命令
编译，编译后会生成target目录
        mvn compile

清理，将编译生成的target目录删掉
        mvn clean

打包，将编译生成的target目录下的class文件打成jar包
        mvn package

一般组合使用这些使用，如`mvn clean compile`、`mvn clean package`

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
    <version>3.2.2</version>
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
            <version>3.2.2</version>
            <configuration>
                <createDependencyReducedPom>false</createDependencyReducedPom>
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


### 创建maven项目
在IDEA中可以通过`File -> New -> Module`，打开`New Module`对话框，在左则选择Maven，并在右则勾选`Create from archetype`：
1. 创建jar项目，则选择`org.apache.maven.archetypes:maven-archetype-quickstart`；
2. 创建web项目，则选择`org.apache.maven.archetypes:maven-archetype-webapp`；


### maven-shade-plugin插件
此插件的主要功能：
1. 将依赖的jar包打包到当前jar包中，常规打包是不会将所依赖jar包打进来的；
2. 指定jar包的默认运行方法：https://maven.apache.org/plugins/maven-shade-plugin/examples/executable-jar.html；
3. 为jar包选择指定的内容：https://maven.apache.org/plugins/maven-shade-plugin/examples/includes-excludes.html

详细介绍详见：http://maven.apache.org/plugins/maven-shade-plugin/plugin-info.html
``` xml
<project>
  ...
  <build>
    <!-- To define the plugin version in your parent POM -->
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-shade-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        ...
      </plugins>
    </pluginManagement>
    <!-- To use the plugin goals in your POM or parent POM -->
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.2</version>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```


