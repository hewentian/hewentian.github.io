---
title: docker 学习笔记
date: 2019-07-26 10:32:30
tags: docker
categories: container
---

今天开始学习一下docker，因为它越来越流行了。首先，我们来安装一下。因为我的系统为`Ubuntu Linux`，所以我的安装过程，参照这里：
https://docs.docker.com/install/linux/docker-ce/ubuntu/

安装过程如下，在系统上首次安装的时候需要执行**安装前的准备工作**：

### 安装前的准备工作
第一步：首先卸载系统自带的旧版docker（如系统有预安装docker）
``` bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

第二步：更新`apt`包索引：
``` bash
$ sudo apt-get update
```

第三步：安装允许`apt`通过HTTPS访问仓库的软件包：
``` bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

第四步： 添加docker官方的GPG key：
``` bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

验证添加KEY的指纹是否是`9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`，我们只需搜索最后8个字符即可：
``` bash
$ sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

第五步：将docker的下载路径添加到下载源中：
``` bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```


### 开始正式安装
第一步：更新`apt`包索引：
``` bash
$ sudo apt-get update
```

第二步：开始安装docker最新的社区版：
``` bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

安装结束后，docker的daemon进程默认自动启动，否则需要手动启动。
``` bash
$ sudo service docker start
```

第三步：验证安装是否正确无误，通过运行一个测试用例：
``` bash
$ sudo docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:6540fc08ee6e6b7b63468dc3317e3303aae178cb8a45ed3123180328bcc1d20f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

至此，安装完成。

### 卸载方法
要卸载软件本身和删除相关的docker文件
``` bash
$ sudo apt-get purge docker-ce
$ sudo rm -rf /var/lib/docker
```


未完待续……

