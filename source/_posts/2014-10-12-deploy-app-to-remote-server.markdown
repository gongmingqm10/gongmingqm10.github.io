---
layout: post
title: "Deploy app to remote server —— 网站部署篇"
date: 2014-10-12 11:14:37 +0800
comments: true
categories: Web
description: Java war远程部署 Tomcat

---

网站开发完毕之后，下一步的任务就是部署了。我们从最简单的入手，如何将本地开发的webapp部署到远程服务器上（这里我们用的是阿里的ECS）。部署是一件很有意思的事情，因为我们可以让本地开发的网站被外界所访问，所以还是很多成就感的。这里的工程采用的是Spring MVC 框架。

###1. Requirement

* 一个能够在本地运行的Web程序  
* 一个远程服务器主机
* Mac或者Linux系统的机器

<!-- more -->

###2. 打包Webapp为war文件

不同的工程可以使用不同的工具进行打包，我们项目工程采用gradle + jetty构建的，因此在打包时还是使用gradle自带的方式进行打包。本项目build.gradle配置如下:  

```
apply plugin: 'idea'
apply plugin: 'war'
apply plugin: 'jetty'


tasks.withType(Compile) {
    options.encoding = "UTF-8"
}
repositories {
    mavenCentral()
}

dependencies {
    providedCompile 'javax.servlet:servlet-api:2.5'

    def springFrameworkVersion = '3.2.2.RELEASE'

    compile "org.springframework:spring-web:${springFrameworkVersion}",
            "org.springframework:spring-webmvc:${springFrameworkVersion}",
            "org.springframework:spring-beans:${springFrameworkVersion}",
            "org.springframework:spring-context:${springFrameworkVersion}",
            "org.springframework:spring-jdbc:${springFrameworkVersion}"



    compile "aopalliance:aopalliance:1.0"
    compile "org.codehaus.jackson:jackson-mapper-asl:1.9.+"
    compile "org.mybatis:mybatis:3.2.3"
    compile 'org.apache.commons:commons-lang3:3.1'
    compile 'commons-fileupload:commons-fileupload:1.2.2'
    compile 'commons-io:commons-io:1.3.2'

    runtime "commons-pool:commons-pool:1.3",
            "commons-dbcp:commons-dbcp:1.2.2",
            "commons-collections:commons-collections:3.2",
            'javax.servlet:jstl:1.2',
            "org.slf4j:slf4j-log4j12:1.6.0",
            "mysql:mysql-connector-java:5.1.5",
            'log4j:log4j:1.2.16'
}

```
根目录下直接使用 `gradle war`就可以快速打包完成。打包完成后我们在 `build/libs/` 目录下可以看到工程的war包, demo.war

###3. 服务器上安装java7, mysql, tomcat7
服务器中要安装的环境主要根据我们项目中所需的环境。安装好环境之后，我们的webapp才可能服务器中正常运行。java7和mysql我们前面已经介绍过，所有这些安装问题其实都可以google找到解答。下面主要介绍tomcat7的安装过程：    
首先通过`ssh root@121.40.97.118`连接到ECS上：

```
cd /tmp  
wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-7/v7.0.56/bin/apache-tomcat-7.0.56.tar.gz
tar xzf apache-tomcat-7.0.56.tar.gz
mv apache-tomcat-7.0.57 /usr/local/tomcat7

```
安装完成后，通过 `cd /usr/lcoal/tomcat7`进入到tomcat根目录下，`./bin/startup.sh`运行tomcat服务器，本机浏览器中访问 `121.40.97.118:8080`，如果成功出现tomcat的配置页面，则tomcat7安装完成。当然我们业可以通过 `/usr/local/tomcat7/conf/tomcat-users.xml`来配置tomcat的管理员角色，具体请参考apache tomcat 官网。

###4. 复制war文件到tomcat7/webapps目录中
我们需要将第2步生成的demo.war复制到服务器的webapp目录中，直接在terminal中使用`scp`命令：  
`scp demo.war root@121.40.97.118:/usr/local/tomcat7/webapps/demo.war`，复制过程完成后，文件开始存在于服务器的webapps文件夹下面，war包能够被tomcat自动解析。访问`121.40.97.118:8080/demo`，我们可以看到自己的app终于可以访问了。
###5. More
到第4步，其实基本完成了部署过程。  
我们自己访问时必须加上8080端口号看的也是略不爽。此时可以通过`/usr/local/tomcat7/conf/server.xml`对里面默认的8080端口进行修改，改为80端口。这次我们直接访问`121.40.97.118/demo`，发现可以访问成功了。    
再看看，我们地址后面还需要加上demo，看起来业有点不爽。我想让这台主机默认直接访问demo工程。通过观察webapps看到里面有个ROOT目录，先备份ROOT目录，然后把我们的demo.war改称ROOT.war再看看效果:  

```
mv ROOT root.backup  
mv demo.war ROOT.war

```
再次访问`121.40.97.118`，这时候我们可以直接看到我们的web应用了。  

如果你还嫌不够酷的话，可以增加一个域名来转到这个网站。我在自己的域名管理中增加了一个二级域名`demo.gongmingqm10.net`映射到`121.40.97.118`，再次访问`demo.gongmingqm10.net`，但是事情没有预料的那么好，显示网站没有备案，好吧。其实下一步我可以去备案了～～


---
此博客主要用于自己学习的记录过程，小弟才疏学浅，如有纰漏的地方请各位看官多多指正，共同进步。
