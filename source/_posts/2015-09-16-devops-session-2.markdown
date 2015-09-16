---
layout: post
title: "DevOps Session 2"
date: 2015-09-16 14:24:51 +0800
description: 
comments: true
categories: DevOps

---

本次课主要学习用户权限概念，为了保证系统的安全性，把所有用户赋予最高权限是极不安全的做法。特别是安装 Tomcat 和 Jenkins 时，使用 sudo 用户来安装很容易被黑客攻击。

## 课前准备

课前准备阶段，我们需要完成简单的作业：

1. 使用 tomcat 用户安装 tomcat7 服务器；
2. 使用 jenkins 用户安装 jenkins，jenkins 可以放在tomcat7中运行；
3. 从本机浏览器中访问 jenkins 主页；

<!-- more -->

![Tomcat](http://7xj9js.com1.z0.glb.clouddn.com/tomcat.png =500x)


### 创建用户

我们需要分别创建`jenkins`和`tomcat`用户。出于安全的考虑，这两个用户不应该有管理员权限。创建用户命令如下：

```
sudo adduser jenkins
...

```

安装提示，进一步设置 jenkins 用户的密码，设置完成后，应该可以通过 `su jenkins` 切换到 jenkins 用户验证用户是否创建成功。


### 安装 Java

直接在主用户下面使用 `apt-get`安装 Java7，命令如下：

```
sudo apt-get update
sudo apt-get install openjdk-7-jdk
```

如果想了解更多，请查看文章：[Install Java7 on Ubuntu](http://stackoverflow.com/questions/16263556/installing-java-7-on-ubuntu)


### 安装 Tomcat

apt 可以直接安装 Tomcat，我们这里从源码安装 tomcat7。

首先需要从 [Tomcat](https://tomcat.apache.org/download-70.cgi) 官网获得源码包下载地址：

```
http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.64/bin/apache-tomcat-7.0.64.tar.gz
```

下载后，我们需要解压 tomcat，然后将 tomcat7 放到 `/opt/`目录中。如果在 tomcat 用户下操作，发现对 opt 文件夹没有写权限。所以我们需要在 `/opt` 中添加 tomcat7 文件夹，该文件夹所有者为 `tomcat`。

#### 新建 /opt/tomcat7 目录

登录具有 sudo 权限的用户 `ming`，`sudo mkdir tomcat7`，然后通过 `sudo chown tomcat tomcat7/` 将 tomcat 用户设置为 `/opt/tomcat7`目录的所有者。

```
tomcat@ubuntu:/opt$ su ming
Password:
ming@ubuntu:/opt$ sudo mkdir tomcat7
ming@ubuntu:/opt$ ls -l
total 4
drwxr-xr-x 2 root root 4096 Sep 16 15:51 tomcat7
ming@ubuntu:/opt$ sudo chown tomcat tomcat7/
ming@ubuntu:/opt$ ls -l
total 4
drwxr-xr-x 2 tomcat root 4096 Sep 16 15:51 tomcat7
ming@ubuntu:/opt$

```

文件新建完成后，切换回 tomcat 用户，开始解压 tomcat7，然后移动 tomcat7 至 `/opt` 目录下：

```
tar xvzf apache-tomcat-7.0.64.tar.gz
mv apache-tomcat-7.0.64/* /opt/tomcat7
```

然后，启动 tomcat7

```
/opt/tomcat7/bin/startup.sh

```

访问 `http://192.168.56.101:8080/` 如果成功访问到 Tomcat 主页面，则表示 Tomcat 安装成功。

### Jenkins 在 Tomcat 中运行

如果 Tomcat 已经运行成功，并且我们成功下载 jenkins.war。

![Jenkins](http://7xj9js.com1.z0.glb.clouddn.com/jenkins.png =500x)
