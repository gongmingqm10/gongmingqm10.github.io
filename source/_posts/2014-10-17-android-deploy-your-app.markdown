---
layout: post
title: "Android deploy your app"
date: 2014-10-17 12:56:13 +0800
comments: true
categories: android
description: Android打包编译及发布

---

发布Android APP说通俗点就是“打包”，大部分情况下我们可以直接通过IDE进行签名打包，打包好的apk文件就可以上传到各大应用市场啦。今天项目开发过程中突然遇到需要更换debug.keystore的问题了，于是突然想好好研究一下Android打包的全过程，以及各个文件的作用。说做就做，how to publish your app?    

###Proguard－代码混淆
我们经常在js库中看到各种代码混淆，代码混淆主要是对你的代码的类名，方法名以及字段名进行命名，并删除一些无用的换行空格等。让你的代码变得难以读懂，这样能够防止别人反编译你的代码后窃取你的内容。  
android中代码混淆主要使用IDE自动生成的文件，并通过一些简单的配置就可以实现。中间的混淆步骤则不需要我们关系，android编译过程中自身会处理这些内部的逻辑。我们只需要配置文件就可以。  
通过官网提供的资料，[http://developer.android.com/tools/help/proguard.html](http://developer.android.com/tools/help/proguard.html)，以及网上的大部分资料都是过时的，目前的AndroidStudio0.89中生成project时候只会生成proguard.cfg文件，要想启用proguard代码混淆，我们只需要在app的build.gradle中添加下述配置：  

```
android {
    buildTypes {
        release {
            runProguard true
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }
}

```

而这个proguard-android.txt则在`{sdk.dir}/tools/proguard/proguard-android.txt`中，这时proguard混淆时默认的文件，不过我们的sdk目录下却没有。Google君终于帮我们找到了：[https://android.googlesource.com/platform/sdk/+/master/files/proguard-android.txt](https://android.googlesource.com/platform/sdk/+/master/files/proguard-android.txt)。于是简单粗暴的在tools/目录下新建proguard/proguard-android.txt文件。这时候android就可以使用默认文件进行混淆啦。
 
###Signing your application
Android需要所有的应用在安装之前被数字签名，Android通过签名凭证识别应用的作者信息等。在App的Debug模式活着Release模式中都可以进行应用签名，在调试模式中一般是android自动生成凭证给App签名，而在Release发布模式中则需要开发者自己生成凭证来给用户签名。  
我们主要关注在Release中怎么给自己的应用签名，签名之后的应用才可以发布到各市场中，在Release时给App签名：  

1. Create a keystore. Keystore是一个包含一些private key的二进制文件，你必须把这个文件放在一个安全独立的地方。
2. Create a private key. Private key 代表App作者的信息，例如一个开发者或者公司。
3. Build a project. 编译你的工程，这时会生成一个未签名的APP。
4. Sign your app. 用你的private key来生成你签名版本的APK。

这样的APK就能够发布到市场中供用户更新下载。  

签名既可以通过IDE直接操作，也可以通过android提供的工具在命令行中进行签名。在IDE中能够简单的生成一个keystore（需要说明的是不同的IDE生成的keystore文件格式是不一样的，主要有.jks, .keystore, 无后缀）。这些keystore在首次创建后下次就可以直接使用了。keystore包含的是一些软件作者的一些信息，也是你更新软件等的重要凭据。通过IDE签名的APK文件都经过了`jarsigner, zipalign`等操作，所以我们也不需要做额外的事情。  
如果通过命令行来进行签名，这样的好处是使用命令来自动部署，开发者可以把所有的步骤写在一个shell文件中，部署时只要简单的run这个命令就可以实现了，如果需要编译很多个针对不同市场的版本，通过命令的模式就能够大大节约发布APP的时间。  

更多详细信息请参考[http://developer.android.com/tools/publishing/app-signing.html](http://developer.android.com/tools/publishing/app-signing.html)  

