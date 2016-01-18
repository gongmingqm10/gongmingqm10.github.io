---
layout: post
title: "OWASP Top 10 Training"
date: 2015-11-17 12:04:05 +0800
description: 
comments: true
categories: Security
published: true

---
## 环境配置及安装

我们使用 DVWA 工程来学习 OWASP TOP10 安全问题。本节的目的是搭建好这一个易受攻击的 PHP/Mysql 应用，以实例的方式学习这十大安全问题。

### 启动 Vagrant 虚拟机

*如果你决定直接在本地电脑上安装 XAMPP，请自动忽略此步。*

为了营造一个安全纯净无干扰的环境，我们选择 Vagrant 启动虚拟机，在虚拟机中安装 XAMPP 环境， Vagrantfile 配置如下：

```
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network :private_network, ip: "192.168.33.10"
end

```
从配置文件中，可以看出我们使用 ubunt 14.04 系统，并且给虚拟机设置了一个内网 IP， 意味着在主机中可以通过 IP 直接访问虚拟机中的网页。

`Vagrant up` 启动该虚拟机，并通过 `Vagrant ssh` 登录虚拟机。

### 安装 XAMPP

XAMPP(Apache + MariaDB + PHP + Perl)，为了安装 DVWA Training 的实例，我们需要先安装相应的环境。

XAMPP 有针对 Linux/Windows/Mac的安装包，下载地址参考[官方地址](https://www.apachefriends.org/index.html). 由于本次试验的在 ubuntu/trusty 虚拟机中，因此使用命令行安装，大致安装过程如下：

```
wget https://www.apachefriends.org/xampp-files/5.6.14/xampp-linux-x64-5.6.14-3-installer.run

sudo chmod +x xampp-linux-x64-5.6.14-3-installer.run
sudo ./xampp-linux-x64-5.6.14-3-installer.run

```
![XAMPP 首页](http://7xj9js.com1.z0.glb.clouddn.com/xampp-snapshots.png =400x)

安装完成后，使用 `sudo /opt/lampp/lampp start` 打开 lmapp 服务，这时候你应该能够访问 `192.168.33.10/xampp`。[详细请参考](http://ubuntuportal.com/2013/12/how-to-install-xampp-1-8-3-for-linux-in-ubuntu-desktop.html)


### 安装 DVWA

如果你已经能够访问 xampp 默认的主页，那么你几乎快成功了。XAMPP 中提供了 Apache 服务器，那么我们只需要将 DVWA 的包放在相应的 Web 目录下面，DVWA 项目就能够运行了。

DVWA 官网：[http://www.dvwa.co.uk/](http://www.dvwa.co.uk/).

DVWA Github 地址：[https://github.com/RandomStorm/DVWA](https://github.com/RandomStorm/DVWA).

开始从命令行安装：

```
wget https://github.com/RandomStorm/DVWA/archive/v1.9.tar.gz
tar -xzvf v1.9.tar.gz
mv DVWA-1.9 /opt/lampp/htdocs/DVWA

```

访问 `192.168.33.10/DVWA`，自动跳转到 setup 界面，这时我们只需要按照提示就可以完成对应的初始化操作。如果你没有预先设置数据库用户名和密码，那么你会看到不能连接到 Mysql 数据库，根据提示重新设置数据库用户名和密码以继续操作。

![DVWA Setup](http://7xj9js.com1.z0.glb.clouddn.com/DVWA%20Setup.png =400x)


按照 [DVWA README](https://github.com/RandomStorm/DVWA) 中进行更改，试验过程中如果发现 `allow_url_include` 始终 Disable, 无法修改成功。请尝试在 `/opt/lampp/etc/php.ini` 中将原来的 `allow_url_include = Off` 修改为 `allow_url_include = on`。

将 PHP 相关的配置修改完成后，注意需要将 XMAPP 中 mysql root 用户默认的密码更改为`p@ssw0rd`，这个密码在 `/opt/lampp/htdocs/DVWA/config/config.inc.php` 中可以找到。当然也可以将此配置中的数据库密码更改为默认的空。

更改完成后，点击 `Create / Reset Database` 能够创建初始数据，随后访问 `192.168.33.10/DVWA` 即可跳转至登录界面。使用默认的 admin/password 可以登录此系统。登录成功之后，意味着我们可以通过实例学习 OWASP TOP10 的安全问题啦。

![DVWA index](http://7xj9js.com1.z0.glb.clouddn.com/DVWA%20Index.png =400x)

## DVWA Practice

关于十大安全问题的实践：未完，持续更新中 :)

### Brute Force

密码暴力破解，根据理解就是根据一个用户名密码字典进行试错。

通过以下链接找到适合你的 Brute Force 的工具：[http://resources.infosecinstitute.com/10-popular-password-cracking-tools/?dl=true](http://resources.infosecinstitute.com/10-popular-password-cracking-tools/?dl=true)

在 Mac 下面我使用 [ZAP](https://github.com/zaproxy/zaproxy/wiki/Downloads?tm=2) 进行暴力破解。ZAP 有适用于 Mac 版本的安装包。

...


