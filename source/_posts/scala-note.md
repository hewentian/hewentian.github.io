---
title: scala 学习笔记
date: 2019-12-30 10:33:00
tags: scala
categories: bigdata
---

### 在Linux下面安装SDK
Scala依赖于JAVA，所以，在安装Scala之前必须先安装JDK，安装过程可以参考：[安装 JDK][link_id_jdk-install]。

第一步：下载Scala，在`https://www.scala-lang.org/download/`可以找到下载链接，目前最新版本是`scala-2.11.12`，下载地址：
https://downloads.lightbend.com/scala/2.11.12/scala-2.11.12.tgz

第二步：解压安装
``` bash
$ cd /home/hewentian/Downloads
$ tar xf scala-2.11.12.tgz
```
解压后会得到`scala-2.11.12`文件夹，接着就是将解压出来的文件夹移动到`/usr/local`的目录下。在这之前当然需要你拥有root的权限`su root`(对于ubuntu)再输入root账户的密码，做好这些准备之后，我们就可以把sdk的文件移动到我们想要的位置了。
``` bash
$ su root
Password: 
$ mv scala-2.11.12 /usr/local/
```

第三步：修改环境变量，若`PATH`已存在，则用冒号作间隔，将sdk的bin目录地址加上，配置如下：
``` bash
vi /etc/profile 

export SCALA_HOME=/usr/local/scala-2.11.12
export PATH=$PATH:$SCALA_HOME/bin
```

记得保存，最后执行
``` bash 
$ source /etc/profile
```

第四步：验证安装是否成功，执行如下命令：
``` bash 
$ scalac -version
Scala compiler version 2.11.12 -- Copyright 2002-2017, LAMP/EPFL
$
$ scala -version
Scala code runner version 2.11.12 -- Copyright 2002-2017, LAMP/EPFL
```

看到上面的输出，证明安装成功。


### 在IDEA中安装Scala开发插件
启动IDEA，在启动界面中依次选择：`Configure->Settings->Plugins`，在右则搜索框中输入Scala进行搜索，找到如下图所示插件。

![](/img/scala-1.png "")

点击`Install`进行安装。安装成功后，需重启IDEA。重启IDEA后，在启动界面中点击`Create New Project`，如果能看到有Scala选项，则证明安装成功。

![](/img/scala-2.png "")

接着在IDEA中配置SDK，点击上图的`Cancel`，回到刚才的启动界面。依次选择：`Configure->Project Defaults->Project Structure`，在弹出的界面中选择`Global Libraries`，然后在右则点击`+`号，并在弹出的下拉列表中选择`Scala SDK`，配置如下：

![](/img/scala-3.png "")

在上图中点击`Apply`，至此IDEA中Scala开发环境配置完成。


### 在IDEA中已创建好的java项目中编写scala代码
1. 在src/main/下创建一个目录scala；
2. 将src/main/scala设置成sources目录：File -> Project Structure -> Modules -> 在右则选择我们的项目 -> 然后在右则选择Sources这个tab -> 选中我们刚才的目录src/main/scala -> 点击上面的 Mark as: Sources -> 最后点右下角的OK；
3. 有可能还要将刚才添加的SDK(Global Libraries)包删除，重新再添加。

[link_id_jdk-install]: ../../../../2017/12/08/jdk-install/

