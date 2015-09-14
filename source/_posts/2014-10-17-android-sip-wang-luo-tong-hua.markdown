---

layout: post
title: "Android SIP 网络通话"
date: 2014-10-17 09:49:35 +0800
comments: true
categories: Android
description: Android网络通话方案 SIP

---

突发奇想，想研究一下Android的网络通话怎么实现，于是从Google搜到了相关的资料。原来Android下集成了SIP（Session Initiation Protocol）。SIP的账号可以通过SIP提供商免费申请，申请后即可以通过用户名实现网络通话。于是也找了网上的一些资料，写了个AndroidSIP的小demo，demo实现后大家可以实现互拨，目前界面很简单，但是通话的功能应该是可以实现的。请各位看官试用之。  
整体效果看起来还是蛮酷的，这里我们申请了两个免费的账号，在设备上安装之后就可以进行Network Call了。


免费SIP账号申请：[http://www.linphone.org/free-sip-service.html](http://www.linphone.org/free-sip-service.html)

官方参考文档：[http://developer.android.com/guide/topics/connectivity/sip.html](http://developer.android.com/guide/topics/connectivity/sip.html)  

项目代码下载：[https://github.com/gongmingqm10/AndroidSIP](https://github.com/gongmingqm10/AndroidSIP)  

任何问题，欢迎大家fork，发pull/request。
