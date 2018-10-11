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
#### 示例A、构建一个从gitHub中拉取原码的maven项目
`[New Item]`->选择`[Maven project]`，并在`[Enter an item name]`中输入mvn-test，然后点击`[ok]`，如下图：
![](/img/jenkins-10.png "构建一个从gitHub中拉取原码的maven项目")

在弹出的界面中选中`[Discard old builds]`并将`Max of builds to keep`设为10，然后设置源码仓库，如下所示：
![](/img/jenkins-11.png "")

** 注意： ** 如果我们的仓库中包含有多个项目，而我们此处要构建的只是其中一个，则我们需要指定构建哪一个：`Additional Behaviours -> Add -> Sparse Checkout paths`，在`Path`处填入: `/{repository_name}/{need_to_build_project}/**`

例如：
Repository URL: `https://github.com/jenkins-docs/simple-java-maven-app.git`
Path: `/simple-java-maven-app/my-app/**`
如果是这种方式，则下面的Root POM也要修改成对应的项目:
Root POM: `simple-java-maven-app/my-app/pom.xml`
上面的Path开头是有`/`的，而Root POM开头是没有`/`的。

设置Build的`Goals and options`为`clean install`，如下：
![](/img/jenkins-12.png "")

其他设置保持默认，点击`[Save]`，在弹出的界面点击`[Build Now]`，然后再点击下方构建历史中正在构建的任务的`[Console Output]`。
![](/img/jenkins-13.png "")

![](/img/jenkins-14.png "")
![](/img/jenkins-15.png "")

如上图所示，构建成功了。切换到上图中的目录中查看目标文件，并运行它：
![](/img/jenkins-16.png "")

这样，一个简单的maven项目就构建完成了。

#### 示例B、构建一个从gitHub中拉取原码的maven web项目，并部署到运行中的tomcat
首先，我们创建一个最简单的maven web项目，并推到：
https://github.com/hewentian/web-test
web-test项目只有三个文件：

	pom.xml
	src/main/webapp/WEB-INF/web.xml
	src/main/webapp/index.jsp

`pom.xml`文件内容如下：
``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hewentian</groupId>
	<artifactId>web-test</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>web-test Maven Webapp</name>
	<url>http://maven.apache.org</url>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<finalName>web-test</finalName>
	</build>
</project>
```

`src/main/webapp/WEB-INF/web.xml`文件内容如下：
``` xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
	<display-name>Archetype Created Web Application</display-name>
</web-app>
```

`src/main/webapp/index.jsp`文件内容如下：
``` jsp
<html>
	<body>
		<h2>Hello World!</h2>
	</body>
</html>
```

接着，我们准备一台tomcat，我已经准备好了一台，位于：

	/home/hewentian/ProjectD/apache-tomcat-8.0.47

因为jenkins使用了`8080`端口，所以tomcat不能使用默认的`8080`端口，我们将其修改为`8867`：
``` bash
$ cd /home/hewentian/ProjectD/apache-tomcat-8.0.47/conf
$ vi server.xml

只修改此处即可
<Connector port="8867" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```

配置tomcat的管理员帐号：
``` bash
$ cd /home/hewentian/ProjectD/apache-tomcat-8.0.47/conf
$ vi tomcat-users.xml

在<tomcat-users>节点里添加如下内容：

<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>

<user username="hwt" password="pwd123" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```
其中的`username="hwt" password="pwd123"`是用于登录Tomcat用的，下面会用到，重启tomcat。

回到jenkins，我们新建一个Item，命名为web-app-test：
![](/img/jenkins-17.png "")
![](/img/jenkins-18.png "")

配置代码仓库，如下图。点击`Credentials`右边的`Add->jenkins`
![](/img/jenkins-19.png "")

在弹出的对话框中，选择`SSH Username with private key`，将`~/.ssh/id_rsa`文件的内容复制到Key中，点`Add`：
![](/img/jenkins-20.png "")

在配置代码仓库中，选择刚才创建的`Credentials`：
![](/img/jenkins-21.png "")

配置构建触发器：
![](/img/jenkins-22.png "")

说明：

1. Build whenever a SNAPSHOT dependency is built：在构建的时候，会根据pom.xml文件的继承关系构建发生一个构建引起其他构建的；
2. Poll SCM：这是CI系统中常见的选项。当您选择此选项，您可以指定一个定时作业表达式来定义Jenkins每隔多久检查一下您源代码仓库的变化。如果发现变化，就执行一次构建。例如，表达式中填写0,15,30,45 * * * *将使Jenkins每隔15分钟就检查一次您源码仓库的变化；
3. Build periodically：此选项仅仅通知Jenkins按指定的频率对项目进行构建，而不管SCM是否有变化。如果想在这个Job中运行一些测试用例的话，它就很有帮助。

配置构建设置：
![](/img/jenkins-23.png "")

接着我们试着点击`Build Now`试下能否成功构建：
![](/img/jenkins-24.png "")

当你看到如下输出时，证明构建成功：
![](/img/jenkins-25.png "")

接着我们配置部署到tomcat，回到web-app-test的jenkins配置，在`Add post-build action`中选择`Deploy war/ear to a container`，如下图：
![](/img/jenkins-26.png "")

在`Credentials`右则点击`Add->Jenkins`，并在弹出的对话框中输入上面在tomcat中配置的用户名：
![](/img/jenkins-27.png "")

说明：

1. 首先tomcat是启动的，并且Tomcat中没有部署web-test.war；
2. WAR/EAR files：war文件的存放位置，如：target/web-test.war 注意：相对路径，target前是没有/的；
3. Context path：访问时需要输入的内容，如wt访问时如下：[http://127.0.0.1:8867/wt/](http://127.0.0.1:8867/wt/)，如果为空，默认是war包的名字；
4. Container：选择你的web容器，如tomca 8.x；
5. Credentials: 在右边的下拉页面中选择访问Tomcat的用户名、密码，如果没有，则点【Add】；
6. Tomcat URL：填入你Tomcat的访问地址，如：http://127.0.0.1:8867/；
7. svn、git、tomcat的用户名和密码设置了是没有办法在web界面修改的。如果要修改则先去Jenkins目录删除hudson.scm.SubversionSCM.xml文件，或者在jenkins用户页中删掉该用户，虽然jenkins页面提供修改方法，但是，无效。

接着我们点击`Build Now`开始构建：
![](/img/jenkins-28.png "")

如果你看到上面输出，则证明构建和部署成功，可以打开浏览器查看：
![](/img/jenkins-29.png "")

到此，大功告成。


#### 示例C、将示例A产生的JAR包部署到远程机器上面运行
我们回到示例A的jenkins配置，在Post Steps下选择`Run only if build succeeds`，点`Add post-build step`并选择`Execute shell`，在Command中填入如下脚本，此脚本由我同事`严忠思`编写，我稍作修改：
``` bash
# 1.定义变量
# SSH 端口
export SSH_PORT="12022"

# 运行 jar 包的机器,多个IP以空格分隔，如: 192.168.30.241 192.168.30.242
export SSH_IP_LIST="192.168.30.241"

# 运行 jar 的用户
export USERNAME="root"

# 环境 dev,test,gray,prod
export RUN_SERVER="dev"

# 远程存放 jar 包文件路径,注这个路径要先手动创建, mkdir -p /www/web/my-app && chown root.root /www/web/my-app
export REMOTE_JAR_DIR="/www/web/my-app/${RUN_SERVER}"

# Jenkins (-DJENKINS_HOME)用 maven 编译打包程序的路径与文件
export JENKINS_JAR_FILE="/home/hewentian/ProjectD/jenkins/workspace/mvn-test/target/my-app-1.0-SNAPSHOT.jar"

#jar 打包文件名
export JAR_FILE="my-app-1.0-SNAPSHOT.jar"

# 运行 JAR 的端口，我这里并不使用这个端口号，故可不填
export JAR_PORT="8802"

# 日志路径
export LOG_PATH="/www/logs/my-app"

#jvm参数
export JAR_JAVA_OPTS="-XX:-UseGCOverheadLimit -Xmx1024m"

# jar 运行命令
export JAR_COMMOND="nohup java ${JAR_JAVA_OPTS} \
-jar ${REMOTE_JAR_DIR}/${JAR_FILE} \
--spring.cloud.config.profile=${RUN_SERVER} \
--server.port=${JAR_PORT} \
--logging.path=${LOG_PATH} \
> ${LOG_PATH}/my-app.log &"

# 等待时间，如果不配置，则脚本默认为 40 秒
export SLEEP_SEC=20

# 2.主程序
/bin/bash -x /home/hewentian/ProjectD/jenkins/script/jar.sh
```
在我的本机执行如下命令，创建存放脚本的目录：
``` bash
$ cd /home/hewentian/ProjectD/jenkins
$ mkdir script
$ cd script
$ touch jar.sh
$ chmod +x jar.sh
```

在jar.sh中输入如下脚本，此脚本同样由我的同事`严忠思`编写：
``` bash
#!/bin/env sh

source /etc/profile > /dev/null 2>&1

echo "-------------------- start print env var --------------------"
echo "SSH_PORT: $SSH_PORT"
echo "SSH_IP_LIST: $SSH_IP_LIST"
echo "USERNAME: $USERNAME"
echo "RUN_SERVER: $RUN_SERVER"
echo "REMOTE_JAR_DIR: $REMOTE_JAR_DIR"
echo "JENKINS_JAR_FILE: $JENKINS_JAR_FILE"
echo "JAR_COMMOND: $JAR_COMMOND"
echo "-------------------- end print env var --------------------"

#### IP 数量
IP_LIST=1
HOST_COUNT=$(echo ${SSH_IP_LIST} | wc -w)

#### 默认定义时间为40秒
if [ "${SLEEP_SEC}" == "" ];then
    SLEEP_SEC=40
fi

#### 检查是否添加公钥
function SSH_CHECK(){
    ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "uname -n"
    if [ "$?" -ne 0 ];then
        echo -e "Jenkins 登录失败, ${SSH_HOST} 没有添加 SSH 公钥，请把 Jenkins 公钥添加到 ${SSH_HOST} \n"
        echo "或检查 ${SSH_HOST}  ~/.ssh 目录与 ~/.ssh/authorized_keys 文件权限(chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys)"
        exit 1
    fi
}

#### 构建目录，如果失败可以验证客户端没权限
function RUN_DIR(){  
    ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "uname -n;/bin/mkdir -p ${REMOTE_JAR_DIR}"
    RUN_ID=`echo $?`
    if [ "${RUN_ID}" -ne 0 ];then
        echo "${SSH_HOST} 的 ${USERNAME} 用户创建目录失败，请检查 ${USERNAME} 用户是否有权限"
        exit 1
    fi
}

#### 存放日志目录
function LOG_PATH_DIR(){  
    ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "uname -n;/bin/mkdir -p ${LOG_PATH}"
    RUN_ID=`echo $?`
    if [ "${RUN_ID}" -ne 0 ];then
        echo "${SSH_HOST} 的 ${USERNAME} 用户创建目录失败，请检查 ${USERNAME} 用户是否有权限"
        exit 1
    fi
}

function RSYNC_JAR(){  
    rsync -azP --delete -e "ssh -p $SSH_PORT -o 'StrictHostKeyChecking=no'" ${JENKINS_JAR_FILE} ${USERNAME}@${SSH_HOST}:${REMOTE_JAR_DIR} > /dev/null
}

function JAR_PID(){  
    PID=$(ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "/usr/sbin/lsof -i:${JAR_PORT} | grep -vi PID | awk '{print \$2}'")
    echo $PID
}

function STOP_JAR(){  
    if [ -n "${PID}" ] || [ -n "${PID2}" ];then    
        ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "kill -9 $PID > /dev/null 2>&1"
        echo "INFO: ${JAR_FILE} 进程已杀"
    else    
        echo "INFO: ${JAR_FILE} is Down"
    fi
    PID=""
}

function START_JAR(){  
    ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "source /etc/profile > /dev/null; cd ${REMOTE_JAR_DIR}; ${JAR_COMMOND} "
}

function CHECK_JAR(){ 
    PID=$(ssh -p ${SSH_PORT} -o "StrictHostKeyChecking=no" ${USERNAME}@${SSH_HOST} "/usr/sbin/lsof -i:${JAR_PORT} | grep -vi PID | awk '{print \$2}'")
    if [ "$PID" != "" ];then
        echo "${SSH_HOST} 的 ${JAR_FILE} 启动成功" 
    else
        echo "${SSH_HOST} 的 ${JAR_FILE} 启动失败,请运维登录服务器查看进程或相关启动日志" 
        exit 1
    fi
}

for SSH_HOST in $SSH_IP_LIST
do 
    SSH_CHECK 
    RUN_DIR    
    LOG_PATH_DIR
    RSYNC_JAR 
    JAR_PID 
    #STOP_JAR
    START_JAR
    
    if [ "${IP_LIST}" -le "${HOST_COUNT}" ];then
        echo "正在检测 ${SSH_HOST} 的 ${JAR_FILE} 程序是否成功启动，请等待 ${SLEEP_SEC} 秒!"
        sleep ${SLEEP_SEC}
        #CHECK_JAR
    fi
    if [ "${IP_LIST}" -gt "${HOST_COUNT}" ];then
        echo "INFO: <${IP_LIST}> 更新下一台机..."
    fi
    let IP_LIST=IP_LIST+1
done
```
回到jenkins去点击`[Build Now]`，在`[Console Output]`观看它的构建情况。等构建成功后，我们登录`192.168.30.241`查看情况：
``` bash
$ cd /home/hewentian/
$ ssh -p 12022 root@192.168.30.241

Last login: Thu Oct 11 11:18:50 2018 from 10.1.23.231
[root@192.168.30.241 ~]# ls /www/
logs  web
[root@192.168.30.241 ~]# ls /www/web/my-app/dev/
my-app-1.0-SNAPSHOT.jar
[root@192.168.30.241 ~]# more /www/logs/my-app/my-app.log 
Hello World!
```
从上述输出可知，我们的构建已经成功！！！

**将脚本存放在`jar.sh`中的好处是此脚本可以供多个项目共同使用，只要在`Execute shell`中根据不同项目定义不同的变量值即可。**


#### 示例D、当构建出错的时候，如何回滚(rollback)到上一个版本
要实现这个功能，我们要在jenkins安装一个插件`Copy Artifact`：
![](/img/jenkins-30.png "")



未完待续……

[link_id_jenkins.war]: http://mirrors.jenkins.io/war-stable/latest/

