---
layout: post
title: "Android Data Cache"
date: 2014-10-28 17:11:26 +0800
comments: true
categories: android

---

Android APP开发过程中，我们通常会加入缓存模块。缓存即在本地保存APP的一些数据，大部分是将网络请求的数据进行本地保存，这样在缓存数据有效期内就可以直接使用缓存数据，降低了APP和服务器的压力，也极大提升了用户体验。Android数据缓存既可以以数据表的形式进行保存，也可以以文件的形式进行缓存。这里我主要通过缓存文件存储数据，并在APP下一次启动时读取。

###Usage
整个Cache模块的设计思想很简单，每个缓存数据都对应一个key，每个缓存数据又会被存到以此key命名的文件中，需要时直接读取。关键类分别为`CacheData`, `CacheManager`, `CacheUtils`。 

* CacheData：数据封装类，所有欲缓存的数据都通过CacheData封装，CacheData中能够定义缓存有效期，并可以通过getData()直接获取真实的缓存数据。
* CacheManager: 缓存管理类，单例模式设计，负责缓存的存储和读取。
* CacheUtils: 缓存常用类，所有的缓存Key都应该在这个类中定义，此类还定义了一些时间常量，缓存有效期中可以使用。

使用方法如下：  

* step1: `CacheManager.init(Context context)`, 在APP加载时就应该对CacheManager初始化。
* step2: 自定义的Model需要实现序列化，使用CacheData进行包装，然后使用CacheManager进行存储和读取。

###Code Download

项目代码托管在[Github Repo](https://github.com/gongmingqm10/AndroidUikit/tree/master/library/src/main/java/org/gongming/common/cache)中，欢迎star和fork。


