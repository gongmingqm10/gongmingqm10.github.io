---
layout: post
title: "Android Activity 启动方式及 Intent Flag"
date: 2015-09-29 10:57:09 +0800
description:
comments: true
categories: Android
published: false

---

Activity可以简单看作应用中的一个个简单页面，那么页面之间的切换也就是我们这里需要希望讲的重点。Android里面页面间的切换可以浏览器段复杂多了。浏览器只有简单的回退按钮，以及方面的URL导航。但是在Android里面，页面切换逻辑直接关系到应用体验以及系统的逻辑。并且有时候为了满足某些特定的业务需求，我们需要有全面的了解，才能在需求面前游刃有余。

![Android Intent Flag](http://7xj9js.com1.z0.glb.clouddn.com/Android%20Intent%20Flag.gif)

与 Activity 跳转紧密相关的是 Android 的三个虚拟键，返回键，Home 键，菜单键。

* 菜单键：菜单键主要用于 App 之间的快速切换。比如用户打开了微信和 QQ。用户当前正在微信中，可以很快通过菜单键切换到 QQ 中与好友互动。
* Home 键：Home 键与菜单键的作用类似，Home 键能够将用户导航到手机主页面，这时候用户可以继续进行其它各种操作。用户在 Home 页面，也可以点击应用图标再次返回到已经打开的应用中。
* 返回键：返回键作用很简单，用户在 App 某个页面中点击返回键，用户能够被导航到上一个页面，如果用户当前的页面是顶级页面（没有上一个页面），那么认为用户退出这个 App。所以，返回键也是最常被开发者复写的一个按键。常见的情形有，双击返回键退出应用等。

本文中我将使用实例循序渐进介绍 Android 中页面的跳转方式及实现方式。

<!-- more -->


###1. 登录界面

在第一个实例中，我们以最常见的登录界面为例：用户首次打开App, 直接进入登录界面，用户在登录页面中需要正确的用户名和密码才能进入主页面。一旦用户成功登录过，则将用户登录状态保存到 SharedPreferences 中。每次打开 App 都会检查用户是否登录过，没有登录则需要先进入登录页面，反之直接进入首页。

![登录逻辑](http://7xj9js.com1.z0.glb.clouddn.com/Flag%20Practice%201.png =x400)
