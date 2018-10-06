---
title: jenkins 学习笔记
date: 2018-10-05 12:02:47
tags: jenkins
categories: other
---

本篇将说说jenkins的使用。

首先，我们要将`jenkins`的安装包下载回来，可以在它的[官网][link_id_jenkins.war]下载最新稳定版：

``` bash
$ cd /home/hewentian/ProjectD/
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war.sha256

验证下载文件的完整性
$ sha256sum -c jenkins.war.sha256 
jenkins.war: OK
```

我们将它安装在当前目录(`/home/hewentian/ProjectD`)下，在当前目录下创建一个jenkins目录，用作`JENKINS_HOME`目录，我们将相关命令放到一个脚本`start_jenkins.sh`中：
``` bash
$ cd /home/hewentian/ProjectD
$ touch start_jenkins.sh
$ vi start_jenkins.sh
```

其中`start_jenkins.sh`脚本的内容如下：
``` bash
#!/bin/sh

JENKINS_HOME=/home/hewentian/ProjectD/jenkins
JENKINS_WAR=/home/hewentian/ProjectD/jenkins.war
LOG_ROOT=$JENKINS_HOME/logs
LOG_FILE=$LOG_ROOT/jenkins.log
WEB_ROOT=$JENKINS_HOME/war

echo "Starting Jenkins ..."
echo "JENKINS_HOME: $JENKINS_HOME"
echo "JENKINS_WAR: $JENKINS_WAR"
echo "LOG_FILE: $LOG_FILE"
echo "WEB_ROOT: $WEB_ROOT"

if [ ! -d $JENKINS_HOME ]; then
    echo "creating: $JENKINS_HOME"
    mkdir $JENKINS_HOME
fi

if [ ! -d $LOG_ROOT ]; then
    echo "creating: $LOG_ROOT"
    mkdir $LOG_ROOT
fi

if [ ! -e $LOG_FILE ]; then
    echo "creating: $LOG_FILE"
    touch $LOG_FILE
fi

if [ ! -d $WEB_ROOT ]; then
    echo "creating: $WEB_ROOT"
    mkdir $WEB_ROOT
fi

java -Xms1024m -Xmx1024m -Djava.awt.headless=true -DJENKINS_HOME=$JENKINS_HOME -jar $JENKINS_WAR --logfile=$LOG_FILE --webroot=$WEB_ROOT --httpPort=8080 --daemon >> $LOG_FILE

tail -f $LOG_FILE
```


启动jenkins：
``` bash
$ chmod +x start_jenkins.sh
$ . start_jenkins.sh
```

如果你看到如下输出：

    Starting Jenkins ...
	JENKINS_HOME: /home/hewentian/ProjectD/jenkins
	JENKINS_WAR: /home/hewentian/ProjectD/jenkins.war
	LOG_FILE: /home/hewentian/ProjectD/jenkins/logs/jenkins.log
	WEB_ROOT: /home/hewentian/ProjectD/jenkins/war
	creating: /home/hewentian/ProjectD/jenkins
	creating: /home/hewentian/ProjectD/jenkins/logs
	creating: /home/hewentian/ProjectD/jenkins/logs/jenkins.log
	creating: /home/hewentian/ProjectD/jenkins/war
	Forking into background to run as a daemon.
	Running from: /home/hewentian/ProjectD/jenkins.war
	Oct 06, 2018 10:48:18 AM org.eclipse.jetty.util.log.Log initialized
	INFO: Logging initialized @780ms to org.eclipse.jetty.util.log.JavaUtilLog
	Oct 06, 2018 10:48:18 AM winstone.Logger logInternal
	.
	.
	. 中间省略部分日志
	.
	Oct 06, 2018 10:48:29 AM jenkins.install.SetupWizard init
	INFO: 
	
	*************************************************************
	*************************************************************
	*************************************************************

	Jenkins initial setup is required. An admin user has been created and a password generated.
	Please use the following password to proceed to installation:

	02b24053bc4844f4a348fdbbbf65c347

	This may also be found at: /home/hewentian/ProjectD/jenkins/secrets/initialAdminPassword

	*************************************************************
	*************************************************************
	*************************************************************

	Oct 06, 2018 10:48:36 AM hudson.model.UpdateSite updateData
	INFO: Obtained the latest update center data file for UpdateSource default
	Oct 06, 2018 10:48:37 AM hudson.model.UpdateSite updateData
	INFO: Obtained the latest update center data file for UpdateSource default
	Oct 06, 2018 10:48:37 AM jenkins.InitReactorRunner$1 onAttained
	INFO: Completed initialization
	Oct 06, 2018 10:48:37 AM hudson.WebAppMain$3 run
	INFO: Jenkins is fully up and running
	Oct 06, 2018 10:48:37 AM hudson.model.DownloadService$Downloadable load
	INFO: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
	Oct 06, 2018 10:48:37 AM hudson.model.AsyncPeriodicWork$1 run
	INFO: Finished Download metadata. 10,126 ms

则证明启动成功，我们按上面的提示打开浏览器，输入：
http://localhost:8080

你将会见到如下界面：
![](/img/jenkins-1.png "jenkins初次启动界面")

将上面日志中的密码输入到上述界面，并点击`[Continue]`按钮，将出现下图界面：
![](/img/jenkins-2.png "jenkins安装插件界面")

为简单起见，选择`Install suggested plugins`安装即可，安装进度如下：
![](/img/jenkins-3.png "jenkins安装插件界面")

接下来是设置admin用户和密码：
Username: hewentian
Password: abc123

![](/img/jenkins-4.png "jenkins设置admin用户")

点击`[Save and Continue]`，并在接下来的界面点击`[Save and Finish]`完成设置。
![](/img/jenkins-5.png "jenkins最终界面")


### 下面进行简单的配置
按下图所示设置JDK、Maven：`[Manage Jenkins]->[Global Tool Configuration]`：
![](/img/jenkins-6.png "设置")
![](/img/jenkins-7.png "设置")

### 下面安装插件
`[Manage Jenkins]->[Manage Plugins]`
安装`Maven Integration`插件，如下图，直接点击`Install wthout restart`，该插件是用于建立maven job
![](/img/jenkins-8.png "安装maven插件")

安装`Deploy to container`插件，用于将构建好的应用部署到容器中：
![](/img/jenkins-9.png "安装Deploy to container插件")


### 下面演示构建项目
首先构建一个从gitHub中拉取原码的项目


未完待续……

[link_id_jenkins.war]: http://mirrors.jenkins.io/war-stable/latest/

