---
layout: post
title: "Hack 京东 Lebook"
date: 2014-10-26 22:46:18 +0800
comments: true
categories: Android
published: true
description: Android获取京东Lebook Epub文件

---

adb devices : 列出所有的设备

adb -s XXXX shell: 进入指定的shell

adb shell: 进入唯一的shell

在terminal中直接： adb pull /data/data/com.jingdong.app.reader/files/epub/3690098/534543.jeb   
~/Desktop/test.epub。可以直接拿到epub文件
