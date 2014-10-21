---
layout: post
title: "Openfire - Android IM 框架使用"
date: 2014-10-21 10:02:58 +0800
comments: true
categories: android
published: false

---
IM(Instant Messaging)在Android中可谓运用广泛。QQ，Wechat，陌陌等应用都可以看作IM实时通讯APP，实时通讯在社交类APP中运用范围较广，其功能看起来也是比较cool的。今天我们将研究一下android IM软件是如何实现的。像IM这种实施通讯软件，除非公司有比较大的实力和精力，才会自己去整这么一套框架。对于广大中小软件开发者来说，想到比较多的就是开源框架。Opensource对于开发者来说简直就是福音，对于整个软件行业也起到了不小的推动性的作用。  
Openfire在这种环境下应运而生，而且作为实时通讯类开源框架迅速走红，下面我们就一起来学习这样一款拯救宇宙的开源框架，文章的最后会运用Openfire做出一个可以实时通讯的AndroidAPP，如果结合上地理位置再多点UI和交互上的设计提升，这不就是陌陌吗？YY了一会，觉得挺有趣，那么 `Just do it`。  

Openfire概念性介绍请点击：[http://blog.csdn.net/ithomer/article/details/7192257](http://blog.csdn.net/ithomer/article/details/7192257)  
Openfire官网：[http://www.igniterealtime.org/](http://www.igniterealtime.org/)  
Openfire安装文档：[http://www.igniterealtime.org/builds/openfire/docs/latest/documentation/install-guide.html](http://www.igniterealtime.org/builds/openfire/docs/latest/documentation/install-guide.html)

###1. 选择服务器
对于实时通讯软件，除了我们的客户端，服务器端更是关键。因为服务器是连接两个会话的桥梁。根据安装文档中，Openfire提供了多个平台的安装版本。Windows, Linux/Unix。所以支持还是很丰富的。至于服务器的选择我们可以选择自己的ECS，本地PC，或者Linux虚拟机。选择本机安装的请略过此步骤。  
为了更加符合真实的安装环境，我选择本地的Linux虚拟机作为服务器环境。方便我以后更好的把服务放在真实的云服务器上。