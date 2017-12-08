---
title: 安装 JDK
date: 2017-12-08 11:44:06
tags: jdk
categories: java
---
## 下面分别说说在`Windows`和`Linux`平台下面安装`JDK`

### 1.在`Windows`下面安装`JDK`
	JDK环境变量配置的步骤如下：
	1. 我的电脑 -> 属性 -> 高级 -> 环境变量；
	2. 配置用户变量：
		a.新建 JAVA_HOME
		C:\Program Files\Java\jdk1.8.0_101 （JDK的安装路径，务必替换成你自已的）
		b.新建 PATH（如果系统本身已有，则修改，加入下面的代码）
		%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
		c.新建 CLASSPATH
		.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
	3. 测试环境变量配置是否成功：
		开始 -> 运行 -> cmd
		键盘敲入: java
	出现相应的命令，而不是出错信息，即表示配置成功！
	
环境变量配置的理解：

1. PATH：作用是指定命令搜索路径，在命令行下面执行命令如：`javac`编译`java`程序时，它会到`PATH`变量所指定的路径中查找看是否能找到相应的命令程序。我们需要把`jdk`安装目录下的`bin`目录增加到现有的`PATH`变量中，`bin`目录中包含经常要用到的可执行文件如`javac/java/javadoc`等待，设置好`PATH`变量后，就可以在任何目录下执行`javac/java`等工具了；
2. CLASSPATH：作用是指定类搜索路径，要使用已经编写好的类，前提当然是能够找到它们了，`JVM`就是通过`CLASSPTH`来寻找类的。我们需要把`jdk`安装目录下的`lib子`目录中的`dt.jar`和`tools.jar`设置到`CLASSPATH`中，当然，当前目录“`.`”也必须加入到该变量中；
3. JAVA_HOME：它指向`jdk`的安装目录，`Eclipse/NetBeans/Tomcat`等软件就是通过搜索`JAVA_HOME`变量来找到并使用安装好的`jdk`。

### 2.在`Linux`下面安装`JDK`
第一步：下载`jdk-8u102-linux-x64.tar.gz`
直接在`ORACLE`的官网中下载就可以:
http://www.oracle.com/technetwork/java/javase/downloads/index.html
PS：要注意系统版本的选择，32位 还是 64位，执行如下命令即可知道答案
``` bash
$ uname -a
Linux hewentian-Lenovo-IdeaPad-Y470 4.10.0-28-generic #32~16.04.2-Ubuntu SMP Thu Jul 20 10:19:48 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

第二步：解压安装
接着就是解压`tar.gz`的文件了
``` bash
$ cd /home/hewentian/Downloads
$ tar -xzvf jdk-8u102-linux-x64.tar.gz
```
解压后会得到`jdk1.8.0_102`文件夹，接着就是将解压出来的文件夹移动到`/usr/local`的目录下
在这之前当然需要你拥有root的权限`su root`(对于ubuntu)再输入root账户的密码，做好这些准备之后，我们就可以把jdk的文件移动到我们想要的位置了。
``` bash
$ su root
Password: 
$ mv jdk1.8.0_102 /usr/local
```

第三步：修改环境变量，若`PATH`已存在，则用冒号作间隔，将jdk的bin目录地址加上，这样java的环境变量将配置成功了，配置完成如下：
``` bash
vi /etc/profile 

export JAVA_HOME=/usr/local/jdk1.8.0_102
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$JAVA_HOME:$PATH
```
记得保存 后执行
``` bash 
$ source /etc/profile
```

第四步：但这样默认使用的JDK可能还不是我们刚才安装的，因为ubuntu可能还会有默认的jdk，如openjdk；所以，为了使默认使用的是我们安装的jdk，还需执行如下命令：
``` bash
$ sudo update-alternatives --install /usr/bin/java java /usr/local/jdk1.8.0_102/bin/java 300  
$ sudo update-alternatives --install /usr/bin/javac javac /usr/local/jdk1.8.0_102/bin/javac 300    

$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
这时如果有多个jdk的话（比如openJDK和SUN JDK），就会出来一个列表，当前默认的会在列表前面有一个"`*`"号，
这时我们就要选择我们刚装的SUN JDK的java的那个序号，输入这个序号，回车就行了。
```
第五步：成功执行命令后，我们安装的JDK就是系统默认的了，执行如下命令，就可以成功看到JDK的相关信息了如：
``` bash
$ java -version　
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

第六步：在ubuntu下面，可能root用户无法使用JDK，这样的话，执行下面的配置：
``` bash
$ vi /root/.bashrc

在文件中加上(注意，下面的变量参数有${}的，与上面的不同)
export JAVA_HOME=/usr/local/jdk1.8.0_102
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:$PATH

最后：
source /root/.bashrc
```
