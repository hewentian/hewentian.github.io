---
title: Linux 学习笔记
date: 2017-09-12 20:41:37
tags: Linux
categories: Linux
---

在安装ubuntu的过程中的分区顺序: EFI -> /boot -> swap -> /:

        EFI     primary     200M
        /boot   primary     512M
        swap    logic       内存2倍
        /       primary     剩下的容量

例如在小米笔记本安装 ubuntu 18.04 的时候，必须关闭安全启动，secure BOOT, 
重启电脑，出现mi时，F2进入BIOS中，在secure中关闭安全启动，对了，要首先设置superviser password， 否则secure boot 无法关闭。 

Ubuntu安装ssh时出现软件包 openssh-server 还没有可供安装的候选者错误

错误如下：

sudo apt-get install opensshserver正在读取软件包列表...

完成正在分析软件包的依赖关系树正在读取状态信息...

完成现在没有可用的软件包 openssh-server，

但是他被其他的软件包引用了这可能意味着这个缺失的软件包可能已被废弃，或者只能在其他发布源中找到

Error:软件包 openssh-server 还没有可供安装的候选者

解决方案：分析原因是我们的apt-get没有更新，当然如果你的是最新的系统不用更新也行，但是我相信很多人都是需要更新的吧，操作命令如下：
``` bash
$ sudo apt-get update
```
更新完毕后执行：
``` bash
$ sudo apt-get install openssh-client
$ sudo apt-get install openssh-server
```

安装完毕之后，启动SSH
``` bash
$ sudo service ssh start
```

最后我们用命令`ps -ef | grep ssh`来看下open-server安装成功没有，如果出现像下面的输出，说明安装成功了。
``` bash
$ ps -ef | grep ssh

hewenti+  2654  2565  0 10:04 ?        00:00:00 /usr/bin/ssh-agent /usr/bin/im-launch env GNOME_SHELL_SESSION_MODE=ubuntu gnome-session --session=ubuntu
hewenti+  8845  6801  0 10:29 pts/1    00:00:00 grep --color=auto ssh
```
这样就可以在其他机器连到这台机器或者执行scp命令了。


在ubuntu中的vi编辑器怎么使用
默认情况下ubuntu上也安装有vi但是奇怪的是这个vi是vim-common版本，基本上用不了所以要先把这个版本的vi卸载掉，然后重新安装vim。
``` bash
$ sudo apt-get remove vim-common
$ sudo apt-get install vim
```


### useradd与adduser区别
``` bash
$ adduser username	# 会在/home下建立一个文件夹username
$ useradd username	# 不会在/home下建立一个文件夹username
$ useradd -m username	# 跟adduser一样，会在/home下建立一个文件夹username
```

### userdel删除用户账号
userdel 会查询系统账户文件，例如`/etc/password`和`/etc/group`，它会删除所有和用户名相关的文件
``` bash
$ userdel peter     # 不带选项使用 userdel，只会删除用户。用户的目录将仍会在/home目录下
$ userdel -r peter  # 使用 -r 选项，在删除用户时将完全删除用户的目录、用户的邮件池
```

还有的就是文件编码的问题，在windows下默认都是用ANSI格式编码，但是在ubuntu下面可能会是其他格式的编码，如果两个系统的编码不一致，就会产生乱码。
最好是都采用UTF-8格式编码


ubuntu root默认密码（初始密码）
ubuntu安装好后，root初始密码（默认密码）是没有的，需要设置。

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

在ubuntu软件源里zlib和zlib-devel叫做zlib1g、zlib1g.dev
``` bash
$ sudo apt-get install zlib1g
$ sudo apt-get install zlib1g.dev
```

直接输入上述命令后还是不能安装。这就要求我们先装ruby，默认的安装源里没有zlib1g.dev。要在packages.ubuntu.com上找。
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

其他使用方法
``` bash
netstat -ntlp    // 查看当前所有tcp端口
netstat -ntulp | grep 9012    // 查看所有9012端口使用情况

lsof -i:8020    // 查看8020端口是否开启
```


### 检查远程端口是否可以连通
方法一：使用telnet

        telnet localhost <port>    // On the server, try to see if the port is open there
        telnet <server> <port>     // On the client, try to see if the port is accessible remotely

示例：
``` bash
$ telnet 192.168.8.112 8088

Trying 192.168.8.112...
Connected to 192.168.8.112.
Escape character is '^]'.
```

方法二：使用nc
``` bash
$ nc -v 192.168.8.112 8088
Connection to 192.168.8.112 8088 port [tcp/omniorb] succeeded!
```

redHat安装nc

        sudo yum install -y nc


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


### ubuntu安装mysql 8 client
``` bash
$ wget -c https://dev.mysql.com/get/mysql-apt-config_0.8.11-1_all.deb
$ sudo dpkg -i mysql-apt-config_0.8.11-1_all.deb
$ sudo apt-get update
$ sudo apt-get install mysql-client
```


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
$ ls -a              # 显示隐藏文件内的所有文件
$ rm -rf .navicat64  # 删除该文件
$                    # 或者删除此目录下的system.reg文件，但是没有效果
```
把文件夹删除后，下次启动navicat会重新生成此文件，14天试用期会按新的时间开始计算。


**注意**：现在已经用[DBeaver](https://dbeaver.io/)来代替`Navicat`了。我目前使用的是`dbeaver-ce-7.3.1-linux.gtk.x86_64.tar.gz`，它支持Mysql 8，解压后，就可以直接使用啦。


### linux中解压rar文件
可以先执行如下命令，看系统是否已经安装`rar`命令
``` bash
$ rar

The program 'rar' is currently not installed. You can install it by typing:
sudo apt install rar
```

如果输出如上面，则`Linux`中未安装`rar`命令，按提示安装即可
``` bash
$ sudo apt install rar
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
$ rar

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
$ rar x 要解压的文件名.rar
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
你同样可以通过grep来过滤找到更精确的包。比如，我想要使用dpkg命令查看系统中安装的gcc包：
``` bash
$ dpkg --get-selections | grep gcc
```

如果知道一个命令，比如mysql，要想知道它安装在哪里，使用
``` bash
$ whereis mysql
mysql: /usr/bin/mysql /etc/mysql /usr/share/man/man1/mysql.1.gz
```

然后找到它的安装包
``` bash
$ dpkg -S /usr/bin/mysql
mariadb-client-core-10.1: /usr/bin/mysql
```

然后卸载它
``` bash
$ sudo apt-get remove --purge mariadb-client-core-10.1
```


### ubuntu 解压zip文件出现乱码
由于zip格式中并没有指定编码格式，Windows下生成的zip文件中的编码是GBK/GB2312等，因此，导致这些zip文件在Linux下解压时出现乱码问题，因为Linux下的默认编码是UTF8

有2种方式解决问题：

1. 通过`unzip`命令解压，指定字符集

        unzip -O CP936 {要解压的文件名}.zip (用GBK, GB18030也可以)
        unzip -O CP936 {要解压的文件名}.zip -d {目标文件夹名}    # -d 指定解压到这个目录

2. 在环境变量中，指定unzip参数，总是以指定的字符集显示和解压文件

        vi /etc/environment

        # 加入下面2行
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

# 也可以将本地linux上面的一个目录，映射到要远程到的windows机。下面将/home/hewentian/Documents映射到远程windows上面的盘hwt
$ rdesktop {Windows7_IP} -f -u {YOUR_LOGIN_NAME} -p {YOUR_PASSWD} -r disk:hwt=/home/hewentian/Documents
```
如果没有执行步骤2，则会报如下错：
``` bash
Autoselected keyboard map en-us
ERROR: CredSSP: Initialize failed, do you have correct kerberos tgt initialized ?
Failed to connect, CredSSP required by server.
```
你也可以将这些命令，写成SHELL脚本
``` bash
$ touch rd.sh
$ chmod +x rd.sh
$ sh rd.sh
```
并在rd.sh中输入以下内容
``` bash
#! /bin/sh
echo "starting rdesktop ..."
echo ""
ipAddr=192.168.1.100
user=administrator
passwd=pw12345678
echo "rdesktop to $ipAddr"

rdesktop $ipAddr -u $user -p $passwd -g 1300*800 -r clipboard:PRIMARYCLIPBOARD -r disk:hwt=/home/hewentian/Documents

echo "succeeded..."
sleep 1
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

### 一个crontab备份数据的示例
``` bash
############################################################
# desc: backup important data. every saturday 23:50 run, and the cron is:
#       50 23 * * 6 /bin/sh /home/hewentian/backupData.sh >> /home/hewentian/backupData.log 2>&1
# author: Tim Ho
# mail: wentian.he@qq.com
# created time: 2019-07-24 10:22:28 AM
############################################################

#!/bin/sh

# create the base backup dir, if not exists
backupBaseDir=/home/hewentian/backupData/
if [ ! -d "$backupBaseDir" ]; then
  mkdir -p "$backupBaseDir"
fi

# create current backup dir, if not exists
backupDir=$backupBaseDir`date +%Y%m%d`"/"
if [ ! -d "$backupDir" ]; then
  mkdir "$backupDir"
fi

# begin to backup data
echo "-------------------- begin to backup data $(date) --------------------"

echo "start to backup userInfo, about 0.30G"
mongoexport -h 192.168.1.100 --port 27017 -u mongo_account -p mongo_account --authenticationDatabase mongo_account -d mongo_account -c userInfo -o "$backupDir"userInfo.json
echo -e "end to backup userInfo $(date)\n"

echo "start to backup carInfo, about 0.18G"
mongoexport -h 192.168.1.100 --port 27017 -u mongo_account -p mongo_account --authenticationDatabase mongo_account -d mongo_account -c carInfo -o "$backupDir"carInfo.json
echo -e "end to backup carInfo $(date)\n"

# at last, delete the files backup 7 days ago
find "$backupBaseDir" -type d -mtime +7 -exec rm -rf {} \;
#find "$backupBaseDir" -mtime +7 -name '*.json' -exec rm -rf {} \;

echo "-------------------- end to backup data $(date) --------------------"
```


### Ubuntu中访问共享文件夹和FTP
1. 访问共享文件夹：在资源浏览器`[Connect to Server]`窗口输入：smb://需要访问的机器IP，等同于Windows下输入：\\IP
2. 访问FTP机器：在资源浏览器`[Connect to Server]`窗口输入：ftp://192.168.1.128/
3. `[Connect to Server]`在`~/.config/nautilus/servers`中，可以编辑它的bookmark

![](/img/connect-to-server.png "Connect to Server")


### curl模拟http发送get或post请求
参照：http://www.voidcn.com/blog/Vindra/article/p-4917667.html

1. get请求

        curl "http://www.hewentian.com"     如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
        curl -i "http://www.hewentian.com"  显示全部信息
        curl -l "http://www.hewentian.com"  只显示头部信息
        curl -v "http://www.hewentian.com"  显示get请求全过程解析
        wget "http://www.hewentian.com"     也可以

2. post请求

        curl -d "param1=value1&param2=value2" "http://www.hewentian.com"

3. json格式的post请求

        curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13800138000","password":"passwd"}' http://www.hewentian.com/apis/users


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

卸载方法
$ sudo apt-get remove google-chrome-stable
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
	-v, --invert-match        select non-matching lines

还可以使用正则表达式：
    grep -E '[0-9]+ms' mongod.log

这个命令查询`mongod.log`日志中包含毫秒时间ms的所有日志。

如果我想查询`hello`或`world`：
    grep -E 'hello|world' mongod.log


### linux下面md5sum的使用
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
Hostname 192.168.1.100
Port 12022
PreferredAuthentications publickey
IdentityFile ~/.ssh/hewentian.pem
```
这样，在使命行中输入: `ssh jump_server`就可以跳到跳板机了

如果你有两台跳板机：开发环境、生产环境，你可以在`config`文件中同时配置两个：
``` bash
Host jump_dev
  User hewentian
  Hostname 192.168.1.100
  Port 12022
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/hewentian.pem

Host jump_prod
  User hewentian
  Hostname 111.112.113.114
  Port 12022
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/hewentian-prod.pem
```

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


### ubuntu 开放指定端口
一般情况下，ubuntu安装好的时候，iptables会被安装上，如果没有的话先安装

        sudo apt-get install iptables

添加开放端口
        sudo iptables -A INPUT -p tcp --dport 4412 -j ACCEPT
        sudo iptables -A OUTPUT -p tcp --sport 4412 -j ACCEPT

        # 临时保存配置，重启后失效
        sudo iptables-save

安装`iptables-persistent`工具，持久化开放端口配置
        sudo apt-get install iptables-persistent

        sudo netfilter-persistent save
        sudo netfilter-persistent reload

完成上述操作就可以永久打开我们需要的端口了


### redHat 开放指定端口
同样的，我们开放4412这个端口。先检查iptables是否启动：

        sudo service iptables status

        Redirecting to /bin/systemctl status  iptables.service
        Unit iptables.service could not be found.

安装iptables-services：

        sudo yum install iptables-services

启动iptables：

        sudo service iptables start

编辑配置文件，将4412端口添加到22端口下：

        sudo vi /etc/sysconfig/iptables

        # sample configuration for iptables service
        # you can edit this manually or use system-config-firewall
        # please do not ask us to add additional ports/services to this default configuration
        *filter
        :INPUT ACCEPT [0:0]
        :FORWARD ACCEPT [0:0]
        :OUTPUT ACCEPT [0:0]
        -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
        -A INPUT -p icmp -j ACCEPT
        -A INPUT -i lo -j ACCEPT
        -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
        -A INPUT -m state --state NEW -m tcp -p tcp --dport 4412 -j ACCEPT
        -A INPUT -j REJECT --reject-with icmp-host-prohibited
        -A FORWARD -j REJECT --reject-with icmp-host-prohibited
        COMMIT

重启：

        sudo service iptables restart

查看开放端口：

        sudo iptables -nL


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


### 安装openvpn
``` bash
首先安装openvpn
$ sudo apt install openvpn

然后启动，其中~/Downloads/hewentian是你的公司下发的你自已的登录帐号相关的文件
$ sudo openvpn --config ~/Downloads/hewentian/hewentian.ovpn
```
这样，你在家里也可以连回公司的网络

如果要求帐号密码，又不想每次都输入，则可以建立一个文件，用于存放用户名和密码。第一行存放用户名，第二行存放密码。
``` bash
$ cd ~/Downloads/hewentian/
$ touch auth-user-pass-File
$ vi auth-user-pass-File

userName
password
```

这样连接的命令为
``` bash
$ sudo openvpn --config ~/Downloads/hewentian/hewentian.ovpn --auth-user-pass ~/Downloads/hewentian/auth-user-pass-File
```


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
$ pip install --upgrade pip 或者 $ pip install pip==9.0.1 
$ pip install Pillow
```


### linux拆分文件
linux拆分文件可以使用split命令，既可以按行拆分，又可以按大小拆分。例如有一个文件，它有1000行，如下：
``` bash
$ cat mydata.txt 
1 ----------------
2 ----------------
3 ----------------
4 ----------------
5 ----------------
6 ----------------
7 ----------------
8 ----------------
9 ----------------
10 ----------------
...
...
...
991 ----------------
992 ----------------
993 ----------------
994 ----------------
995 ----------------
996 ----------------
997 ----------------
998 ----------------
999 ----------------
1000 ----------------
```

我们将文件按行拆分，每100行一个文件，后缀有3位数字，以数字递增，后缀为mydata_
``` bash
$ ls
mydata.txt

$ split -l 100 mydata.txt -d -a 3 mydata_
$ ls
mydata_000  mydata_001  mydata_002  mydata_003  mydata_004  mydata_005  mydata_006  mydata_007  mydata_008  mydata_009  mydata.txt

$ wc -l *
  100 mydata_000
  100 mydata_001
  100 mydata_002
  100 mydata_003
  100 mydata_004
  100 mydata_005
  100 mydata_006
  100 mydata_007
  100 mydata_008
  100 mydata_009
 1000 mydata.txt
 2000 total
```

我们将文件按大小拆分，每个文件5k，后缀有3位数字，以数字递增，后缀为mydata_
``` bash
$ ll mydata.txt 
-rw-r--r-- 1 hewentian hewentian 20893 Aug  5 15:28 mydata.txt

$ split -b 5k mydata.txt -d -a 3 mydata_

$ ll mydata*
-rw-r--r-- 1 hewentian hewentian  5120 Aug  5 17:39 mydata_000
-rw-r--r-- 1 hewentian hewentian  5120 Aug  5 17:39 mydata_001
-rw-r--r-- 1 hewentian hewentian  5120 Aug  5 17:39 mydata_002
-rw-r--r-- 1 hewentian hewentian  5120 Aug  5 17:39 mydata_003
-rw-r--r-- 1 hewentian hewentian   413 Aug  5 17:39 mydata_004
-rw-r--r-- 1 hewentian hewentian 20893 Aug  5 15:28 mydata.txt
```

将文件拆分成指定数目的小文件，使用`-n`参数
``` bash
$ split -n 5 mydata.txt -d -a 3 mydata_
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

默认使用22端口，如果是其他端口，请使用-P指定
scp -P 12022 local_file remote_ip:remote_file 

复制目录命令格式：
scp -r local_folder remote_username@remote_ip:remote_folder 
或者 
scp -r local_folder remote_ip:remote_folder 
```

从远程复制到本地
从远程复制到本地，只要将从本地复制到远程的命令的后2个参数调换顺序即可，如下实例
``` bash
$ scp root@192.168.1.100:/home/root/a.txt /home/hewentian/Documents/
$ scp -r root@192.168.1.100:/home/root/a/ /home/hewentian/Documents/
```

### cat, more, less的区别
more功能类似cat，cat命令是整个文件的内容从上到下显示在屏幕上。more会以一页一页的显示方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按b键就会往回（back）一页显示，而且还有搜寻字串的功能 。more命令从前向后读取文件，因此在启动时就加载整个文件。
less与more类似，但使用less可以随意浏览文件，而more仅能向前移动，却不能向后移动，而且less在查看之前不会加载整个文件。

### find 命令将查找到的文件执行操作
这里将当前目录下（包括子目录）的所有以.txt结尾的文件删掉
``` bash
find . -name "*.txt" -exec rm {} \;
```


### Several methods to execute shell scripts
1. . scriptFileName (needn’t excute permission，the script file is under current directory, executing in current shell）
2. sh scriptFileName （ needn’t execute permission，the script file is under current directory , create a child process to execute the script file）
3. ./scriptFileName （ need execute permission，the script file is under current directory ）
4. scriptFileName （ need execute permission， the script file is in directory which listed in PATH） 


### 查看文件编码file命令
``` bash
$ file -i tmp.txt 
tmp.txt: text/plain; charset=utf-8

$ file -i a.txt 
a.txt: text/plain; charset=iso-8859-1
```

### Linux环境下打开来自Windows的文本文件出现乱码
使用`iconv`命令，将目标文件编码方式转为UTF-8，命令如下：
``` bash
$ iconv -f gbk -t utf8 -o outputFile sourceFile

其实，outputFile、sourceFile的名字可以相同，这样就将原文件的编码修改了
```


### sed命令修改文件
``` bash
# 将beforeStr修改为afterStr，保存为新的文件work2.txt
$ sed 's/beforeStr/afterStr/g' work.txt > work2.txt

# 覆盖原来的文件，即在原来的文件修改
$ sed -i 's/beforeStr/afterStr/g' work.txt

# 删除文件第1行
$ sed -i '1d' filename

# 删除第n行
$ sed -i 'nd' filename

# 删除最后一行
$ sed -i '$d' filename

# 删除文件中包含某个关键字的所有行
$ sed -i '/word/d' filename

# 删除文件中包含某个关键字开头的所有行
$ sed -i '/^word/d' filename

# 在文档指定行中增加一行
$ sed -i '/* /a*' filename

示例：
$ more ent.txt 
11111
22222
33333
55555

$ sed -i '/33/a44444' ent.txt 
$ more ent.txt 
11111
22222
33333
44444
55555
```


### sed命令提取文件中指定格式的数据
有文件内容如下：
        2020-05-28 17:49:22,416  [com.hewentian.crawler.DayCrawler] [WARN] - {"date":"2020-01-01","type":1}
        2020-05-28 17:49:27,419  [com.hewentian.crawler.DayCrawler] [WARN] - {"date":"2020-01-01","type":2}

我想提取花括号中的内容，命令如下：
``` bash
$ sed 's/^.*\({.*}\).*/\1/g' dayError.log

{"date":"2020-01-01","type":1}
{"date":"2020-01-01","type":2}
```

### 去除重复的行
可以联合使用sort和uniq命令，uniq只会去除连续重复的行。
``` bash
$ cat ent.txt 
22222
11111
55555
55555
33333
44444
44444
11111
aaaaa

$ uniq ent.txt 
22222
11111
55555
33333
44444
11111
aaaaa

$ sort ent.txt | uniq
11111
22222
33333
44444
55555
aaaaa

或仅显示有重复的行
$ sort ent.txt | uniq -d
11111
44444
55555
```


### 找出两个文件中内容不同的部分并输出
查找在b.txt文件中存在，在a.txt文件中不存在的内容，并输出到文件diff.txt。依次执行如下命令：
``` bash
$ sort -r a.txt -o a.txt
$ sort -r b.txt -o b.txt
$ script diff.txt
$ grep -vFf a.txt b.txt
$ exit
```


### Ubuntu 中卸载软件的几种命令
参考：https://blog.csdn.net/dzjian_/article/details/79768813

1、在终端里通过 apt-get 安装的软件：

	安装软件：sudo apt-get install softname1 softname2 softname3 ……
	卸载软件：sudo apt-get remove softname1 softname2 softname3 ……
	卸载并清除配置：sudo apt-get remove --purge softname1
	更新软件信息数据库：sudo apt-get update
	进行系统升级：sudo apt-get upgrade, sudo apt-get distupgrade
	搜索软件包：sudo apt-cache search softname1 softname2 softname3 ……


2、在终端里通过 deb 安装的软件：

	安装deb软件包：dpkg -i softname.deb
	删除软件包：dpkg -r softname.deb
	删除配置文件：dpkg --purge softname.deb	注意：-r 和 --purge不能同时使用，先删除软件包，再删除配置文件
	查看软件包信息：dpkg -info softname.deb
	查看文件拷贝详情：dpkg -L softname.deb
	查看系统中已安装软件包信息：dpkg -l
	重新配置软件包：dpkg -reconfigure softname


3、卸载源代码编译的的软件：

	cd 源代码目录
	make clean
	./configure
	（make）
	make uninstall
	rm -rf 目录

4、清理系统：

        sudo apt-get autoclean     将已经删除了的软件包的.deb安装文件从硬盘中删除掉
        sudo apt-get clean         会把你已安装的软件包的安装包也删除掉，当然多数情况下这些包没什么用了
        sudo apt-get autoremove    删除为了满足其他软件包的依赖而安装的，但现在不再需要的软件包


### linux系统ssh免密码登录另一台linux机器执行某个脚本
例如，我要在本机通过ssh免密执行一个在IP为`192.168.30.241`的linux机器上的脚本，目标机器的用户为root，SSH端口为：12022

首先，需要将本机的公钥文件`id_rsa.pub`的内容追加到主机`192.168.30.241`上的`~/.ssh/authorized_keys`文件中。如果本机没有该公钥文件，可通过如下命令产生：
``` bash
$ ssh-keygen -t rsa -C "youremail@example.com"
```
上述命令执行后，本机目录~/.ssh下会出现两个文件：id_rsa和id_rsa.pub。其中，id_rsa.pub为公钥文件。

将本机的id_rsa.pub文件传到`192.168.30.241`上：
``` bash
$ scp -P 12022 /home/hewentian/.ssh/id_rsa.pub root@192.168.30.241:/tmp/id_rsa.pub
root@192.168.30.241's password: 
id_rsa.pub                                             100%  399     0.4KB/s   00:00
```

在`192.168.30.241`机器上：
``` bash
$ su root
$ cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys
```

接下来，就可以在本机不输入密码的情况下SSH执行在远程主机`192.168.30.241`中的命令了。命令格式如下：

	直接登录远程的机器：ssh -p 端口号 远程用户名@远程主机名或IP地址
	只执行在远程机器的命令：ssh -p 端口号 远程用户名@远程主机名或IP地址 '远程命令或者脚本'

例如：

    ssh -p 12022 root@192.168.30.241
	ssh -p 12022 root@192.168.30.241 'hostname'
    ssh -p 12022 root@192.168.30.241 '/root/test.sh'
	ssh -p 12022 root@192.168.30.241 'source /etc/profile > /dev/null; cd /root/; sh test.sh'

注意：当远程脚本中使用了一些命令依赖于环境变量时，该脚本需要在其第一行中包含执行profile文件的命令：
``` bash
$ source /etc/profile
或者
$ source ~/.bash_profile
```
否则，远程脚本可能报错。

### FTP工具filezilla
在Ubuntu上面安装`filezilla`还是很方便的，首先检查系统是否已安装此软件:
``` bash
$ filezilla
The program 'filezilla' is currently not installed. You can install it by typing:
sudo apt install filezilla
```
由上可知，系统中并未安装此`filezilla`，我们按提示安装即可：
``` bash
$ sudo apt install filezilla
[sudo] password for hewentian: 
Reading package lists... Done
Building dependency tree
Reading state information... Done
...
...
...
Setting up libwxbase3.0-0v5:amd64 (3.0.2+dfsg-1.3) ...
Setting up libwxgtk3.0-0v5:amd64 (3.0.2+dfsg-1.3) ...
Setting up filezilla (3.15.0.2-1ubuntu1) ...
Processing triggers for libc-bin (2.23-0ubuntu10) ...
```
安装成功后，使用很简单。在命令行中直接输入`filezilla`即可。
![](/img/filezilla.png "")


### ubuntu安装小便签
我们选择安装`indicator-stickynotes`，这个可以在
http://ppa.launchpad.net/umang/indicator-stickynotes/ubuntu/pool/main/i/indicator-stickynotes/
下载，我们下载目前的最新版本`indicator-stickynotes_1.0.0-0_ppa1_all.deb`
安装方法也很简单：
``` bash
$ sudo dpkg -i indicator-stickynotes_1.0.0-0_ppa1_all.deb 
```
启动方式可以直接在命令行中输入`indicator-stickynotes`，当然也可以在资源管理器中查找：
![](/img/indicator-stickynotes.png "")


### ubuntu下面，微信的使用
微信的官网好像没有Linux版本下载，但是可以在以下连接下载到：
https://github.com/geeeeeeeeek/electronic-wechat
下载相应版本后，解压后，即可使用。



### 截屏软件
shutter是值得推荐的一款截图软件，功能丰富，堪称神器，安装方式如下：
``` bash
$ sudo apt-get install shutter
```

在Ubuntu 18.04中安装后，发现编辑按钮变编程灰色。Shutter需要libgoo-canvas-perl库，该库在Ubuntu 18.04主存档中不可用，解决方法：

    下载下面这三个软件，前两个可能系统已经安装。
    https://launchpad.net/ubuntu/+archive/primary/+files/libgoocanvas-common_1.0.0-1_all.deb
	https://launchpad.net/ubuntu/+archive/primary/+files/libgoocanvas3_1.0.0-1_amd64.deb
	https://launchpad.net/ubuntu/+archive/primary/+files/libgoo-canvas-perl_0.06-2ubuntu3_amd64.deb
	
	sudo dpkg -i libgoocanvas-common_1.0.0-1_all.deb 
    sudo dpkg -i libgoocanvas3_1.0.0-1_amd64.deb 
    sudo dpkg -i libgoo-canvas-perl_0.06-2ubuntu3_amd64.deb
    sudo apt --fix-broken install 
    sudo dpkg -i libgoocanvas3_1.0.0-1_amd64.deb 
重启电脑生效。


### 录屏软件
Simple Screen Recorder是一款简单的屏幕录像工具，能够在屏幕上录制视频、教程，界面简单，功能够用。安装的过程可能有点慢，请耐心等待，并多试几次。
``` bash
$ sudo add-apt-repository ppa:maarten-baert/simplescreenrecorder
$ sudo apt-get update
$ sudo apt-get install simplescreenrecorder
```
![](/img/simpleScreenRecorder.png "")


### 系统监视器
它可以实时查看电脑的cpu、内存占用率、温度等等，非常方便，还可以自已定制要显示的内容。
``` bash
$ sudo add-apt-repository ppa:fossfreedom/indicator-sysmonitor
$ sudo apt-get update
$ sudo apt-get install indicator-sysmonitor
```


### 下载软件
ubuntu上面目前没有迅雷可用，但是可以使用uget：
``` bash
$ sudo add-apt-repository ppa:plushuang-tw/uget-stable
$ sudo apt-get update
$ sudo apt-get install uget
```


### 代理软件
推荐使用 shadowsocks：
``` bash
$ sudo add-apt-repository ppa:hzwhuang/ss-qt5
$ sudo apt-get update
$ sudo apt-get install shadowsocks-qt5
```
当然，这仅是工具，你还得有相关帐号。


### Markdown编辑器
ubuntu上面一个好用的markdown编辑器typora：
``` bash
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
$ sudo add-apt-repository 'deb https://typora.io ./linux/'
$ sudo apt-get update
$ sudo apt-get install typora
```


### 剪贴板软件
clipit非常好用，在设置中记得将`Automatically paste selected item`选上，否则选中了不会自动粘贴：
``` bash
$ sudo add-apt-repository ppa:shantzu/clipit
$ sudo apt-get update
$ sudo apt-get install clipit
```


### 图片编辑器
ubuntu上可以使用GIMP，好用:
``` bash
$ sudo apt-get install gimp
```


### 思维导图 Xmind
到 http://www.xmind.net/download/linux/ 下载 xmind-8-update8-linux.zip，解压后即可使用。


### ubuntu18.04怎么设置字体样式，调整字体大小
安装gnome-tweaks桌面配置工具
``` bash
sudo apt install gnome-tweaks
```
alt+f2 在运行窗口输入 gnome-tweaks 命令，然后回车。

### ubuntu 18.04系统设置应用到桌面快捷方式的使用方法
首先在系统文件夹/usr/share/applications中找到对应的desktop文件，将其复制到桌面文件夹即可，如果找不到对应文件则需要按如下步骤生成desktop文件。

1. “Ctrl+Alt+t"打开终端，输入命令：gnome-desktop-item-edit；如果显示不存在，则需要安装命令：sudo apt install gnome-panel

2. 安装完成后执行创建快捷方式：gnome-desktop-item-edit ~/Desktop/ --create-new，弹出对话框后，填入和选择相应的参数及路径。


### ubuntu18.04将最小、最大、关闭按钮放到左边

	gsettings set org.gnome.desktop.wm.preferences button-layout 'minimize,maximize,close:' 


### ubuntu18.04安装nvidia显卡
参考：https://linuxconfig.org/how-to-install-the-nvidia-drivers-on-ubuntu-18-04-bionic-beaver-linux

First, detect the model of your nvidia graphic card and the recommended driver. To do so execute:
``` bash
$ ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:1c.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001D12sv00001D72sd00001703bc03sc02i00
vendor   : NVIDIA Corporation
driver   : nvidia-driver-390 - distro non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```

From the above output we can conclude that the current system has NVIDIA graphic card installed and the recommend driver to install is nvidia-driver-390.

If you agree with the recommendation feel free to use ubuntu-drivers command again to install all recommended drivers:

    $ sudo ubuntu-drivers autoinstall

Alternatively, install desired driver selectively using the apt command. For example:

    $ sudo apt install nvidia-driver-390

Once the installation is concluded, reboot your system and you are done.

### 免密登录远程主机
将本地公钥添加到远程主机的`authorized_keys`中，例如远程主机为 192.168.1.123，然后就可以ssh免密登录过去了
``` bash
$ ssh-copy-id 192.168.1.123

$ ssh 192.168.1.123
```

### Ubuntu 18.04 解决耳机插入没有声音的问题
下载pulseaudio音量控制软件配置驱动即可。
``` bash
$ sudo apt install pavucontrol
```
安装后进入配置选项，将内置音频的声卡安装选为模拟立体声双工即可。
或者进入`Output Devices -> Port`选择`Headphones(unplugged)`这里耳机就会有声音，而电脑自带扬声器没声。


### Ubuntu 18.04禁用关闭笔记本盖子自动待机
``` bash
$ sudo vi /etc/systemd/logind.conf

将其中的：
#HandleLidSwitch=suspend
改成：
HandleLidSwitch=ignore
```
之后重启systemd-logind
``` bash
$ sudo service systemd-logind restart
```
ok.


### 设置日期和时间
在ubuntu下可以通过以下命令设置时间日期，或者使用NTP进行时间同步
``` bash
$ timedatectl set-ntp no
$ timedatectl set-time '2019-01-23 16:01:23'
```


### awk的使用
它可以将文件内容按指定的分隔符进行输出，例如有文件a.txt，其内容如下：

    1###abc
    2###def
    3###ghi

如果我要以`###`为分隔符提取数据，可以这样：
``` bash
$ cat a.txt | awk -F'###' '{print $1,$2}'

1 abc
2 def
3 ghi
```
也可以单独提取某一列。


### Linux下通过一行命令查找并杀掉进程
如果我们运行了一个进程：
``` bash
$ nohup java -jar he-app.jar 2>&1 &
```

通过一条命令杀掉它：
``` bash
$ ps -ef | grep he-app.jar | grep -v grep | awk '{print $2}' | xargs kill -9
```


### 测试网速的工具
可以使用`speedtest-cli`这个工具来测试，如果还未安装，可使用如下命令安装：
``` bash
$ sudo apt install python-pip
$ sudo pip install speedtest-cli
```

使用也好简单：
``` bash
$ speedtest-cli 

Retrieving speedtest.net configuration...
Testing from China Unicom Guangdong province (58.249.3.236)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by ChinaTelecom-GZ (Guangzhou) [2.51 km]: 20.736 ms
Testing download speed................................................................................
Download: 8.25 Mbit/s
Testing upload speed................................................................................................
Upload: 9.03 Mbit/s
```


### 通过shadowsocks访问Google
首先你得在海外有一台机器（假设IP为 111.112.113.114），在该机器上面安装`shadowsocks`服务端：
``` bash
$ yum -y install epel-release
$ yum install -y yum-utils
$ yum-config-manager --enable epel
$ yum install -y python-pip
$ pip install --upgrade pip
$ pip install shadowsocks
 ```

配置服务端:
``` bash
$ vi shadowsocks.json 
{
    "server":"111.112.113.114",
    "server_port":10086,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"myPassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

在后台启动：
``` bash
$ ssserver -c shadowsocks.json -d start
```

如果服务器端有安装docker，也可以通过docker方式，用两条命令完成安装：
``` bash
$ docker pull shadowsocks/shadowsocks-libev

$ docker run -d \
--name ss-server \
-p 4343:4343 \
-p 4343:4343/udp \
-e SERVER_ADDR=0.0.0.0 \
-e SERVER_PORT=4343 \
-e METHOD=aes-256-cfb \
-e PASSWORD="abcd1234*#(" \
-e TIMEOUT=300 \
-e DNS_ADDRS="8.8.8.8,8.8.4.4" \
shadowsocks/shadowsocks-libev
```


然后在客户端（用户本机）安装shadowsocks
``` bash
$ sudo apt install shadowsocks
```

配置shadowsocks要连接的海外主机：
``` bash
$ cd /home/hewentian/ProjectD/
$ touch shadowsocks.json
$ vi shadowsocks.json

{
    "server":"111.112.113.114",
    "server_port":10086,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"myPassword",
    "timeout":300,
    "method":"aes-256-cfb"
}
```

然后，启动本地的服务shadowsocks：
``` bash
$ sslocal -c shadowsocks.json

INFO: loading config from shadowsocks.json
2019-05-10 09:31:46 INFO     loading libcrypto from libcrypto.so.1.1
2019-05-10 09:31:46 INFO     starting local at 127.0.0.1:1080
```

最后，还要在电脑中配置代理，如下图：

![](/img/shadowsocks.png "")

这样，就可以正常访问Google了。


### 修改ubuntu 18.04的sources.list源为阿里
有时候，我们安装软件，会遇到找不到可用的安装包的情况，如下所示：
``` bash
$ cc

Command 'cc' not found, but can be installed with:

sudo apt install gcc
sudo apt install clang
sudo apt install pentium-builder
sudo apt install tcc

$ sudo apt install gcc

E: Failed to fetch http://cn.archive.ubuntu.com/ubuntu/pool/main/g/gcc-8/libmpx2_8.2.0-1ubuntu2~18.04_amd64.deb  404  Not Found [IP: 91.189.91.23 80]
E: Failed to fetch http://cn.archive.ubuntu.com/ubuntu/pool/main/g/gcc-8/libquadmath0_8.2.0-1ubuntu2~18.04_amd64.deb  404  Not Found [IP: 91.189.91.23 80]
E: Failed to fetch http://cn.archive.ubuntu.com/ubuntu/pool/main/g/gcc-7/libgcc-7-dev_7.3.0-27ubuntu1~18.04_amd64.deb  404  Not Found [IP: 91.189.91.23 80]
E: Failed to fetch http://cn.archive.ubuntu.com/ubuntu/pool/main/g/gcc-7/gcc-7_7.3.0-27ubuntu1~18.04_amd64.deb  404  Not Found [IP: 91.189.91.23 80]
E: Failed to fetch http://cn.archive.ubuntu.com/ubuntu/pool/main/g/gcc-defaults/gcc_7.3.0-3ubuntu2.1_amd64.deb  404  Not Found [IP: 91.189.91.23 80]
E: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/l/linux/linux-libc-dev_4.15.0-39.42_amd64.deb  404  Not Found [IP: 91.189.91.23 80]
E: Aborting install.
```

按提示进行安装，却报404。这时候，可以考虑将源修改为国内的，例如阿里的源：
``` bash
$ cd /etc/apt/
$ sudo cp sources.list sources.list_bak
$ sudo vi sources.list

在文件的最后，加上下面的内容：
# https://opsx.alibaba.com/mirror
deb https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse 
deb https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

最后，删除缓存，然后更新一下：
``` bash
$ sudo rm -rf /var/lib/apt/lists/

$ sudo apt-get update
$ sudo apt-get upgrade
```


### redHat升级内核
有时候，系统内核太低了，在安装某些软件后，会提示：

        kernel too old

导致无法使用。这时就要升级系统的内核了。这里以redHat为例。

1. 查看系统当前内核版本：

        uname -r

2. 更新nss：

        sudo yum update nss

3. 安装elrepo的yum源，升级内核需要使用elrepo的yum源，在安装yum源之前还需要我们导入elrepo的key：

        sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
        sudo rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm

4. 升级内核。在yum的elrepo源中有ml和lt两种内核，其中ml(mainline)为最新版本的内核，lt为长期支持的内核。

安装ml内核使用如下命令：

        yum --enablerepo=elrepo-kernel -y install kernel-ml

安装lt内核使用如下命令：

        yum --enablerepo=elrepo-kernel -y install kernel-lt

此处选择lt内核：

        sudo yum --enablerepo=elrepo-kernel -y install kernel-lt

5. 修改grub.conf文件，内核升级完后需要修改内核的启动顺序。

        sudo vi grub.conf
        default=1 改为 default=0

6. 重启系统：

        sudo reboot


### top命令
top命令在执行过程中可以使用的一些交互命令：

        c    切换显示命令名称和完整命令行
        M    根据驻留内存大小进行排序
        P    根据CPU使用百分比大小进行排序
        k    终止一个进程。系统将提示用户输入需要终止的进程PID，以及需要发送给该进程什么样的信号。一般的终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束该进程。默认值是信号15。在安全模式中此命令被屏蔽

top常用命令参数
        top                    每隔5秒显式所有进程的资源占用情况
        top -d 2               每隔2秒显式所有进程的资源占用情况
        top -c                 每隔5秒显式进程的资源占用情况，并显示进程的命令行参数（默认只有进程名）
        top -p 3306 -p 6378    每隔5秒显示pid是3306和pid是6379的两个进程的资源占用情况
        top -d 2 -c -p 6379    每隔2秒显示pid是6379的进程的资源使用情况，并显示该进程启动的命令行参数


### linux创建和删除用户
创建用户
        useradd -m username 创建用户，要加-m参数才会在/home目录下创建用户目录
        passwd username  为useradd创建的用户设置密码
        adduser username 创建用户，自动会在/home目录下创建用户目录，但是它一创建用户，就会要求输入密码

删除用户
若使用userdel username命令删除用户时，并不能删除该用户的所有信息，只是删除了/etc/passwd、/etc/shadow、/etc/group/、/etc/gshadow四个文件里的该账户和组的信息。默认情况下创建一个用户账号，会创建一个home目录和一个用户邮箱（在/var/spool/mail目录以用户名命名）。下次再创建用户时，就会出现：该用户已存在的提示。

正确删除用户
        userdel -r username


### 使用msmtp发送邮件
安装

From source
CentOS repository doesn't have a RPM package for MSMTP so we need to install it from source:

        yum -y install make pkgconfig gcc gcc-c++ gnutls gnutls-devel gnutls-utils openssl openssl-devel libidn libidn-devel
        wget https://marlam.de/msmtp/releases/msmtp-1.8.12.tar.xz
        tar xJvf msmtp-1.8.12.tar.xz
        cd msmtp-1.8.12/
        ./configure --with-ssl=openssl
        make
        make install


On Ubuntu/Debian distribution use apt-get:

        sudo apt-get install msmtp

配置
The configuration file of MSMTP is stored in `~/.msmtprc` for each user and `/etc/msmtprc` is the system wide configuration file. Open the configuration file in your directory.

        vi ~/.msmtprc

``` bash
defaults
tls on
logfile /var/log/msmtp.log

account th
host smtp.exmail.qq.com
port 465
auth on
tls on
tls_starttls off
tls_certcheck off
from test@th.com
user test@th.com
password yourThPassw0rd

account default: th
```

This file can also have more than one account, just ensure that the "account" value is unique for each section. When sending with `-a {accountId}` to specific the account. Save the file and use chmod to make this file readable only by the owner since it contains passwords. This step is mandatory because msmtp won't run if the permissions are more than 600.

        chmod 600 ~/.msmtprc

check from the command-line to ensure it works properly. To do this:

        echo -e "Subject: test msmtp\r\n\r\nThis is a test for th." | /usr/bin/msmtp -d -C ~/.msmtprc -t wentian.he@qq.com
        echo -e "Subject: test msmtp\r\n\r\nThis is a test for th." | /usr/bin/msmtp -d -C ~/.msmtprc -t wentian.he@qq.com,other@qq.com    # 可以同时给多个地址发邮件

if everything is setup correctly, you can copy this file to the/etc directory, but this is option:

        sudo cp ~/.msmtprc /etc/.msmtprc


### 监控系统磁盘、CPU、内存使用量
``` bash
############################################################
#    */10 * * * * /bin/sh /home/hewentian/system_monitor.sh
############################################################

#!/bin/sh

ip=$(ifconfig enp1s0 | grep "inet " | awk -F ' ' '{print $2}')

msg=""

# 获取磁盘使用量，记得修改下面的磁盘编号
disk_total=$(df -h | grep "dev/sda1" | awk -F ' ' '{print $2}')
disk_used=$(df -h | grep "dev/sda1" | awk -F ' ' '{print $3}')
disk_used_percent=$(df -h | grep "dev/sda1" | awk -F '[ %]+' '{print $5}')
echo "disk used: ${disk_used} / ${disk_total}, ${disk_used_percent}%"

# 获取内存使用占比
mem_total=$(free -m | grep Mem | awk -F ' ' '{print $2}')
mem_used=$(free -m | grep Mem | awk -F ' ' '{print $3}')
mem_used_percent=$(awk 'BEGIN{printf "%.0f\n",('$mem_used'/'$mem_total')*100}')
echo "mem used: ${mem_used}M / ${mem_total}M, ${mem_used_percent}%"

# 获取CPU使用率
cpu_idle=$(top -n 1 | grep Cpu | awk '{print $8}' | cut -f 1 -d '.')
echo "cpu idle: ${cpu_idle}"


# 下面进行判断
if [ $disk_used_percent -gt 10 ];then
    msg="disk used: ${disk_used} / ${disk_total}, ${disk_used_percent}%"
fi

if [ $mem_used_percent -gt 30 ];then
    msg=$msg"\nmem used: ${mem_used}M / ${mem_total}M, ${mem_used_percent}%"
fi

if [ $cpu_idle -lt 99 ];then
    msg=$msg"\ncpu idle: ${cpu_idle}%"
fi

echo ""

if [ -n "$msg" ]; then
    msg="ip: ${ip}\n"$msg
    echo "${msg}"
    echo "Subject: system monitor\r\n\r\n${msg}" | /usr/bin/msmtp -d -C ~/.msmtprc -t wentian.he@qq.com
fi
```


### 获取本机的公网IP

        curl ifconfig.me
        curl cip.cc
        curl ipinfo.io
        curl myip.ipip.net


### ubuntu播放rmvb视频问题
如果出现如下问题：
        The playback of this movie requires a realmedia demuxer plugin  which is not installed.

可执行如下命令修复：
        sudo apt install libdvdnav4 libdvd-pkg gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly libdvd-pkg
        sudo apt install ubuntu-restricted-extras
        sudo dpkg-reconfigure libdvd-pkg

或者安装新的播放器：
        sudo apt-get install vlc


### unattended-upgrade进程的CPU占用100%
当用top查看CPU占用太高的进程时，如果看到：

        /usr/bin/python3 /usr/bin/unattended-upgrade --download-only

解决方法：

        sudo vi /etc/apt/apt.conf.d/10periodic
        sudo vi /etc/apt/apt.conf.d/20auto-upgrades

将`APT::Periodic::Unattended-Upgrade "1"`中的`1`改成`0`，重启即可。


### pulseaudio进程的CPU占用40%
当用top查看CPU占用太高的进程时，如果看到：

        /usr/bin/pulseaudio --start --log-target=syslog

解决方法：

        sudo vi /var/lib/gdm/.pulse/client.conf
        sudo chown gdm:gdm /var/lib/gdm/.pulse/client.conf

增加下面两行：
        autospawn = no
        daemon-binary = /bin/true

如果还是占用太高，再执行下面这行：
        mkdir -p ~/.config/speech-dispatcher && echo "DisableAutoSpawn" >> ~/.config/speech-dispatcher/speechd.conf


### ubuntu休眠后无法唤醒黑屏
1. 检查是否安装了`laptop-mode-tools`
        dpkg -l | grep laptop-mode-tools

2. 未安装的话，就先安装
        sudo apt-get install laptop-mode-tools

3. 判断Laptop是否启用了laptop_mode模式
        cat /proc/sys/vm/laptop_mode

如果显示结果为0，则表示未启动，如果为非0的数字则表示启动了

4. 启动laptop_mode
修改配置文件`/etc/default/acpi-support`，更改`ENABLE_LAPTOP_MODE=true`。直接在终端中输入`sudo laptop_mode start`启动了laptop_mode之后，在ubuntu挂起后，基本上就不会遇到无法唤醒的情况了。

如果在`/etc/default/acpi-support`中未找到`ENABLE_LAPTOP_MODE=true`被注释的项，看文件最后一行的提示：
        # Note: to enable "laptop mode" (to spin down your hard drive for longer
        # periods of time), install the laptop-mode-tools package and configure
        # it in /etc/laptop-mode/laptop-mode.conf.

打开`/etc/laptop-mode/laptop-mode.conf`文件，设置以下项

        # Enable laptop mode power saving, when on battery power.
        #
        ENABLE_LAPTOP_MODE_ON_BATTERY=1

        #
        # Enable laptop mode power savings, even when on AC power.
        # This is useful when running as a headless machine, in low power mode
        #
        ENABLE_LAPTOP_MODE_ON_AC=1

        #
        # Enable laptop mode when the laptop's lid is closed, even when we're on AC
        # power? (ACPI-ONLY)
        #
        ENABLE_LAPTOP_MODE_WHEN_LID_CLOSED=1

启动laptop_mode并查看结果
        sudo laptop_mode start
        cat /proc/sys/vm/laptop_mode


### ubuntu设置快速启动图标
        cd ~/.local/share/applications/

以已有的为基础，复制一个
        cp a.desktop b.desktop

然后修改复制出来的文件即可。


