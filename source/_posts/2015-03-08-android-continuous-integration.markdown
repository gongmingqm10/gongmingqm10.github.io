---
layout: post
title: "Android Continuous Integration"
date: 2015-03-08 23:39:35 -0500
description: Android CI, Android Testing, Android Jenkins, Android Robolectric, Android Calabash
comments: true
categories: android

---

随着Android平台的逐渐成熟，伴随着一系列针对Android测试框架的推出，开发人员终于可以如愿以偿的在移动端的开发上进行单元测试，集成测试以及功能测试。在敏捷流程中从开发，到测试，到验收最终成为面向用户的Release版本，经历的是Story一个完整的生命周期。CI(Continuous Integration, 持续交付)在敏捷实践中也因此扮演了非常重要的角色。

如果说Web的持续集成，以及各类测试框架有一定的历史积淀了。那么Android的持续集成可以说是新鲜事物，大部分IT公司知道如何对服务器端或者Web端进行一系列自动化测试，保证其功能的正确性。而对于移动端的产品比较多的则是由测试人员组成的人肉测试。移动端的这种人工测试，无论是对测试人员，还是要经常打包并且来修复各种Bug的开发人员来说，其代价是巨大的。

<!-- more -->

从Android 2.3.3 版本就开始，我就成为了Android的开发者。从开发者的角度见证了Android的步步升级，也从普通用户的角度见证了Android在中低端市场上的统治权。虽然我对Apple的产品也很满意，但是我对Android的感情却也是无法割舍的。我庆幸自己终于能够在Android上也见证测试驱动开发的实践，也庆幸自己有机会去亲身实践，从零开始学习并了解Android的持续集成。


###前言

本文主要是从零开始，以学习者的角色来探索如何构建可用的Android CI环境。最后的目标是，在Jenkins上从build单元测试，到功能测试的运行，最后通过一键部署编译出可供QA测试的QA版本，可供Release的Release版本，并借助HockeyApp，生成可下载的链接。

1. 构建出一个含有单元测试和集成测试简单的App原型。
2. 通过vagrant在本地运行Ubuntu虚拟机，并安装Jenkins服务器，在Ubuntu上安装配置Android运行环境。
3. 在Jenkins中创建Android Pipeline, Android Build -> Android Functional Test -> Android Deploy Hockey App。
4. 在Android工程中创建不同的环境变量，使得构建时能够选择不同的构建变量，编译生成QA和Release等不同环境下的App。

### 1. 构建Android基础工程，本地运行测试
这里的基础工程主要指的是能够运行测试的基础功能，我选择使用Robolectric做单元测试，使用espresso做继承测试（后面的文章也会探讨使用）。基础工程主要参考了[robolectric/decard-gradle](https://github.com/robolectric/deckard-gradle), 通过一些gradlew的配置，使我们的工程能够直接运行单元测试和集成测试。

运行单元测试:

```
./gradlew test

```

运行espresso测试：

```
./gradlew connectedAndroidTest
```

一旦你这个实例运行成功，说明这一步其实已经完成。后面我们只需要通过jenkins来跑这几条命令，然后展示结果即可。


### 2. 在虚拟机中安装Jenkins和Android环境
为了从零开始，我选择了一台干净的Ubuntu/trusty机器，为了使构建从零开始，可以更好的模拟我们在EC2机器真实的情
形。从测试的角度来说，我选择使用[Vagrant](https://www.vagrantup.com/)来启动虚拟机。

```
XiaoMing:ci minggong$ vagrant init ubuntu/trusty64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
XiaoMing:ci minggong$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'ubuntu/trusty64'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Fixed port collision for 22 => 2222. Now on port 2200.
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2200 (adapter 1)
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] Mounting shared folders...
[default] -- /vagrant
XiaoMing:ci minggong$ vagrant ssh
Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.13.0-24-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Last login: Tue Apr 22 19:47:09 2014 from 10.0.2.2
```

如果在Vagrantfile中已经设置了静态IP，则可以直接通过IP登陆。vagrant默认用户名为 `vagrant`, 密码为`vagrant`。

使用 `vagrant ssh` 登录虚拟机之后，则开始安装各种环境：

* Java
* Jenkins
* Android Environment

#### 2.1 安装Java

apt-get直接安装jdk1.7:

```
vagrant@ubuntu-14:~$ sudo apt-get update
vagrant@ubuntu-14:~$ sudo apt-get install openjdk-7-jdk -y
vagrant@ubuntu-14:~$ java -version
java version "1.7.0_75"
OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-1~trusty1)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)

```

#### 2.2 安装Jenkins
参考[Jenkins官方教程](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu):

```
wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins -y

```
安装完Jenkins后，首先验证下Jenkins是否安装完毕, `curl http://localhost:8080`, 如果发现能够输出一堆HTML标签，那么证明Jenkins已经安装成功并已经启动了。Jenkins常见的使用命令如下，默认启动在8080端口：

```
vagrant@ubuntu-14:~$ sudo /etc/init.d/jenkins -help
Usage: /etc/init.d/jenkins {start|stop|status|restart|force-reload}
```
Jenkins默认de配置文件为`/etc/default/jenkins`, 可以看到Jenkins_home定义的路径：

```
 #jenkins home location
 JENKINS_HOME=/var/lib/jenkins
```
如果需要更改默认的Jenkins home 路径，需要更改 `/etc/default/jenkins`地址，然后重新启动jenkins即可生效。


#### 2.3 安装Android运行环境
最后一步我们需要安装Android运行环境，这样我们的Jenkins才能够运行Android的相关测试。主要参考文章 [How to Build Apps with Jenkins](https://www.digitalocean.com/community/tutorials/how-to-build-android-apps-with-jenkins)

* [Android SDK官网](http://developer.android.com/sdk/index.html)找到最新的SDK安装包地址：http://dl.google.com/android/android-sdk_r24.1.2-linux.tgz, 下载并解压：

```
$ cd /opt
$ sudo wget http://dl.google.com/android/android-sdk_r24.1.2-linux.tgz
$ sudo tar zxvf android-sdk_r24.1.2-linux.tgz
$ sudo rm android-sdk_r24.1.2-linux.tgz

```

* 设置Android环境变量：

```
$ vi /etc/profile.d/android.sh

```

Add the following to android.sh file

```
export ANDROID_HOME="/opt/android-sdk-linux"
export PATH="$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH"

```

然后 `source /etc/profile`， 使文件生效。

* 配置 Android SDK

最简单粗暴的方法是下载所有的SDK及相关的一系列工具，当然也可以挨个下载，具体参考上面提到的这篇文章。我采用直接更新全部的方法：

```
android update sdk --no-ui

```

经过漫长的等待，下载完毕之后这部分的工作也基本完成。(这一步一般留在半夜自行下载)

#### 2.4 安装Git

为了使Jenkins能够直接从Git Repository上下载代码，需要在Ubuntu中安装Git:

```
sudo apt-get update
sudo apt-get install git -y

```

#### 2.5 配置Jenkins

Jenkins启动后，首先注册登录用户，然后创建新的Build Project。构建就基本完成了。当然你也可以在前一篇博客 [Set up Jenkins for Android integration using Docker](http://www.gongmingqm10.net/blog/2015/03/11/setup-jenkins-for-android-integration-using-docker/)中找到一些插件，通过这些插件能够更好的实现Android的持续集成，包括通过Hockey App 持续的发布新的版本。

具体安装细节，可以参考[Setup Android CI using Docker](http://www.gongmingqm10.net/blog/2015/03/11/setup-jenkins-for-android-integration-using-docker/)

粗略说来，安装插件: `Git plugin, Android emulator plugin, Copy artifact plugin, Android lint plugin, Hockey app plugin, Build monitor plugin`

运行时发现jenkins用户对jenkins-home目录没有操作权限，需要通过chmod增加权限。在当前vagrant用户下，`su jenkins` 能够切换到jenkins目录下，但是需要提供密码。所以可以自行修改密码：

```
vagrant@ubuntu-14:/var/lib/jenkins$ sudo passwd jenkins
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
vagrant@ubuntu-14:/var/lib/jenkins$ su jenkins
Password:
jenkins@ubuntu-14:~$
```
权限设置成功后，再次运行测试，提示Android Build Tools没有安装成功：

```
+ ./gradlew clean build

FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> failed to find Build Tools revision 21.1.2


```

Android Build Tools的版本号是我们在app/build.gradle中指定的21.1.2。通过 `android list sdk --all`能够查看所有可以下载的sdk列表。

```
Packages available for installation or update: 138
   1- Android SDK Tools, revision 24.1.2
   2- Android SDK Platform-tools, revision 22
   3- Android SDK Build-tools, revision 22.0.1
   4- Android SDK Build-tools, revision 22 (Obsolete)
   5- Android SDK Build-tools, revision 21.1.2
   6- Android SDK Build-tools, revision 21.1.1 (Obsolete)
   7- Android SDK Build-tools, revision 21.1 (Obsolete)
   8- Android SDK Build-tools, revision 21.0.2 (Obsolete)
   9- Android SDK Build-tools, revision 21.0.1 (Obsolete)

```

可以看到21.1.2的索引值是5，所以可以使用 `android update sdk -u --all --filter <number>`更新，发现出现权限问题。

```
Installing Archives:
  Preparing to install archives
  Downloading Android SDK Build-tools, revision 21.1.2
  URL not found: /opt/android-sdk-linux/temp/build-tools_r21.1.2-linux.zip (Permission denied)
  Done. Nothing was installed.
```
可以直接在当前用户下给/opt/android-sdk-linux以及其子文件夹增加777权限。

```
sudo chmod -R 777 /opt/android-sdk-linux
```
然后安装Android build tools 21.1.2, `android update sdk -u --all --filter 5`。安装完成后还可能会出现错误:

```
:app:mergeDevDebugResources FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:mergeDevDebugResources'.
> Error: org.gradle.process.internal.ExecException: A problem occurred starting process 'command '/opt/android-sdk-linux/build-tools/22.0.1/aapt''
```

Google了一番，才发现是Android在64为Linux机器上运行而产生的问题，需要安装`ia32-libs`, 但是通过apt安装确被告知ia32-libs不存在，无法安装。[stackoverflow](http://stackoverflow.com/questions/23182765/how-to-install-ia32-libs-in-ubuntu-14-04-lts-trusty-tahr)解决办法如下：

```
sudo -i
cd /etc/apt/sources.list.d
echo "deb http://old-releases.ubuntu.com/ubuntu/ raring main restricted universe multiverse" >ia32-libs-raring.list
apt-get update
apt-get install ia32-libs
```
安装成功后，Android 单元测试即可成功运行。

单元测试之后，我们往往需要运行功能测试，功能测试的时候则需要打开模拟器。也就是 headless android emulator。首先在虚拟机中创建 avd, 然后启动运行模拟器。

运行之前需要先安装 API-21对应的 armeabi-v7a， 才能够创建虚拟机。通过查看`android list sdk --all`得知其对应的序列号是68，所以可以通过android update安装：

```
android update sdk -u --all --filter 68
```
安装完成后创建API-21虚拟机：

```
android create avd -f -a -s 1080x1920 -n Nexus-21 -t android-21 --abi
```
虚拟机创建成功后需要打开虚拟机：

```
emulator -avd Nexus-21 -no-skin -no-audio -no-window
```
顺利的话，到这里功能测试应该也能够运行了。但是在我的虚拟机里面却没办法启动android emulator。看来想构建真正的android功能测试，还是用一台配有显示器的Mac mini来运行比较的靠谱。

### 3.Other
如何在Android中配置不同的Flavor，如何上传构建好的App到HockeyApp中。欢迎查看我的[博客](http://www.gongmingqm10.net/blog/2015/03/11/setup-jenkins-for-android-integration-using-docker/)。
