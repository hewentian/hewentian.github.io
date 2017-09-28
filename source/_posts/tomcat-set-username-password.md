---
title: tomcat 登录用户名 与 密码设置
date: 2017-09-11 10:21:02
tags: tomcat
categories: tomcat
---
``` xml
403 Access Denied

You are not authorized to view this page.

If you have already configured the Host Manager application to allow access and you have used your browsers back button, used a saved book-mark or similar then you may have triggered the cross-site request forgery (CSRF) protection that has been enabled for the HTML interface of the Host Manager application. You will need to reset this protection by returning to the main Host Manager page. Once you return to this page, you will be able to continue using the Host Manager application's HTML interface normally. If you continue to see this access denied message, check that you have the necessary permissions to access this application.

If you have not changed any configuration files, please examine the file conf/tomcat-users.xml in your installation. That file must contain the credentials to let you use this webapp.

For example, to add the admin-gui role to a user named tomcat with a password of s3cret, add the following to the config file listed above.

<role rolename="admin-gui"/>
<user username="tomcat" password="s3cret" roles="admin-gui"/>

Note that for Tomcat 7 onwards, the roles required to use the host manager application were changed from the single admin role to the following two roles. You will need to assign the role(s) required for the functionality you wish to access.

admin-gui - allows access to the HTML GUI
admin-script - allows access to the text interface
The HTML interface is protected against CSRF but the text interface is not. To maintain the CSRF protection:

Users with the admin-gui role should not be granted the admin-script role.
If the text interface is accessed through a browser (e.g. for testing since this interface is intended for tools not humans) then the browser must be closed afterwards to terminate the session.
```
Tomcat Manager是Tomcat自带的、用于对Tomcat自身以及部署在Tomcat上的应用进行管理的web应用。
在默认情况下，Tomcat Manager是处于禁用状态的。准确地说，Tomcat Manager需要以用户角色进行
登录并授权才能使用相应的功能，不过Tomcat并没有配置任何默认的用户，因此需要我们进行相应的
用户配置之后才能使用Tomcat Manager。

在使用Tomcat时，我们通过点击 http://localhost:8080/ 下面的【Host Manager】可以管理Tomcat服务器中的web工程；

设置用户名、有密码的方法：

1、登录用户名和密码我们需要在 目录：${TOMCAT_HOME}/conf/tomcat-users.xml 中修改如下配置：
``` xml
<role rolename="admin-gui"/>
<user username="tomcat" password="s3cret" roles="admin-gui"/>
```

2、进入当前目录下的：context.xml 文件，并作如下修改（其实这一步，可以省略的）
``` xml
<?xml version="1.0" encoding="UTF-8"?>  
    <WatchedResource>WEB-INF/web.xml</WatchedResource>  
    <Manager pathname="/manager" debug="0"  privileged="true" docBase="${TOMCAT_HOME}/webapps/manager" />      
    <Valve className="org.apache.catalina.valves.CometConnectionManagerValve" />    
</Context> 
```

完成这两步之后，重启tomcat ，输入刚才设置的用户名和密码，即可登录Tomcat 管理web project

