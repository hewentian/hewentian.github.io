---
title: nginx 学习笔记
date: 2019-06-10 19:20:16
tags: nginx
categories: web server
---

今天一看，天哪，原来已经有4个月没有更新博客了，我在想，这4个月我都干嘛去了，内心立马慌起来了（你知道的，程序员是不能停止学习的）。但是细想，虽然没更新博客，但还是看了几本书：《从0到1》、《毛泽东选集》卷一、《MongoDB in Action》、《SpringBoot in Action》、《白帽子讲Web安全》，内心立马淡定了不少。好了，题外话不多说了，马上进入主题。


`nginx`是一个`HTTP`和反向代理服务器、邮件代理服务器和通用的`TCP/UDP`代理服务器，最初由俄罗斯程序员`Igor Sysoev`所开发。

详细介绍可以参见：
http://nginx.org/en/
http://nginx.org/en/docs/


本文将说下`nginx`的简单安装使用，因为在项目中要使用到的。我在虚拟机VirtualBox中的机器中安装，系统为`Ubuntu Linux`，机器节点相关配置如下（之前安装过Hadoop集群的机器）：

    master：
        ip: 192.168.56.110
        hostname: hadoop-host-master

首先，我们要将`nginx`的安装包下载回来，截止本文写时，它的最新稳定版本为`1.16.0`，可以在它的[官网][link_id_nginx-1.16.0.tar.gz]下载。我先在我的物理机器下载回来。

``` bash
$ cd /home/hewentian/Downloads/
$ wget http://nginx.org/download/nginx-1.16.0.tar.gz
$ wget http://nginx.org/download/nginx-1.16.0.tar.gz.asc

验证下载文件的完整性，这里略
```

在物理机上将它传到要安装的机器`hadoop-host-master`：
``` bash
$ scp nginx-1.16.0.tar.gz hadoop@hadoop-host-master:~/
```

接下来，我们进入`hadoop-host-master`中操作：
``` bash
$ ssh hadoop@hadoop-host-master

$ tar xf nginx-1.16.0.tar.gz
$ cd nginx-1.16.0/
$ ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  man  README  src


$ sudo ./configure 
checking for OS
 + Linux 4.15.0-39-generic x86_64
...

中间省略部分


Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

在`./configure`过程中会检查依赖的缺失情况，如果有缺失，则会在这里提示。我们根据提示安装即可。

一般来说，nginx编译会依赖：zlib、zlib-devel、openssl、openssl-devel、pcre、pcre-devel、gcc、g++

在CentOS上可以通过以下方式进行安装：
``` bash
$ yum -y install zlib zlib-devel openssl openssl-devel pcre pcre-devel
```

而在Ubuntu上面则可以通过下面的方式进行安装：
``` bash
$ sudo apt-get install build-essential    这会同时安装 gcc、g++
$ sudo apt-get install zlib1g
$ sudo apt-get install openssl libssl-dev
$ sudo apt-get install libpcre3 libpcre3-dev
$ sudo apt-get install zlib1g-dev
```

接下来进行编译安装：
``` bash
$ sudo make && make install

objs/ngx_modules.o \
-ldl -lpthread -lcrypt -lpcre -lz \
-Wl,-E
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
	-e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
	-e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
	-e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
	< man/nginx.8 > objs/nginx.8
make[1]: Leaving directory '/home/hadoop/nginx-1.16.0'
```
它默认会安装到`/usr/local/nginx/`目录下。

启动命令如下：
``` bash
$ sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

修改配置文件后，检查配置文件是否正确的命令如下：
``` bash
$ sudo /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf

nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

可以打开 http://192.168.56.110/ 来看看是否已启动。

![](/img/nginx-1.png "nginx 初始启动界面")

另外，使用 http://hadoop-host-master/ 也是一样的。

![](/img/nginx-2.png "nginx 初始启动界面")

当你看到上面的界面后，nginx的安装到这里就成功了。


接下来，我们作一些简单的配置示例。但是在开始之前，在执行访问的机器上面（在这里是我的物理机器），配置一下HOST，**增加**下面这3行：
``` bash
$ sudo vi /etc/hosts

192.168.56.110 www.hewentian.com
192.168.56.110 img.hewentian.com
192.168.56.110 api.hewentian.com
```

### 示例一：将某目录下的图片，让其他机器可以通过WEB访问
在安装了nginx的机器`hadoop-host-master`上有个目录`/home/hadoop/Pictures`，这个目录下放有图片（我的机器里有一张：my_computer.png）。现在想通过web，让其他机器的用户访问这个目录下的图片。

修改`nginx.conf`配置，添加如下代码即可：
``` bash
$ sudo vi /usr/local/nginx/conf/nginx.conf

server {
        listen       80;
        server_name  img.hewentian.com;                 # 修改 1/3： WEB访问的位置
        access_log  logs/img.access.log  main;          # 修改 2/3： 日志存放位置。记得将此配置文件中的 log_format  main 前面的注释打开

        location / {
            root   /home/hadoop/Pictures/;              # 修改 3/3： 图片存放位置
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

保存文件后，退出。
检查配置文件是否正确：
``` bash
$ sudo /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
```

然后重启nginx，首先找出nginx进程号：

    ps -ef | grep nginx

然后杀死nginx的主进程：

    sudo kill -9 [nginx进程号]

重启nginx

    sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

在物理机器上面打开浏览器，试着访问： http://img.hewentian.com/my_computer.png

![](/img/nginx-3.png "nginx 作为图片服务器")

查看nginx访问日志：
``` bash
$ tail /usr/local/nginx/logs/img.access.log

192.168.56.1 - - [11/Jun/2019:22:46:33 +0800] "GET / HTTP/1.1" 403 555 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [11/Jun/2019:22:46:50 +0800] "GET /my_computer.png HTTP/1.1" 200 59332 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [11/Jun/2019:22:47:21 +0800] "GET /my_computer.png HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:00:51:04 +0800] "GET /my_computer.png HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:00:51:04 +0800] "GET /my_computer.png HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:00:51:04 +0800] "GET /my_computer.png HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:00:51:05 +0800] "GET /my_computer.png HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:01:02:06 +0800] "GET /my_computer.png HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
```

### 示例二：实现简单的负载匀衡
有个应用，它有个接口`/hello`，是返回当前服务器的IP地址。我是使用SpringBoot来简单开发的，代码只有几行：

``` java
package com.hewentian.web.controller;

import org.apache.log4j.Logger;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * <p>
 * <b>HelloController</b> 是 返回当前服务器IP地址的Controller
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date 2019-06-12 14:52:49
 * @since JDK 1.8
 */
@RestController
public class HelloController {
     private static Logger log = Logger.getLogger(HelloController.class);

    @RequestMapping("/hello")
    public String index() {
        String address = "";

        try {
            InetAddress inetAddress = InetAddress.getLocalHost();
            address = inetAddress.getHostAddress();
        } catch (UnknownHostException e) {
            log.error(e.getMessage(), e);
        }

        return address + " is serving for you: Hello World";
    }
}
```

部署在下面的2台服务器上，都是监听8080端口。

    slave1:
        ip: 192.168.56.111
        hostname: hadoop-host-slave-1
    slave2:
        ip: 192.168.56.112
        hostname: hadoop-host-slave-2

现在想在网页上访问：http://192.168.56.110 或 http://api.hewentian.com 的时候，就会访问到上面这2台机器，实现负载均衡。

修改`nginx.conf`配置，添加如下代码即可：
``` bash
$ sudo vi /usr/local/nginx/conf/nginx.conf

upstream api_worker {
    server 192.168.56.111:8080 weight=3000;
    server 192.168.56.112:8080 weight=3000;
    keepalive 2000;
}

server {
    listen      80;
    server_name 192.168.56.110  api.hewentian.com;
    access_log  logs/api.access.log  main;

    location / {
        proxy_pass http://api_worker/;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

重启nginx。正常情况下，流量会匀衡地分到2台服务器，如下图所示。
![](/img/nginx-4.png "nginx 作为负载匀衡服务器")
![](/img/nginx-5.png "nginx 作为负载匀衡服务器")

如果两台机器中的某一台挂掉了，流量会自动分到另外一台，这样也实现了简单的高可用。

查看nginx访问日志：
``` bash
$ tail /usr/local/nginx/logs/api.access.log 

192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /hello HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /favicon.ico HTTP/1.1" 200 946 "http://api.hewentian.com/hello" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /hello HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /favicon.ico HTTP/1.1" 200 946 "http://api.hewentian.com/hello" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /hello HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /favicon.ico HTTP/1.1" 200 946 "http://api.hewentian.com/hello" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /hello HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /favicon.ico HTTP/1.1" 200 946 "http://api.hewentian.com/hello" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /hello HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
192.168.56.1 - - [12/Jun/2019:02:46:52 +0800] "GET /favicon.ico HTTP/1.1" 200 946 "http://api.hewentian.com/hello" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
```


### 示例三：实现同一个域名下的两个服务
有两个服务：
一个是一个WEB服务，是指向另一台机器的，其中`/hello`是一个接口：
http://www.hewentian.com/api/hello

另一个是用来看某个目录下的图片的：
http://www.hewentian.com/img/my_computer.png

修改`nginx.conf`配置，添加如下代码即可：
``` bash
$ sudo vi /usr/local/nginx/conf/nginx.conf

upstream api_worker {
    server 192.168.56.111:8080 weight=3000;
    keepalive 2000;
}

server {
    listen      80;
    server_name www.hewentian.com;
  
    location / {
        root   html;
        index  index.html;
    }
  
    location /api/ {
        proxy_pass http://api_worker/;
        #proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_http_version 1.1;
        #proxy_set_header Connection "";
        access_log  logs/api.access.log  main;
    }

    location /img/ {
        alias /home/hadoop/Pictures/;
        access_log  logs/img.access.log  main;
    }
}
```

重启nginx。访问上述地址，将得到如下图效果：

![](/img/nginx-6.png "nginx 实现同一个域名下的两个服务")

![](/img/nginx-7.png "nginx 实现同一个域名下的两个服务")

查看nginx访问日志：
``` bash
$ tail -n1 /usr/local/nginx/logs/api.access.log /usr/local/nginx/logs/img.access.log 

==> /usr/local/nginx/logs/api.access.log <==
192.168.56.1 - - [12/Jun/2019:04:09:55 +0800] "GET /api/hello HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"

==> /usr/local/nginx/logs/img.access.log <==
192.168.56.1 - - [12/Jun/2019:04:08:04 +0800] "GET /img/my_computer.png HTTP/1.1" 200 59332 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36" "-"
```



[link_id_nginx-1.16.0.tar.gz]: http://nginx.org/download/nginx-1.16.0.tar.gz


