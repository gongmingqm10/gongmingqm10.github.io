---
layout: post
title: "DevOps Session 3"
date: 2015-10-12 09:19:37 +0800
description: DevOps, Vagrant, Jenkins
published: false
comments: true
categories: DevOps
---

##前言

###什么是 Vagrant

> Create and configure lightweight, reproducible, and portable development environments.

创建以及配置轻量的、可再生的、可移植的开发环境。

简单说来，Vagrant 可以帮助你构建一个统一的开发环境，统一的开发环境包括系统的一致性，系统中所安装的软件的一致性。这些一致性能够
保证各开发人员的开发环境保持完全的一致，从而减少了开发环境不一致所可能引起的各种问题。

如果你想了解更多，请访问 [Vagrant网站](https://www.vagrantup.com/)。

###什么是 Jenkins

> An extensible open source continuous integration server

在之前的博客中对 Jenkins 也有过详细的介绍，Jenkins 属于一个开源的持续集成服务器。Jenkins 能够快速的构建各种流水线，各种流水线
会执行不同的命令，比如运行测试、部署服务器等。Jenkins 拥有丰富的组件支持，因此在项目中也应用广泛。

了解更多，请访问 [jenkins-ci.org](https://jenkins-ci.org/)。

<!-- more -->

###课程目标及内容

本次课目标如下：

1. 掌握Vagrant的使用，自动化配置管理；
2. 掌握Jenkins的使用，使用构建流水线，发布软件包；

课程内容如下：

1. Vagrant基础，包括创建虚拟机、使用shell脚本配置等；
2. Jenkins基础，包括插件安装、buildpipeline构建，常用管理功能；

##课前准备

1. 安装vagrant；
2. 使用vagrant镜像安装ubuntu系统，并运行。网络设置需要达到前两次课程的效果，能够从外部ssh访问虚拟机；
3. 使用vagrant配置脚本创建角色：devops，在该角色下创建用户：jenkins；
4. 使用vagrant配置脚本，在jenkins用户下安装jenkins；
5. 启动jenkins，并创建jenkins访问管理用户：user_test；

验收条件：可以使用user_test登陆jenkins页面进行设置。
