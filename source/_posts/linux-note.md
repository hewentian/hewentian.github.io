---
title: Linux 学习笔记
date: 2017-09-12 20:41:37
tags: Linux
categories: Linux
---

在安装ubuntu的过程中的分区顺序: /boot -> / -> swap:

	/boot	primary		512M
	/		primary		剩下的容量减去内存2倍
	swap	logic		内存2倍

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

### useradd与adduser区别
``` bash
$ adduser username	# 会在/home下建立一个文件夹username
$ useradd username	# 不会在/home下建立一个文件夹username
$ useradd -m username	# 跟adduser一样，会在/home下建立一个文件夹username
```

### userdel删除用户账号
userdel 会查询系统账户文件，例如`/etc/password`和`/etc/group`，它会删除所有和用户名相关的文件
``` bash
$ userdel peter # 不带选项使用 userdel，只会删除用户。用户的家目录将仍会在/home目录下
$ userdel -r peter # 使用 -r 选项，在删除用户时将完全删除家目录、用户的邮件池
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
第一次执行start_navicat时，会在用户主目录下生成一个名为`.navicat64`的隐藏文件夹，只要重新删除该文件即可重新计算14天试用期。
``` bash
$ cd /home/hewentian
$ ls -a 				# 显示隐藏文件内的所有文件
$ rm -rf .navicat64 	# 删除该文件
$ 						# 或者删除此目录下的system.reg文件，但是没有效果
```
把文件夹删除后，下次启动navicat 会重新生成此文件，14天试用期会按新的时间开始计算。


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



### 当安装软件的时候出现如下错误的时候
``` bash
linux-image-extra-4.10.0-38-generic depends on linux-image-4.10.0-38-generic; however:
  Package linux-image-4.10.0-38-generic is not configured yet.
```
解决方法如下：
``` bash
$ sudo dpkg --configure -a
```


### 在Ubuntu上检查一个软件包是否安装

要检查特定的包，比如firefox是否安装了，使用这个命令：
``` bash
$ dpkg -s firefox
```
同样，你可以使用dpkg-query 命令。这个命令会有更好的输出，当然，你可以用通配符。
``` bash
$ dpkg-query -l firefox
```
要列出你系统中安装的所有包，输入下面的命令：
``` bash
$ dpkg --get-selections
```
你同样可以通过grep来过滤割到更精确的包。比如，我想要使用dpkg命令查看系统中安装的gcc包：
``` bash
$ dpkg --get-selections | grep gcc
```

### ubuntu 解压zip文件出现乱码
由于zip格式中并没有指定编码格式，Windows下生成的zip文件中的编码是GBK/GB2312等，因此，导致这些zip文件在Linux下解压时出现乱码问题，因为Linux下的默认编码是UTF8

有2种方式解决问题：

1. 通过`unzip`命令解压，指定字符集
unzip -O CP936 {要解压的文件名}.zip (用GBK, GB18030也可以)

2. 在环境变量中，指定unzip参数，总是以指定的字符集显示和解压文件
/etc/environment中加入2行
UNZIP="-O CP936"
ZIPINFO="-O CP936"


### 在 Ubuntu 上面远程连回 Windows 7
1. 首先需要开启 Windows 7 上的远程桌面: 打开[控制面板] -> [管理工具] -> [服务] -> [Terminal Services], 开启该服务（有可能找不到）;
2. 然后右击 [我的电脑] -> 选择[属性] -> 远程设置 -> 远程 -> 选择远程协助中的 [允许远程协助连接这台计算机]，下面的远程桌面选择 [允许运行任意版本远程桌面的计算机连接];
3. 有可能还需要关闭[防火墙];
4. 在 Ubuntu 上面使用如下命令连回 Windows 7; 
``` bash
$ rdesktop {Windows7_IP}
或者 
$ rdesktop {Windows7_IP} -f -u {YOUR_LOGIN_NAME} -p {YOUR_PASSWD} 
# -f 全屏，直接输入用户名和密码, 以全屏方式进入 windows 的退出方式是 [开始] -> [断开连接]
```
如果没有执行步骤2，则会报如下错：
``` bash
Autoselected keyboard map en-us
ERROR: CredSSP: Initialize failed, do you have correct kerberos tgt initialized ?
Failed to connect, CredSSP required by server.
```

### Linux下的任务调度分为两类，系统任务调度和用户任务调度。
1. 系统任务调度：系统周期性所要执行的工作，比如写缓存数据到硬盘、日志清理等。在/etc目录下有一个crontab文件，这个就是系统任务调度的配置文件。
/etc/crontab文件包括下面几行：
``` bash
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

2. 用户任务调度：用户定期要执行的工作，比如用户数据备份、定时邮件提醒等。用户可以使用 crontab 工具来定制自己的计划任务。所有用户定义的crontab 文件都被保存在 /var/spool/cron目录中。其文件名与用户名一致

查看crontab服务状态：
``` bash
$ /etc/init.d/cron status  # 可用的参数 force-reload | reload | restart | start | status | stop
```
### crontab 的用法
``` bash
usage:	crontab [-u user] file
	crontab [ -u user ] [ -i ] { -e | -l | -r }
		(default operation is replace, per 1003.2)
	-e	(edit user's crontab)
	-l	(list user's crontab)
	-r	(delete user's crontab)
	-i	(prompt before deleting user's crontab)
```
例子如下：
``` bash
$ crontab -e
```
在打开的文件中输入如下内容，这将会在每分钟往文件中输出时间
``` bash
*/1 * * * * /bin/date >> /home/hewentian/Documents/a.txt
```

在资源浏览器 [Connect to Server] 窗口输入：smb://需要访问的机器IP
等同于Windows下输入：\\IP


### curl模拟http发送get或post请求
参照：http://www.voidcn.com/blog/Vindra/article/p-4917667.html

1. get请求 
curl "http://www.baidu.com"  如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
curl -i "http://www.baidu.com"  显示全部信息
curl -l "http://www.baidu.com" 只显示头部信息
curl -v "http://www.baidu.com" 显示get请求全过程解析

wget "http://www.baidu.com"也可以

2. post请求
curl -d "param1=value1&param2=value2" "http://www.baidu.com"

3. json格式的post请求
curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13800138000","password":"passwd"}' http://domain/apis/users.json

### 在ubuntu上面安装 maven
``` bash
$ su root 	# 切换到 root 用户
$ cd /home/hewentian/Downloads	# 进入到 maven 下载的目录
$ tar xzvf apache-maven-3.3.9-bin.tar.gz
# cd /usr/local
# mv /home/hewentian/Downloads/apache-maven-3.3.9 ./
$# 在 /etc/profile 文件中添加 maven 路径
# vi /etc/profile
$ # 在打开的文件中添加如下代码
# add maven
export M2_HOME=/usr/local/apache-maven-3.3.9
export PATH=$M2_HOME/bin:$PATH

保存，并退出，执行如下语句，验证安装结果
$ sour$ce /etc/profile
$ mvn -version

$ # 如无意外，你将看到如下输出
Maven home: /usr/local/apache-maven-3.3.9
Java version: 1.8.0_102, vendor: Oracle Corporation
Java home: /usr/local/java/jdk1.8.0_102/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.10.0-28-generic", arch: "amd64", family: "unix"
```

### ubuntu 16.04 安装 google-chrome-stable_current_amd64.deb 方法
``` bash
$ cd {google-chome所在目录}
$ sudo dpkg -i google-chrome-stable_current_amd64.deb
$ # 如果安装过程输出如下错误
$ # google-chrome-stable depends on libappindicator1; however:
$ # 那么执行如下命令
$ sudo apt-get -f install libappindicator1 libindicator7
$ # 然后再执行原来的安装命令
$ sudo dpkg -i google-chrome-stable_current_amd64.deb
```

### Ubuntu 16.04 关闭笔记本触摸板
1. 禁用触摸板
``` bash
$ sudo rmmod psmouse
```

2. 重启触摸板
``` bash
$ sudo modprobe psmouse
```

### linux下查看某命令的具体位置使用which，如：
``` bash
$ which ab
```


### 在Linux/unix 平台上的`sqlplus`中，如果输错了字符，要想删除，习惯性的按下`backspace`键后，发现非但没有删除想要删掉的字符，还多出了两个字符^H。当然，我们可以同时按下`ctrl+backspace`键来删除，但对于习惯了用`backspace`来删除的用户，这样很不爽。这可以通过修改tty终端的设置来实现`backspace`删除功能。通过使用stty命令，就可以查看或者修改终端的按键设置。
例如，设置backspace为删除键：
``` bash
$ stty erase ^h
```
如果要改回使用ctrl+backspace为删除键
``` bash
$ stty erase ^?
```
如果需要重启后自动设置终端，可以将上述命令加入到profile中。可以通过stty -a命令来查看所有的终端设置。下面是在linux下执行的输出：
``` bash
$ stty -a
$ cat /etc/os-release
```

### 关于Linux中的so文件
1. so文件就是通常说的动态链接库，就跟windows下的dll文件差不多；
2. ko是内核模块文件，驱动之类的啥的；
3. 不过在linux系统下文件的后缀多数情况下只是个标识，有可能代表不了文件的真实属性的。


### linux下采用LD_PRELOAD机制动态修改方法和注入代码
LD_PRELOAD是Linux下的一个环境变量，动态链接器在载入一个程序所需的所有动态库之前，
首先会载入LD_PRELOAD环境变量所指定的动态库。运用这个机制，我们可以修改/替换已有动态库中的方法，
加入我们自己的逻辑，从而改变程序的执行行为。不过该方法只对动态链接的程序有效，对静态链接的程序无效。


### linux下面查看文件的MD5值，用命令：md5sum
用法一，产生MD5值：
``` bash
$ md5sum [文件]
```
用法二，从文件中读取MD5的校验值并予以检查，要求同目录下面要有如下两个文件：

	solr-6.5.0.zip
	solr-6.5.0.zip.md5

命令如下：
``` bash
$ md5sum -c solr-6.5.0.zip.md5
如果相同，输出：
solr-6.5.0.zip: OK
```
md5值重定向将生成md5值重定向到指定的文件，通常文件的扩展名我们会命为.md5
``` bash
$ md5sum data > data.md5
$ md5sum data
0a6de444981b68d6a049053296491e49  data

$ cat data.md5 
0a6de444981b68d6a049053296491e49  data
```

### linux查找一个目录的命令
``` bash
查找目录：find /（查找范围） -name '查找关键字' -type d
```

### Linux查找含有某字符串的所有文件
	grep -rn "hello,world!" *
 
	* : 表示当前目录所有文件，也可以是某个文件名
	-r 是递归查找
	-n 是显示行号
	-R 查找所有文件包含子目录
	-i 忽略大小写

### md5sum的使用
``` bash
对一个文件计算MD5值
$ md5sum apache-tomcat-8.0.47.tar.gz
0fd249576c33fa71947bc7d296d5b0a9  apache-tomcat-8.0.47.tar.gz

对一个文件计算MD5值，并保存到文件中
$ md5sum apache-tomcat-8.0.47.tar.gz > apache-tomcat-8.0.47.tar.gz.md5

用MD5文件校验原来的文件是否被修改过
$ md5sum -c apache-tomcat-8.0.47.tar.gz.md5
apache-tomcat-8.0.47.tar.gz: OK
```

### 端口的分类： 
端口在报头中占两个字节，也就是16位。端口号用来表示和区别网络中的不同应用程序。端口分为三大类： 

1. 公认端口（Well Known Ports）：0-1023之间的端口号，在LINUX中这些端口你不能随便使用。这些端口由 IANA 分配管理。IANA 把这些端口分配给最重要的一些应用程序，让所有的用户都知道，当一种新的应用程序出现后，IANA必须为它指派一个公认端口。 常用的公认端口有：

		FTP:	21
		TELNET:	23
		SMTP:	25
		DNS:	53
		TFTP:	69
		HTTP:	80
		SNMP:	161

2. 注册端口（Registered Ports）：从1024-49151，是公司和其他用户向互联网名称与数字地址分配机构（ICANN）登记的端口号，利用因特网的传输控制协议（TCP）和用户数据报协议（UDP）进行通信的应用软件需要使用这些端口。在大多数情况下，这些应用软件和普通程序一样可以被非特权用户打开。

3. 客户端使用的端口号：49152~65535，这类端口号仅在客户进程运行时才动态选择，因此又叫做短暂端口号。被保留给客户端进程选择暂时使用的。也可以理解为，客户端启动的时候操作系统随机分配一个端口用来和服务器通信，客户端进程关闭下次打开时，又重新分配一个新的端口。

端口就像一道门，外部可以通过不同的端口和本机上不同服务的进程进行交流。而IP 地址和端口号标识了接入互联网主机的唯一 一个进程


### ubuntu 中使用 SQuirreL 连接各种 RDBMS
参考：http://www.opensoce.com/367.html

使用`SQuirreL SQL Client`连接各种`RDBMS`数据库，其官方网站为：http://sourceforge.net/projects/squirrel-sql，
截至写本文时，其最新版本为：squirrel-sql-snapshot-20171121_2249，其最大的魅力在于：

1. 基于Java，具备良好的夸平台特性，在Windows下一样可以很好的使用；
2. 只要具备相应数据库的类库，就可以连接对应的数据库，例如SQL Server、MySQL、Oracle、Sybase、DB2等等。

首先下载：
http://sourceforge.net/projects/squirrel-sql

然后确保系统已经安装java，然后切换到`squirrel-sql-snapshot-20171121_2249-standard.jar`所在目录执行：
``` bash
$ java -jar squirrel-sql-snapshot-20171121_2249-standard.jar
```
然后根据提示选择安装路径，我安装到`/home/hewentian/ProjectD/squirrel-sql-snapshot-20171121_2249`，后续安装步骤中，插件我选择了

	Data import
	DBCopy
	DBDiff
	Microsoft SQL Server
	MySQL
	Oracle
安装完成之后，需要去下载相应的`RDBMS`的jar包：
SQL Server(jtds):
http://sourceforge.net/projects/jtds/files/

mysql(MySQL Connector):
http://dev.mysql.com/downloads/connector/j/

oracle(JDBC):
http://www.oracle.com/technology/global/cn/software/tech/java/sqlj_jdbc/index.html

然后将`jtds-1.3.1.jar`、`ojdbc14.jar`、`mysql-connector-java-5.1.25.jar`复制到`SQuirrel SQL`安装目录下的`lib`目录，这样`SQuirrel SQL`就具备了连接`SQL Server`、`Oracle`、`MySQL`的能力。

最后，到`SQuirreL`的目录重启SQuirreL：
``` bash
$ cd /home/hewentian/ProjectD/squirrel-sql-snapshot-20171121_2249
$ ./squirrel-sql.sh
```
之后在左侧Aliases中添加一个Alias，填入正确的连接信息内容类似于：

	Name: 输入连接的名字
	Driver: jTDS Microsoft SQL
	URL: 
	User Name: 
	Password: 
这样就可以连接了。


### 修改SSH的默认22端口
``` bash
首先查看是否有使用22端口
$ lsof -i:22
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd     934 root    3u  IPv4   8688      0t0  TCP *:ssh (LISTEN)
sshd    2305 root    3r  IPv4  25836      0t0  TCP 123.57.238.142:ssh->66.133.135.219.broad.gz.gd.dynamic.163data.com.cn:60064 (ESTABLISHED)

查看防火墙的情况，看下是否会允许你要修改的端口通过
$ iptables -nL

$ vim /etc/ssh/sshd_config
找到Port这一行，修改为你想要的端口，这里设置为：12022，前提是该端口还没被其他程序使用

重启SSH
$ /etc/init.d/sshd restart
Stopping sshd:                                             [  OK  ]
Starting sshd:                                             [  OK  ]

$ lsof -i:12022
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    2382 root    3u  IPv4  28005      0t0  TCP *:12022 (LISTEN)
sshd    2388 root    3r  IPv4  28216      0t0  TCP 123.57.238.142:12022->66.133.135.219.broad.gz.gd.dynamic.163data.com.cn:41068 (ESTABLISHED)

可以看到端口已经修改过来了，再使用之前的命令就无法登录了
$ ssh -p 22 root@123.57.238.142
ssh: connect to host 123.57.238.142 port 22: Connection refused
```

### ubuntu 连接到跳板机
先将密钥文件`hewentian.pem`放到用户SSH目录下
``` bash
$ cd ~/.ssh
$ cp ~/Downloads/hewentian.pem ./

修改权限 
$ chmod 600 hewentian.pem

创建一个配置文件，如果没有的话
$ touch config

并在其中输入如下内容
Host jump_server
User hewentian
Hostname 192.168.30.30
Port 12022
PreferredAuthentications publickey
IdentityFile ~/.ssh/hewentian.pem
```
这样，在使命行中输入: `ssh jump_server`就可以跳到跳板机了


### ubuntu 下 zssh 的使用
首先，ubuntu下需要安装下面两个包，如果还未安装的话：
``` bash
$ sudo apt install lszrz
$ sudo apt install zssh
```

1. 使用 zssh 替代 ssh 连接到目标系统，并登入：
``` bash
zssh jump_server
```

2. 发送文件到目标系统。比如，我们要上传a.txt文件
``` bash
首先进入目标机的要上传到的目录，这里进入用户根目录下的Downloads
$ cd ~/Downloads
（然后，按 ctrl + @ 进入文件传输状态，这个时候会浏览本地机器的文件系统了）
zssh > ls
a.txt
要将a.txt上传到目标机，执行如下命令即可
zssh > sz a.txt
```

3. 下载文件到本地。比如，我们想从目标系统下载 ~/Downloads/a.txt 到本地
``` bash
$ cd ~/Downloads
$ ls
a.txt

$ sz a.txt 
�B00000000000000
（按 ctrl + @ 进入文件传输状态）
zssh > rz
Receiving: a.txt                                                     
Bytes received:       7/      7   BPS:1198                  

Transfer complete

$ 
```
在目标系统输入 sz 时，我们开启了文件发送，此处可能会有乱码，暂时不管；然后，按 Ctrl+@ 进入文件传输模式，输入 rz 并回车进行文件下载，下载完成后，自动退出文件传输模式

在自己的linux机上，如ubuntu等，安装上zssh，先用zssh登陆上跳板机，再在跳板机上ssh到相应服务器，然后ctrl+@,就可以相应上传下载文件了，先记着，后续再补详细资料。
``` bash
上传本地文件到服务器
在服务器上先cd至相应要放上传文件的目录之后

rz -bye                 //在远程服务器的相应目录上运行此命令,表示做好接收文件的准备
ctrl+@                  //运行上面命令后,会出现一些乱码字符,不要怕,按此组合键,进入zssh
zssh >                  //这里切换到了本地机器
zssh > pwd           //看一下本地机器的目录在那
zssh > ls               //看一下有那些文件
zssh > sz 123.txt   //上传本地机器的当前目录的123.txt到远程机器的当前目录

下载服务器文件到本地
sz filename             //在远程机器上,启动sz, 准备发送文件
                               //看到一堆乱码,不要怕,这会按下组合键
ctrl+@
zssh > pwd              //看看在那个目录,cd 切换到合适的目录
zssh > rz -bye                 //接住对应的文件
```


### Linux下打开ISO文件方法
Linux下用mount挂载命令
``` bash
$ mount -o loop /home/hewentian/Downloads/ubuntu.iso /mnt/cdrom
```

取消挂载
``` bash
$ umount /mnt/cdrom
```


### Linux 统计当前文件夹下的文件个数、目录个数
1) 统计当前文件夹下文件的个数
``` bash
$ ls -l | grep "^-" | wc -l
```

2) 统计当前文件夹下目录的个数
``` bash
$ ls -l | grep "^d" | wc -l
```

3) 统计当前文件夹下文件的个数，包括子文件夹里的 
``` bash
$ ls -lR | grep "^-" | wc -l
```

4) 统计文件夹下目录的个数，包括子文件夹里的
``` bash
$ ls -lR | grep "^d" | wc -l
```

说明：
``` bash
$ ls -l
```
长列表输出当前文件夹下文件信息(注意这里的文件，不同于一般的文件，可能是目录、链接、设备文件等)
 
``` bash
$ grep "^-" 
```
这里将长列表输出信息过滤一部分，只保留一般文件，如果只保留目录就是 ^d
``` bash
$ wc -l
```
统计输出信息的行数，因为已经过滤得只剩一般文件了，所以统计结果就是一般文件信息的行数，又由于一行信息对应一个文件，所以也就是文件的个数。


### 安装opevpn
``` bash
首先安装openvpn
$ sudo apt install openvpn

然后启动，其中~/Downloads/hewentian是你的公司下发的你自已的登录帐号相关的文件
$ sudo openvpn ~/Downloads/hewentian/hewentian.ovpn 
```
这样，你在家里也可以连回公司的网络


### linux 查询 CPU 信息
``` bash
$ cat /proc/cpuinfo | grep "model name" && cat /proc/cpuinfo | grep "physical id"

model name	: Intel(R) Core(TM) i3-2350M CPU @ 2.30GHz
model name	: Intel(R) Core(TM) i3-2350M CPU @ 2.30GHz
model name	: Intel(R) Core(TM) i3-2350M CPU @ 2.30GHz
model name	: Intel(R) Core(TM) i3-2350M CPU @ 2.30GHz
physical id	: 0
physical id	: 0
physical id	: 0
physical id	: 0

```


### linux 查询 内存大小
``` bash
$ cat /proc/meminfo | grep MemTotal

MemTotal:        8115068 kB
```


###  linux 查询 硬盘大小
``` bash
$ fdisk -l | grep Disk

Disk /dev/sda: 119.2 GiB, 128035676160 bytes, 250069680 sectors
Disklabel type: dos
Disk identifier: 0x07462b5e
Disk /dev/sdb: 465.8 GiB, 500107862016 bytes, 976773168 sectors
Disklabel type: dos
Disk identifier: 0x547aca8b
```


### 查看内存使用量和交换区使用量
``` bash
$ free -m
              total        used        free      shared  buff/cache   available
Mem:           7924        5783         472         331        1668        1460
Swap:         15623        1176       14447
```

### 查看各分区使用情况
``` bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           793M  9.4M  784M   2% /run
/dev/sda2       102G   27G   71G  28% /
tmpfs           3.9G  130M  3.8G   4% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1       464M  174M  263M  40% /boot
tmpfs           793M  104K  793M   1% /run/user/1000
```

### 查看指定目录的大小
``` bash
$ du -sh
4.5G	.

还可以另上指定文件，这样就计算那个文件或目录的大小
$ du -sh ROOT.zip 
67M	ROOT.zip
```


### gpg 验证已下载文件的真实性和完整性
首先检查系统是否已经安装GPG
``` bash
$ dpkg-query -l gnupg*
```
否则使用如下命令安装(ubuntu)
``` bash
$ sudo apt-get install gnupg
```
安装完毕后，生成密钥对，要使用到你的用户名和邮箱，以及保护你私钥的密码，如下：
``` bash
$ gpg --gen-key
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Tim Ho
Email address: wentian.he@qq.com
Comment: 
You selected this USER-ID:
    "Tim Ho <wentian.he@qq.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
.........+++++
+++++
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
+++++

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 73 more bytes)
+++++
gpg: key 46DC4BA0 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
pub   2048R/46DC4BA0 2018-01-14
      Key fingerprint = 06F1 0995 A7FE DF6F 007F  CE59 B307 E8C8 46DC 4BA0
uid                  Tim Ho <wentian.he@qq.com>
sub   2048R/74EB06EB 2018-01-14

```
密钥生成完毕后，公钥和私钥都将存储在~/.gnupg目录中，供之后使用。

导入文件所有者的公钥
``` bash
$ gpg --import rabbitmq-release-signing-key.asc 
gpg: key 6026DFCA: "RabbitMQ Release Signing Key <info@rabbitmq.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
```
当所有者的公钥导入完毕，它会输出一个密钥编号(比如“6026DFCA”)，如上所示

现在，运行这个命令，检查已导入公钥的指纹：
``` bash
$ gpg --fingerprint 6026DFCA
pub   4096R/6026DFCA 2016-05-17
      Key fingerprint = 0A9A F211 5F46 87BD 2980  3A20 6B73 A36E 6026 DFCA
uid                  RabbitMQ Release Signing Key <info@rabbitmq.com>
sub   4096R/12EBCE19 2016-05-17
```
检查上面的指纹是否与所有者官网提供的一致。

下面的步骤可选，
``` bash
$ gpg --edit-key 6026DFCA
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  4096R/6026DFCA  created: 2016-05-17  expires: never       usage: SC  
                     trust: full          validity: unknown
sub  4096R/12EBCE19  created: 2016-05-17  expires: never       usage: E   
[ unknown] (1). RabbitMQ Release Signing Key <info@rabbitmq.com>
```
在GPG提示符下键入“trust”，这会让你可以选择该密钥的信任级别：从1到5，这里选择4
``` bash
gpg> trust
pub  4096R/6026DFCA  created: 2016-05-17  expires: never       usage: SC  
                     trust: full          validity: unknown
sub  4096R/12EBCE19  created: 2016-05-17  expires: never       usage: E   
[ unknown] (1). RabbitMQ Release Signing Key <info@rabbitmq.com>

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 4

pub  4096R/6026DFCA  created: 2016-05-17  expires: never       usage: SC  
                     trust: full          validity: unknown
sub  4096R/12EBCE19  created: 2016-05-17  expires: never       usage: E   
[ unknown] (1). RabbitMQ Release Signing Key <info@rabbitmq.com>

gpg> sign

pub  4096R/6026DFCA  created: 2016-05-17  expires: never       usage: SC  
                     trust: full          validity: unknown
 Primary key fingerprint: 0A9A F211 5F46 87BD 2980  3A20 6B73 A36E 6026 DFCA

     RabbitMQ Release Signing Key <info@rabbitmq.com>

Are you sure that you want to sign this key with your
key "Tim Ho <wentian.he@qq.com>" (E99DCF91)

Really sign? (y/N) y

You need a passphrase to unlock the secret key for
user: "Tim Ho <wentian.he@qq.com>"
2048-bit RSA key, ID E99DCF91, created 2018-01-14

gpg> save
```
然后sign，最后save，这里输入的passphrase错误，与我之前的密码不符，未明原因。
最后，查看导入的keys
``` bash
$ gpg --list-keys
/home/hewentian/.gnupg/pubring.gpg
----------------------------------
pub   2048R/E99DCF91 2018-01-14
uid                  Tim Ho <wentian.he@qq.com>
sub   2048R/6F7627AD 2018-01-14

pub   4096R/6026DFCA 2016-05-17
uid                  RabbitMQ Release Signing Key <info@rabbitmq.com>
sub   4096R/12EBCE19 2016-05-17

pub   2048R/46DC4BA0 2018-01-14
uid                  Tim Ho <wentian.he@qq.com>
sub   2048R/74EB06EB 2018-01-14
```
最后：验证文件的真实性/完整性
``` bash
$ gpg --verify rabbitmq-server_3.7.2-1_all.deb.asc rabbitmq-server_3.7.2-1_all.deb
gpg: Signature made Sat 23 Dec 2017 03:03:34 PM CST using RSA key ID 6026DFCA
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
gpg: Good signature from "RabbitMQ Release Signing Key <info@rabbitmq.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 0A9A F211 5F46 87BD 2980  3A20 6B73 A36E 6026 DFCA
```
该命令的输出里面含有“Good signature from ”，这表明已下载的.deb文件已成功通过了验证。要是已下载文件在签名生成后以任何一种方式而遭到篡改，验证就会失败。


### Linux如何运行.AppImage文件
AppImage是新型的打包软件，它可以解决Linux上面的依赖问题。在使用上面相比其他的软件使用极为简单。
首先给它增加可执行权限，然后执行即可。如下：
``` bash
$ chmod +x {AppImage文件}
$ ./{AppImage文件}
```

### yum切换为阿里的源
有时候，我们下载软件慢，可以切换到阿里的yum.
首先备份现有的源，以便还原。
``` bash
$ cd /etc/yum.repos.d/
$ mv CentOS-7.repo CentOS-7.repo_bak
```
下载阿里的源
``` bash
$ wget -O /etc/yum.repos.d/CentOS-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
然后执行如下命令即可
``` bash
$ yum remove epel-release
$ yum clean all
$ yum makecache
$ yum -y install epel-release
```

### 在处理python图片的时候，如果提示某些属性不可用，可以升级Pillow
``` bash
$ pip uninstall Pillow
$ pip install --upgrade pip
$ pip install Pillow
```

### linux合并文件
可以使用`cat`命令从文件中读入两个文件，然后将其重定向到一个新的文件。
用法示例，将`file1.txt`和`file2.txt`合并到`file.txt`：
``` bash
$ cat file1.txt file2.txt > file.txt
```
其中，`file.txt`可以不存在，会被自动创建。`file1.txt`和`file2.txt`的顺序决定了它们在`file.txt`中的顺序。

也可以只使用`cat`命令读入一个文件，然后使用`>>`将文本流追加到另一个文件的末位。
用法示例，将`file1.txt`追加到`file2.txt`的末尾：
``` bash
$ cat file1.txt >> file2.txt
```

### Linux scp命令
Linux scp命令用于Linux之间复制文件和目录。
scp是 secure copy的缩写, scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令
1、从本地复制到远程
``` bash
命令格式：
scp local_file remote_username@remote_ip:remote_folder 
或者 
scp local_file remote_username@remote_ip:remote_file 
或者 
scp local_file remote_ip:remote_folder 
或者 
scp local_file remote_ip:remote_file 

复制目录命令格式：
scp -r local_folder remote_username@remote_ip:remote_folder 
或者 
scp -r local_folder remote_ip:remote_folder 
```