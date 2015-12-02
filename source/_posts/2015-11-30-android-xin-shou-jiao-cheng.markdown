---
layout: post
title: "Android 新手成长计划"
date: 2015-11-30 21:17:42 +0800
description: Android 新手入门教程, Android
comments: true
categories: Android

---

![Android starter](http://www.webmastergrade.com/wp-content/uploads/2011/04/Android-VS-Apple.jpg =500x)

最近在公司里开始遇到了新人培养的问题，新人被 assign 到 Android 项目上，为了让他能够很快有所产出，于是我们得思考怎么样快速锻炼一个新人并让他对 Android 开发产生兴趣并持续做下去呢？当然也会有一时兴起的同事会想着学习 Android，Android 入门门槛不高，但是要想真正的熟练驾驭，确实是一件道阻且长的事情，需要付出额外的努力。

## 兴趣驱动法则

其实“兴趣”不只是在 Android 开发上体现的重要，兴趣在任何地方都是最重要的。之前接触过一些朋友，某次会突然告诉我想学习 Android，但是往往一个 Hello World 都可以将之难倒。这时候学习者明显缺少兴趣驱动。Android 开发好玩的一点是我们开发的应用都可以很简单的在手机上安装使用。所以你可以尽情尝试各种新奇的点子。

当然，兴趣并不是本文讨论的重点，我们假设真心想学习的人都是出于兴趣。

## 循序渐进的练习计划

有了兴趣远远还不够，兴趣只能促使你完成一个 Hello World，要想真正做出东西还应该有循序渐进的计划来配合。我当时学习 Android 的方法就是每周一个任务。把学习新知识当作升级打怪，这样子你还会松懈吗？

### 你应该掌握的 Android 开发知识

Android 开发的入门门槛不高，但是细节知识真心不少。作为初学者，我们可以先了解广度，然后了解深度。刚开始可以接触尝试了解一点远离性的知识，然后直接运用一些流行的框架。快速迭代完成一个 App。那么对于新手而言，应该掌握的 Android 开发知识有哪些呢？

#### 1. Hello Android

开发调试基础看起来比较模糊，具体一点是完成一个 Hello World，让此应用运行在你的手机或者模拟器中。借此了解各个文件夹模块的作用以及 App 运行的基础原理。涵盖知识点如下：

* 搭建 Android 开发环境： Java + Android Studio + SDK；
* 熟悉项目结构：src, res, assets, values, layouts, drawable/mipmap, styles, AndroidManifest.xml, Activity；
* 解读 Hello World 运行机制：项目从哪里运行，Activity 与 layout 的关联，Activity 中如何找到布局中的实例化控件，Activity 的生命周期；

#### 2. 计算器

计算器的经典程度不亚于 Hello World，Android 计算器能够让初学者迅速建立信心，同时作为一个单纯的无网络交互的应用，计算器主要处理布局，点击事件，计算逻辑。因此在了解了 Hello World 之后做计算器，再合适不过了。计算器最终效果图如下：

![计算器](http://7xj9js.com1.z0.glb.clouddn.com/calculator.png =300x)

那么对于计算器这个 demo 的要求如下：

* 界面效果如上图，并且能够进行简单的整数运算：`1+1, 4*2, 6/3, 1-10` 等；
* 异常处理，除数为 0 时界面提示 `Error`；
* 小数点以及连续运算的支持；
* 继承 Robolectric 添加单元测试；
* 每次打开 App 结果栏会自动显示上一次的运算结果；
* 添加`History`菜单，点击列表显示历史计算表达式以及结果；

通过这些要求，涵盖的知识点如下：

* 布局管理器的了解及使用：RelativeLayout, LinearLayout；
* Android 控件添加点击事件，TextView 动态设值；
* Android 项目集成 Robolectric，添加简单的单元测试；
* Android 数据持久化保存 - SharedPreferences；
* Android 数据持久化存储 - Sqlite 数据库；
* Android ListView 列表及数据绑定；
* Android Activity 跳转 - Intent；
* 添加 [ButterKnife](http://jakewharton.github.io/butterknife/) 框架，重构项目；
* 添加 [GreenDao](http://greenrobot.org/greendao/) 框架，重构数据库数据存储及读取；

#### 3. 豆瓣图书列表 - 图片缓存

豆瓣图书列表最初起源于公司内部的一次 Android 培训，由于对 Android 初学者而言，还比较有意义，于是设置为专题的形式供大家练习。需求很简单，将给定的图书列表数据(JSON 格式)显示到界面上。包括图书信息在列表中展示，网络图片的加载以及缓存。JSON 数据[点击此处](http://7xj9js.com1.z0.glb.clouddn.com/douban-books.json)查看。完成之后效果图如下：

![Douban Books](http://7xj9js.com1.z0.glb.clouddn.com/douban-reading.png =300x)

具体要求如下：

* 列表显示图书信息：图书图片，名称，作者／出版社／出版年份，评价，简介；
* 多次滑动列表，保证文字以及图片显示不会发生错位，并且始终流畅（有列表控件重用）；
* 添加图片缓存：图片加载成功后，断开网络，退出 App，打开应用仍然能够正确显示图片；

通过这些要求我们期望初学者掌握的技能点如下：

* Android ListView 自定义列表样式；
* Android Adapter 使用 ViewHolder 进行控件复用；
* Android 主线程（UI线程）的概念，主线程阻塞的解决方案，Handler 的概念以及使用方法；
* 图片缓存的应用，内存级别的缓存以及磁盘级别的缓存；
* 使用 [Picasso](http://square.github.io/picasso/)/[ImageLoader](https://github.com/nostra13/Android-Universal-Image-Loader) 图片加载及缓存框架，简化开发；

如果你对图片缓存部分的实现有任何疑惑，请参考工程 [AndroidWorkShop](https://github.com/gongmingqm10/AndroidWorkshop/tree/2nd-step4)。图片磁盘缓存部分参考了 Android 官方教程。

#### 4. 豆瓣图书列表 - 网络请求

这个任务其实是对第 3 条任务的改造。不同之处在于，本次任务侧重于让初学者了解如何快速构建出一个有网络请求模块的 App。我们的效果图和上一条任务的相同，不同点在于本次我们会使用真实的网络API数据。并且会对 API 请求回来的数据进行缓存。具体要求如下：

* 推荐使用 [Retrofit](http://square.github.io/retrofit/) 框架，从豆瓣读书 API 网络获取[图书列表](https://api.douban.com/v2/book/search?q=Java)及[图书详情](http://api.douban.com/v2/book/2130190)。详情参见[豆瓣API](http://developers.douban.com/wiki/?title=guide)；
* 列表显示豆瓣图书列表，图片显示可直接使用 Picasso；
* 点击 Item 跳转至图书详情界面，显示该书籍详细信息；
* 缓存此 API 的搜索结果 (文件或者数据库Key-Value的形式缓存JSON数据)；
* 使用 SwipeRefreshLayout 添加下拉刷新功能，下拉刷新时重新添加请求数据并更新缓存；

此任务我们期望初学者掌握的知识点：

* Retrofit 框架的使用；
* Android 数据缓存的简易实现方案（数据库／文件形式）；
* 简单的列表点击事件，页面跳转等；
* [Android Design Support Library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html) 中常用的组件及使用；

#### 5. 设计并实现你自己的 App

如果你顺利完成了上述四条任务，那么这一步你应该已经具备了完成一个简单的 App 的技能，这一步可以按照个人兴趣完成自己的创意点子。在开始编码之前，可以先制作出一套 UI 交互图，然后整理 API 并开始编码。

### Android 学习常用的链接

Android 拥有全面的官方学习教程以及各种丰富的开源库。我们初学时可以多浏览官方教程学习一些好的实现，下面列举了一些很常用的链接：

1. [Android 官方文档](developer.android.com/training/index.html)
2. [Android 官方博客](http://android-developers.blogspot.com/)
3. [Google Material Design](www.google.com/design/spec/style/color.html#)
4. [Square Open Source](http://square.github.io/)
5. [Jake Wharton 的博客](http://jakewharton.com/)


全文很多观点都是一家之言，更多的是结合自己的项目经验以及学习经验总结而成，如有谬误及不当之处，请读者不吝赐教。




