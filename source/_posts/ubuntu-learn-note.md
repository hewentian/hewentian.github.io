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