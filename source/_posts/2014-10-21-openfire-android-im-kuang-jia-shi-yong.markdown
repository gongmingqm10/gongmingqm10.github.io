---
layout: post
title: "Openfire - Android IM 框架使用"
date: 2014-10-21 10:02:58 +0800
comments: true
categories: android
description: Android IM 实时通讯 Openfire使用纪录

---
IM(Instant Messaging)在Android中可谓运用广泛。QQ，Wechat，陌陌等应用都可以看作IM实时通讯APP，实时通讯在社交类APP中运用范围较广，其功能看起来也是比较cool的。今天我们将研究一下android IM软件是如何实现的。像IM这种实施通讯软件，除非公司有比较大的实力和精力，才会自己去整这么一套框架。对于广大中小软件开发者来说，想到比较多的就是开源框架。Opensource对于开发者来说简直就是福音，对于整个软件行业也起到了不小的推动性的作用。  
Openfire在这种环境下应运而生，而且作为实时通讯类开源框架迅速走红，下面我们就一起来学习这样一款拯救宇宙的开源框架，文章的最后会运用Openfire做出一个可以实时通讯的AndroidAPP，如果结合上地理位置再多点UI和交互上的设计提升，这不就是陌陌吗？YY了一会，觉得挺有趣，那么 `Just do it`。  

Openfire概念性介绍请点击：[http://blog.csdn.net/ithomer/article/details/7192257](http://blog.csdn.net/ithomer/article/details/7192257)  
Openfire官网：[http://www.igniterealtime.org/](http://www.igniterealtime.org/)  
Openfire安装文档：[http://www.igniterealtime.org/builds/openfire/docs/latest/documentation/install-guide.html](http://www.igniterealtime.org/builds/openfire/docs/latest/documentation/install-guide.html)

###1. 选择服务器
对于实时通讯软件，除了我们的客户端，服务器端更是关键。因为服务器是连接两个会话的桥梁。根据安装文档中，Openfire提供了多个平台的安装版本。Windows, Linux/Unix。所以支持还是很丰富的。至于服务器的选择我们可以选择自己的ECS，本地PC，或者Linux虚拟机。选择本机安装的请略过此步骤。  
为了更加符合真实的安装环境，我选择本地的Linux虚拟机作为服务器环境。方便以后部署到真实的服务器环境中。    
在进行下一步之前请先在server上安装mysql和java。
###2. Install openfire  
从官网上下载对应的版本，我这里选择[Openfire_3.9.3 Linux版本](http://www.igniterealtime.org/downloads/index.jsp)，登陆服务器后直接使用wget下载

```
wget http://www.igniterealtime.org/downloadServlet?filename=openfire/openfire-3.9.3-1.i386.rpm

//下载完成后使用rpm进行安装

rpm -ivh openfire-3.9.3-1.i386.rpm

```
Openfire安装完成后会在/opt目录下生成openfire/目录。
安装完成后就需要进行数据库的配置，按照官方文档：

```
Make sure that you are using MySQL 4.1.18 or later (5.x recommended) ¹.
Create a database for the Openfire tables:
mysqladmin create [databaseName]
(note: "databaseName" can be something like 'openfire')
Import the schema file from the resources/database directory of the installation folder:
Unix/Linux: cat openfire_mysql.sql | mysql [databaseName]; 
Windows: type openfire_mysql.sql | mysql [databaseName];
Start the Openfire setup tool, and use the appropriate JDBC connection settings.

```

首先使用`mysqladmin create openfire`创建名为openfire的数据库, `cd /opt/openfire/resources/database`进入openfire的数据库资源目录，使用`cat openfire_mysql.sql | mysql openfire`导入openfire的schema数据库文件。

###3. 启动Openfire服务
使用rpm安装完成后的openfire会在/etc/init.d/中自动生成openfire 文件，可以直接在这里打开服务.

```
Usage /etc/init.d/openfire {start|stop|restart|status|condrestart|reload}

```

运行openfire服务：`/etc/init.d/openfire start`，运行`/etc/init.d/openfire status`显示没有成功运行。  
查看log `cat /opt/openfire/logs`，显示`nohup: cannot run command /opt/openfire/jre/bin/java: No such file or directory`，看来openfire找不到java，所以不能成功启动，但是我们的系统的确已经安装了java，所以可以通过软链解决：  

```
cd /opt/openfire/jre/bin
cp java java.bak
rm java
ln -s /usr/bin/java java
service openfire start

```

软链完成之后，再次开启openfire服务：

```
[root@iZ23572i0rtZ bin]# /etc/init.d/openfire status
openfire is not running
[root@iZ23572i0rtZ bin]# /etc/init.d/openfire start
Starting openfire:
[root@iZ23572i0rtZ bin]# /etc/init.d/openfire status
openfire is running

```
status显示openfire已经成功启动。通过虚拟机ip访问9090端口，这时会跳转到setup界面，只需要通过setup便可以完成相关配置。

配置完成之后即可以登录自己的管理控制台。

###4. Integrate to Android
既然服务器安装完成，那我们可以着手我们的APP。与Openfire关联的客户端XMPP协议库是smack。摸索一番之后发现，要在Android中使用Smack必须使用ASmack库。[ASmack下载地址](http://asmack.freakempire.de/)。  
以实例为主，通过smack实现两个客户端之间的即时通信。[下载地址](https://github.com/gongmingqm10/SmackDemo)

####Attention
* 在AndroidManifest中必须添加Internet permission,否则连接失败。`<uses-permission android:name="android.permission.INTERNET"/>`.
* 客户端对客户端创建聊天时，SID为 username@XXXX, XXXX表示的是服务器名字，我这里是ECS主机名。

###5. Github Repository

[https://github.com/gongmingqm10/SmackDemo](https://github.com/gongmingqm10/SmackDemo)


