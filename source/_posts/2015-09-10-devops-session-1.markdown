---
layout: post
title: "DevOps Session 1"
date: 2015-09-10 22:09:55 +0800
description:
comments: true
categories: DevOps

---

最近公司在组织DevOps培训，于是就有个这个系列的文章，作为培训笔记，作为以后参考。如果你还不知道DevOps概念，请参考[Wikipedia](https://zh.wikipedia.org/wiki/DevOps)。

> DevOps（英文Development和Operations的组合）是一组过程、方法与系统的统称，用于促进开发（应用程序/软件工程）、技术运营和质量保障（QA）部门之间的沟通、协作与整合。

##课前准备

因为需要使用一台Linux机器来作为测试主机，我们使用VirtualBox安装[Ubuntu Server](http://releases.ubuntu.com/15.04/ubuntu-15.04-server-amd64.iso)。安装完成之后，我们能够登录这台“服务器”。

<!-- more -->

##课堂内容

###磁盘管理

主要学习磁盘相关的基础命令，能够查看磁盘使用情况，挂载新的磁盘等操作。参考文章：[Linux磁盘管理三板斧的使用心得](http://os.51cto.com/art/201012/240726_all.htm)。

#### df 命令

`df -h` 显示磁盘使用情况

#### fdisk 命令

`sudo fdisk -l` 列出已知的分区类型

`sudo fdisk /dev/sda1` 对/dev/sda1进行磁盘分区

#### mkfs 命令

mkfs 命令用于在设备上（通常为磁盘）创建 Linux文件系统。mkfs 本身并不执行建立文件系统的工作，而是去调用相关的程序来执行。

`sudo mkfs -t ext4 /dev/sda1` 将 /dev/sda1 分区格式化为 ext4 格式

#### partprobe 命令

使用 partprobe 可以不用重启系统配合 fdisk 工具创建新的分区。

#### mount 命令

mount 命令用于挂载文件系统

###常用目录操作

####特殊目录

* `.` 当前目录
* `..`父级目录
* `~` 当前用户的home目录
* `-` 前一个工作目录
* `~account` account 用户的 home 目录
* `.filename` 隐藏文件

####操作目录

* `cd` 进入某个目录
* `ls` 列出某个目录的文件信息
* `ls -l` 列表展现目录下所有文件信息，显示目录权限、创建者、时间、文件大小等
* `ls －al` 列表展现目录下所有文件（包含隐藏文件）信息，显示目录权限、创建者、时间、文件大小等
* `pwd` 显示当前目录
* `mkdir` 创建目录
* `rmdir` 删除空目录（如果需要删除非空目录，请使用 `rm -r`）

###常用文件操作

* `cat` 显示文件所有内容
* `tac` 从最后一行开始显示
* `echo "hello world" > log.txt` 向 log.txt 中写入内容 "hello world"， 将覆盖原来的文件内容
* `echo "hello world 1" >> log.txt` 向 log.txt 中附加内容 "hello world1"，添加的内容会在文件最后一行。如果文件不存在，则会被创建。
* `nl log.txt` 输入带行号的文件内容
* `head log.txt` 只看文件前几行内容
* `tail log.txt` 只看最后几行
* `od` 以二进制方式显示档案内容
* `cp` 复制
* `mv` 移动
* `rm` 删除

###查找命令

#### find 命令

find 命令主要用来查找文件。更多使用方法请 Google 之。

`find . -name "mysql*" -type f` 查找以“mysql”开头的所有文件(不是文件夹)

#### which 命令

在 PATH 变量指定的目录中，搜索某个系统命令的位置。同时也可以验证某个命令是否存在。

`which grep`


## 总结

第一节课主要了解了 Linux 下面一些基础的命令使用方法。这些命令都是比较常用的，当然还是需要多实战，才能够更好的理解并运用TA。
