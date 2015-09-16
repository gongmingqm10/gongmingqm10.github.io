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

## 用户权限与控制

Linux 有着完善的用户管理，不同的用户大致分为如下几组：

* 文件拥有者
* 群组
* 其他人

#### passwd

用来记录文件的基本属性

```
cat /etc/passwd
```

### 将 jenkins 和 tomcat 用户添加到 DevOps 群组中

首先新建群组，然后将用户添加到群组中。

```
ming@ubuntu:/$ sudo groupadd DevOps
ming@ubuntu:/$ sudo usermod -G DevOps jenkins
```

### 查看群组下所有用户

为了验证我们上述的用户添加组操作已经成功，需要查看群组下所有用户：

```
cat /etc/group
```

### 给 DevOps 设置管理员 ming

使用 gpasswd 命令来设置：`sudo gpasswd －A ming DevOps`

```
ming@ubuntu:/$ sudo gpasswd --help
Usage: gpasswd [option] GROUP

Options:
  -a, --add USER                add USER to GROUP
  -d, --delete USER             remove USER from GROUP
  -h, --help                    display this help message and exit
  -Q, --root CHROOT_DIR         directory to chroot into
  -r, --remove-password         remove the GROUP's password
  -R, --restrict                restrict access to GROUP to its members
  -M, --members USER,...        set the list of members of GROUP
  -A, --administrators ADMIN,...
                                set the list of administrators for GROUP
Except for the -A and -M options, the options cannot be combined.
```

## 文件权限

以下列目录为例，查看该目录下面所有文件/文件夹的权限：

```
ming@ubuntu:/$ ls -l
total 80
drwxr-xr-x   2 root root  4096 Sep 14 15:51 app
drwxr-xr-x   2 root root  4096 Sep 16 15:31 bin
drwxr-xr-x   3 root root  4096 Sep  7 17:30 boot
drwxr-xr-x  18 root root  4160 Sep 16 13:46 dev
drwxr-xr-x 107 root root  4096 Sep 16 19:43 etc
drwxr-xr-x   6 root root  4096 Sep 16 19:29 home
lrwxrwxrwx   1 root root    33 Sep  7 17:16 initrd.img -> boot/initrd.img-3.19.0-15-generic
drwxr-xr-x  21 root root  4096 Sep  7 17:26 lib
drwxr-xr-x   2 root root  4096 Sep  7 17:16 lib64
drwx------   2 root root 16384 Sep  7 17:16 lost+found
drwxr-xr-x   3 root root  4096 Sep  7 17:16 media
drwxr-xr-x   2 root root  4096 Apr 18 05:34 mnt
drwxr-xr-x   3 root root  4096 Sep 16 15:51 opt
dr-xr-xr-x 103 root root     0 Sep 16 13:46 proc
drwx------   2 root root  4096 Sep  7 17:16 root
```
r(可读, 2^2) w (可写, 2^1) x (可执行, 2^0)

第一个参数表述文件类型：文件夹、文件、链接等。

rwx(Owner) rwx(Group) rwx(Others)

### 改变文件属性

使用 usermod 将用户 jenkins 主目录设置为 `/devops/jenkins`：

```
sudo usermod -m -d /devops/jenkins jenkins
```

更改 `devops/jenkins` 和 `/devops/tomcat` 文件可以被 jenkins 和 tomcat 用户互相读：

```
ming@ubuntu:/devops$ sudo chmod 740 jenkins
ming@ubuntu:/devops$ ls -l
total 8
drwxr----- 2 jenkins jenkins 4096 Sep 16 20:11 jenkins
drwxr-xr-x 5 tomcat  tomcat  4096 Sep 16 20:11 tomcat
```

## 参考

如果你的 VirtualBox 虚拟机不能通过 SSH 登录，请参考 [Enabling VirtualBox SSH on IPv6](http://louisrli.github.io/blog/2012/08/30/virtualbox-ipv6-ssh/#.VflDXZ2qqkp).

Linux 命令大全之 passwd 命令：[http://man.linuxde.net/passwd](http://man.linuxde.net/passwd)

鸟哥的 Linux 账号管理与 ACL 权限设定：[http://linux.vbird.org/linux_basic/0410accountmanager.php](http://linux.vbird.org/linux_basic/0410accountmanager.php)



