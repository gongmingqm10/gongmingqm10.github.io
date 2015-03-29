---
layout: post
title: "Android Continuous Integration"
date: 2015-03-08 23:39:35 -0500
description: Android CI, Android Testing, Android Jenkins, Android Robolectric, Android Calabash 
comments: true
categories: android
published: false

---

随着Android平台的逐渐成熟，伴随着一系列针对Android测试框架的推出，开发人员终于可以如愿以偿的在移动端的开发上进行单元测试，集成测试以及功能测试。在敏捷流程中从开发，到测试，到验收最终成为面向用户的Release版本，经历的是Story一个完整的生命周期。CI(Continuous Integration, 持续交付)在敏捷实践中也因此扮演了非常重要的角色。

如果说Web的持续集成，以及各类测试框架有一定的历史积淀了。那么Android的持续集成可以说是新鲜事物，大部分IT公司知道如何对服务器端或者Web端进行一系列自动化测试，保证其功能的正确性。而对于移动端的产品比较多的则是由测试人员组成的人肉测试。移动端的这种人工测试，无论是对测试人员，还是要经常打包并且来修复各种Bug的开发人员来说，其代价是巨大的。

从Android 2.3.3 版本就开始，我就成为了Android的开发者。从开发者的角度见证了Android的步步升级，也从普通用户的角度见证了Android在中低端市场上的统治权。虽然我对Apple的产品也很满意，但是我对Android的感情却也是无法割舍的。我庆幸自己终于能够在Android上也见证测试驱动开发的实践，也庆幸自己有机会去亲身实践，从零开始学习并了解Android的持续集成。


###大纲

本文主要是从零开始，以学习者的角色来探索如何构建可用的Android CI环境。最后的目标是，在Jenkins上从build单元测试，到功能测试的运行，最后通过一键部署编译出可供QA测试的QA版本，可供Release的Release版本，并借助HockeyApp，生成可下载的链接。

1. 构建出一个含有单元测试和集成测试简单的App原型。
2. 通过vagrant在本地运行Ubuntu虚拟机，并安装Jenkins服务器，在Ubuntu上安装配置Android运行环境。
3. 在Jenkins中创建Android Pipeline, Android Build -> Android Functional Test -> Android Deploy Hockey App。
4. 在Android工程中创建不同的环境变量，使得构建时能够选择不同的构建变量，编译生成QA和Release等不同环境下的App。
5. 聊聊Travis CI，聊聊ansible，尝试将Android环境的安装过程通过命令或者一键化实现。

### 1. 构建Android基础工程，本地运行测试




