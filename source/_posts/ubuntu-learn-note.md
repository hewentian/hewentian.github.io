---
title: ubuntu 学习笔记
date: 2017-09-12 20:41:37
tags: Linux
categories: Linux
---

Ubuntu安装ssh时出现软件包 openssh-server 还没有可供安装的候选者错误

错误如下：

sudo apt-get install opensshserver正在读取软件包列表...

完成正在分析软件包的依赖关系树正在读取状态信息...

完成现在没有可用的软件包 openssh-server，

但是他被其他的软件包引用了这可能意味着这个缺失的软件包可能已被废弃，或者只能在其他发布源中找到

E:软件包 openssh-server 还没有可供安装的候选者

解决方案：分析原因是我们的apt-get没有更新，当然如果你的是最新的系统不用更新也行，但是我相信很多人都是需要更新的吧，操作命令如下：
``` bash
$ sudo apt-get update
```
更新完毕后执行：
``` bash
$ sudo apt-get install openssh-server
```
最后我们用命令ps -e|grep ssh 来看下open-server安装成功没有，如果出现如下截图红色标出的部分，说明安装成功了。
``` bash
$ ps -e|grep ssh
```

在ubuntu中的vi编辑器中怎么使用
默认情况下ubuntu上也安装有vi但是奇怪的是这个vi是vim-common版本，基本上用不了所以要先把这个版本的vi卸载掉才可以，卸载命令是
``` bash
$ sudo apt-get remove vim-common
```
卸载成功之后接着执行 sudo apt-get install vim,安装好之后就能使用了


要检查特定的包，比如firefox是否安装了，使用这个命令：
``` bash
$ dpkg -s firefox

ubuntu下useradd与adduser区别，新建用户不再home目录下
useradd username不会在/home下建立一个文件夹username
adduser username会在/home下建立一个文件夹username
useradd -m username跟adduser一样，可以建立一个文件夹username
```
永久性删除用户账号
``` bash
$ userdel peter
```

还有的就是文件编码的问题，在windows下默认都是用ANSI格式编码，但是在ubuntu下面可能会是其他格式的编码，如果两个系统的编码不一致，就会产生乱码。
最好是都采用UTF-8格式编码


ubuntu root默认密码（初始密码）
ubuntu安装好后，root初始密码（默认密码）不知道，需要设置。

1、先用安装时候的用户登录进入系统

2、输入：
``` bash
sudo passwd
```
按回车

3、输入新密码，重复输入密码，最后提示passwd：password updated sucessfully

此时已完成root密码的设置

4、输入：
``` bash
$ su root
```
切换用户到root试试.......




如果执行./configure无法执行，
要安装：
``` bash
$ sudo apt-get install build-essential
```

在ubuntu软件源里zlib和zlib-devel叫做zlib1g zlib1g.dev
``` bash
$ sudo apt-get install zlib1g
$ sudo apt-get install zlib1g.dev
```

直接输入上述命令后还是不能安装。这就要求我们先装ruby.
在ubuntu里，zlib叫zlib1g，相应的zlib-devel叫zlib1g.dev。默认的安装源里没有zlib1g.dev。要在packages.ubuntu.com上找。
``` bash
$ sudo apt-get install ruby
```

然后再装zlib1g-dev就可以了
``` bash
$ sudo apt-get install zlib1g-dev
```

解决依赖包openssl安装，命令：
[cpp] view plain copy 在CODE上查看代码片派生到我的代码片
``` baah
$ sudo apt-get install openssl libssl-dev  
```

解决依赖包pcre安装，命令：
[cpp] view plain copy 在CODE上查看代码片派生到我的代码片
``` bash
$ sudo apt-get install libpcre3 libpcre3-dev  
```

解决依赖包zlib安装，命令：
[cpp] view plain copy 在CODE上查看代码片派生到我的代码片
``` bash
$ sudo apt-get install zlib1g-dev  
```

### 当你安装完ubuntu后，安装五笔输入法：
1.打开命令行输入：
``` bash
$ sudo apt-get install fcitx-table-wubi
```
安装完后，重启（也可以在下面的设置完毕之后，再重启）

2.`System Settings -> language support` 在 Keyboard input method system 中选中 fcitx
然后点击 Apply System-Wide, 然后重启

3.重启后，`System Settings -> Text Entry`, 中输入 Wubi(Fcitx)，这样，就完成了五笔输入法的安装。


### ubuntu 安装 Robo3t 出现 Available platform plugins are: xcb.
解决办法,打开终端输入一下命令
``` bash
$ mkdir ~/robo-backup
$ mv robo3t-1.1.1-linux-x86_64-c93c6b0/lib/libstdc++* ~/robo-backup/
```

输入命令之后会在robo-backup中出现两个文件 libstdc++.so.6 和 libstdc++.so.6.0.22，然后启动robo3t就ok了

其实，也可以將其删掉，只要你还有压缩包，即可。


### ubuntu 下 Eclipse 中 syso 快捷键 Alt + / 不能使用的问题
在`window -> Preferences -> general -> keys`中，
找到 content asist 修改下边值
Binding 改成 Alt+/   		输入的时候要注意，长按Alt键，然后按下/键即可
When 改为 Editing Text 
ok.


### ubuntu 使用`switchHosts`修改host后启动`eclipse`项目报错
``` java
错误: 抛出异常错误: `java.net.MalformedURLException: Local host name unknown: java.net.UnknownHostException:
localhost: hewentian-Lenovo-IdeaPad-Y470`: 未知的名称或服务
Error: Exception thrown by the agent : java.net.MalformedURLException: Local host name unknown: 
java.net.UnknownHostException: localhost: hewentian-Lenovo-IdeaPad-Y470: Name or service not known
```
原因是/etc/hosts文件里没有主机名为：hewentian-Lenovo-IdeaPad-Y470，解决方法就是在`switchHosts`中增加如下代码：
``` xml
127.0.0.1  hewentian-Lenovo-IdeaPad-Y470
localhost  hewentian-Lenovo-IdeaPad-Y470
```


### Linux查看程序端口占用情况
``` bash
$ netstat -apn | grep 8761
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 :::8761                 :::*                    LISTEN      24831/java      
tcp6       0      0 127.0.0.1:49958         127.0.0.1:8761          ESTABLISHED 24824/java      
tcp6       0      0 127.0.0.1:50494         127.0.0.1:8761          TIME_WAIT   -               
tcp6       0      0 127.0.0.1:50502         127.0.0.1:8761          TIME_WAIT   -               
tcp6       0      0 127.0.0.1:50484         127.0.0.1:8761          TIME_WAIT   -               
tcp6       0      0 127.0.0.1:8761          127.0.0.1:49958         ESTABLISHED 24831/java      
unix  3      [ ]         STREAM     CONNECTED     2887618  21883/chrome        
unix  3      [ ]         STREAM     CONNECTED     2887619  22047/chrome --type 
```
其中最后一栏是PID/Program name 

### ubuntu下安装notepadqq
安装过程中参考了如下文章：
https://www.linuxtechi.com/notepadqq-notepad-for-ubuntu-linux/

Step:1 Add Ubuntu PPA Repository

‘notpadqq‘ package is not available in the default Ubuntu repository, we will add Ubuntu PPA repository using below command.
``` bash
$ sudo add-apt-repository ppa:notepadqq-team/notepadqq
```
Step:2 Refresh the Repositories using below apt-get command.
``` bash
$ sudo apt-get update
```
Step:3 Install notepaddqq Debian package
``` bash
$ sudo apt-get install notepadqq
```
成功。




### ubuntu Navicat for MySQL 安装以及破解方案  

首先上官网上下载LINUX版本： http://www.navicat.com/download/navicat-for-mysql

1. 下载 navicat120_mysql_en_x64.tar.gz 文件

2. 下载后解压tar文件
``` bash
tar -xzvf /home/hewentian/Downloads/navicat120_mysql_en_x64.tar.gz
```
3. 解压后  进入解压后的目录运行命令：
``` bash
./start_navicat   
```
这样OK啦

连接上数据库后里面的中文数据是乱码,把Ubuntu的字符集修改为`zh_CN.utf8`就行了,修改方法:

1. 查看当前系统的字符集：echo $LANG
2. 查看系统支持的字符集: locale -a  
3. 修改字符集: export LANG=zh_CN.utf8  
4. 打开start_navicat文件，会看到 export LANG="en_US.UTF-8" 将这句话改为 export LANG="zh_CN.UTF-8"

经过以上修改，我的还是有中文乱码。

破解试用期方案：
第一次执行start_navicat时，会在用户主目录下生成一个名为.navicat的隐藏文件夹。
``` bash
cd /home/hewentian/.navicat/  
```
此文件夹下有一个system.reg文件
``` bash
rm system.reg
```
把此文件删除后，下次启动navicat 会重新生成此文件，14天试用期会按新的时间开始计算。


### linux中解压rar文件
可以先执行如下命令，看系统是否已经安装 rar 命令
``` bash
hewentian@hewentian-Lenovo-IdeaPad-Y470:~/Downloads$ rar
The program 'rar' is currently not installed. You can install it by typing:
sudo apt install rar
```
如果输出如上面，则`Linux`中未安装`rar`命令，按提示安装即可
``` bash
hewentian@hewentian-Lenovo-IdeaPad-Y470:~/Downloads$ sudo apt install rar
[sudo] password for hewentian: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Suggested packages:
  unrar
The following NEW packages will be installed:
  rar
0 upgraded, 1 newly installed, 0 to remove and 142 no
```
等待安装完成即可，验证是否成功安装，输入`rar`即可，看到如下输出，即证明安装成功。
``` bash
hewentian@hewentian-Lenovo-IdeaPad-Y470:~/Downloads$ rar

RAR 5.30 beta 2   Copyright (c) 1993-2015 Alexander Roshal   4 Aug 2015
Trial version             Type RAR -? for help

Usage:     rar <command> -<switch 1> -<switch N> <archive> <files...>
               <@listfiles...> <path_to_extract\>

<Commands>
  a             Add files to archive
  c             Add archive comment
  ch            Change archive parameters
  cw            Write archive comment to file
  d             Delete files from archive

```
解压方法很简单：
``` bash
hewentian@hewentian-Lenovo-IdeaPad-Y470:~/Downloads$ rar x 要解压的文件名.rar
```
这将压缩包解压在当前目录下




