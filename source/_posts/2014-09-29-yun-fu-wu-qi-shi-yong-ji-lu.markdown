---
layout: post
title: "云服务器使用记录"
date: 2014-09-29 15:01:31 +0800
comments: true
categories:
description: 阿里云服务器使用教程

---

###我的第一个云服务器ECS
云服务器简单来说就是一台远程主机，在MAC上可轻松登录进去进行，然后在命令行中可以轻松部署你的网站，应用等等。于是花了100大洋租了几个月的阿里云Aliyun Linux 5.7 主机，出于学习的目的，所有的配置都采用最低配，日后有需要再进行升级吧。  
使用teminal，用SSH登录进去  

```
minggong:octopress minggong$ ssh root@XXX.XX.XXX.XX
root@root@XXX.XX.XXX.XX's password: 

```
`XXX.XX.XXX.XX`是我主机的IP地址，然后根据提示输入密码，然后就顺利的登录进去，刚进去的系统是很干净的。由于系统是centerOS的（试了半天的apt-get都无效），那就使用yum来安装软件吧。

### yum安装java

**BUG1:** 首次尝试运行 `sudo yum update`，但是控制台立即返回`Failed to set locale, defaulting to C`.  
**SOLVE1:** 执行命令 `echo "export LC_ALL=en_US.UTF-8"  >>  /etc/profile`，退出连接重新登录，问
题解决.


**BUG2:** 再次尝试运行 `sudo yum update`，返回错误`No Packages marked for Update`.  
**SOLVE2:** 尝试了N种方法终于在[阿里云主机安装JDK](http://www.itartisan.cn/article/aliyun-redhat-setup-jdk-mysql-nginx-tomcat.html)上找到了解决办法，如下：

**a.通过rpm命令查看有哪些yum包，然后进行卸载**  
`rpm -qa|grep yum`

yum-3.2.22-20.el5  
yum-metadata-parser-1.1.2-3.el5

`rpm -e --nodeps yum-3.2.22-20.el5rpm -e --nodeps yum-metadata-parser-1.1.2-3.el5`

**b.wget从163镜像上下载CentOS的yum包，先 cd /home，把文件下载/home文件夹中，便于管理：**

```
wget http://mirrors.163.com/centos/5/os/x86_64/CentOS/yum-3.2.22-40.el5.centos.noarch.rpm
wget http://mirrors.163.com/centos/5/os/x86_64/CentOS/yum-metadata-parser-1.1.2-4.el5.x86_64.rpm
wget http://mirrors.163.com/centos/5/os/x86_64/CentOS/yum-fastestmirror-1.1.16-21.el5.centos.noarch.rpm

```
下载完成后使用 `rpm -ivh yum*`，这几个文件需要同时安装，所以用一行命令来安装。

c.更新CentOS-Base.repo源

进入`/etc/yum.repos.d/`目录，并备份原始的CentOS-Base.repo文件：  
`cd /etc/yum.repos.d/ &&  mv CentOS-Base.repo CentOS-Base.repo.back `

wget下载Cent-OS.repo文件到`/etc/yum.repos.d/`目录下：  
`wget http://www.linuxidc.com/files/2011/05/06/CentOS-Base.repo`


d.生成缓存文件到/var/cache/yum目录  
`yum makecache`

---

OK，最后终于work了。运行`sudo yum update `，可以看到一些update。  
解决了YUM的安装问题，我们步入正题安装java：

```
[root@iZ23572i0rtZ home]# yum search java | grep -i --color JDK
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.6.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.6.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.7.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.7.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.7.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.7.0-openjdk-src.x86_64 : OpenJDK Source Bundle
ldapjdk.x86_64 : The Mozilla LDAP Java SDK
ldapjdk-javadoc.x86_64 : Javadoc for ldapjdk
[root@iZ23572i0rtZ home]# yum install java-1.7.0-openjdk.x86_64

```

然后按照步骤，一步步确认，最后就安装成功，查看 `java -version` 验证是否正确安装：

```
[root@iZ23572i0rtZ home]# java -version
java version "1.7.0_65"
OpenJDK Runtime Environment (rhel-2.5.1.2.el5_10-x86_64 u65-b17)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
[root@iZ23572i0rtZ home]# 


```




