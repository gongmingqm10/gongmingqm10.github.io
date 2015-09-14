---
layout: post
title: "Material Design Your Android App"
date: 2015-06-25 15:22:27 +0800
description:
comments: true
categories: [Android, Design]

---

14年6月26日的Google I/O大会上，Google推出了一门全新的设计语言Material Design。Material Design在随后的一年里被逐渐应用到Android, Web等平台中。Material Design的推出意味着Google对移动端和网页端设计的整合。固然没有引起轩然大波，但是Material Design却让Android开发者看到了一点福音。

虽然Android系统广受诟病，越来越多的用户投奔苹果的iOS系统。然而这阻挡不了Android的不断改进。14年Google I/O, Google推出了Android的5.0版本，代号Lollipop。5.0版本在设计上采用全新的Material Design，虽然这次升级对ROM生产商来说是道“过不去的坎儿”，但是对开发者群体而言却是一次很不错的升级。因为开发者可以凭借有些的设计资源打造出更好看的App。

<!-- more -->

##关于Material Design

> Material design is a comprehensive guide for visual, motion, and interaction design across platforms and devices.

这是Google官方给的解释， Material Design 中文直译为 “物质的设计”。可以简单理解为画面中的所有元素都可以看作是真实的物质，物质存在于空间中，并且存在光、影、运动等特征。所以Material Design的众多原则都是基于Material这个概念而提出的。

###Material Design基本原则

Material Design作为设计与交互的一部分，由设计师来阐述更为专业。以下只是抛砖引玉的介绍，也许身边的设计师朋友会更专业的见解。

####Material is the metaphor

> A material metaphor is the unifying theory of a rationalized space and a system of motion

举例来说，在Android Lollipop系统中，当用户点击操作时，界面会反馈出水纹般的涟漪向周边散去，作为对用户点击操作的直接反馈。Material的设计都是基于现实，并在现实世界的基础上予以创新的。创新之处表现在元素之间整理运动的和谐。即使是在现实世界进行创新，最重要的是要Material Design不会破坏物理世界的规则。

####Bold, graphic, intentional

> The foundational elements of print-based design—typography, grids, space, scale, color, and use of imagery—guide visual treatments.

设计中的基本元素网格，空间，比例，字体，颜色等的结合能够给用户带来更好的体验，从而产生更大的价值。Material Design给我们指定了一些常见的搭配组合，比如颜色的组合、App中不同部分字体大小，组件之间的边距等。

####Motion provides meaning

> Motion respects and reinforces the user as the prime mover. Primary user actions are inflection points that initiate motion, transforming the whole design.

动画通常是App中最常见的交互，在Material Design出来之前，大部分App是很少有动画的。因为动画从设计到最后被用户使用，需要动画的设计以及开发，有时还会受限于平台。Material Design在动画方面有所加强，官方推出了许多默认的动作，页面各个元素之间的相互运动等。期望通过元素的运动向用户传达更为核心的价值和功能。

###为什么我们需要Material Design

设计主要是为了提升用户体验，辅助产品功能，起到锦上添花的效果。Google推出Material Design也许希望对所有的Google产品进行设计上的统一化。

从Android平台来看，以前的Google Apps很难说有什么风格，其他厂商开发的App也是风格各异。Google希望通过Lollipop对App统一风格，所以在5.0＋平台上的Gmail, Youtube, Calender等都进行了风格的统一化，借此引导更多的设计师和开发者开发出更“接地气”的App。

Material Design强调跨平台设计，如果你遵从Material Design，你会发现将无论是宽屏平板设备、还是手机，你会发现你的应用可以很智能的进行展示。这将极大减轻适配平板带来的工作量。是不是有点类似于Web上的响应式设计呢？

当然选择是自由的，如果你觉得你的App不需要Material Design，那么TA真的不需要。

##如何Material Design App

据个人观察，目前国内市场上大部分的App没有Material Design的概念。原因不得而知。但是遵循Material Design的应用在国外却比较常见。比如我个人常用的雅虎天气。

那么如何空手开发Material Design 应用呢？假定读到这里的你是个开发人员：

### 1.合适的SDK版本和相关兼容包

build.gradle

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "net.gongmingqm10.training"
        minSdkVersion 16
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
   	...
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
	//required
    compile 'com.android.support:appcompat-v7:22.2.0'
    //optional for RecyclerView
    compile 'com.android.support:recyclerview-v7:22.2.0'
    //optional for GridLayout
    compile 'com.android.support:gridlayout-v7:22.2.0'
    //optional for CardView
    compile 'com.android.support:cardview-v7:22.2.0'
    //optional for some useful libraries
    compile 'com.android.support:design:22.2.0'
}

```

###2.使用 Material theme

values-v21/styles.xml

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Main theme colors -->
        <!--   your app branding color for the app bar -->
        <item name="colorPrimary">@color/color_primary</item>
        <!--   darker variant for the status bar and contextual app bars -->
        <item name="colorPrimaryDark">@color/color_primary_dark</item>
        <!--   theme UI controls like checkboxes and text fields -->
        <item name="colorAccent">@color/color_accent</item>

    </style>
</resources>


```

values/styles.xml

```
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <!-- Support library compatibility -->

        <item name="actionBarStyle">@style/MyActionBar</item>
    </style>

    <style name="AppTheme.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>

    <!-- ActionBar styles -->
    <style name="MyActionBar"
        parent="@style/Widget.AppCompat.Light.ActionBar.Solid.Inverse">
        <item name="android:background">@color/color_primary</item>
        <item name="android:titleTextStyle">@style/MyActionBarTitleText</item>

        <!-- Support library compatibility -->
        <item name="background">@color/color_primary</item>
        <item name="titleTextStyle">@style/MyActionBarTitleText</item>
    </style>

    <style name="MyActionBarTitleText" parent="TextAppearance.AppCompat.Widget.ActionBar.Title">
        <item name="android:textColor">@color/white</item>
    </style>

</resources>

```

使用你定义的AppTheme

```
    <application
        android:name=".TrainingApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
		...
	</application>
```

###3.使用专为Material Design打造的Android控件

你可以参考[Google Training Material Design](http://developer.android.com/training/material/index.html)获取更多的Material Design组件的使用方法。

####Lists and Cards

#####RecyclerView
引入RecyclerView到工程中：

```
compile 'com.android.support:recyclerview-v7:22.2.0'
```
RecyclerView控件是ListView的更高效更灵活的版本。可以看作是对ListView更好的封装：

* 解决ListView多列显示的难题；
* 不用担心开发人员没有复用View，因为它会强迫你这样做；
* 更方便的动画支持，移除中间某一项时，可以设置后面几项动画移动;

#####CardView

```
compile 'com.android.support:cardview-v7:22.2.0'
```
卡片式布局，你的界面就是有导航和卡片组成的。卡片能够设置圆角和阴影。

####Design support library

使用Design support包含的空间，需要引入Design support库：

```
compile 'com.android.support:design:22.2.0'
```

Design support提供了一些很常见很实用的控件。

#####Navigation View

Material Design推荐使用DrawerLayout抽屉式布局显示导航，所以Android一直没有为底部导航推出相应的控件。按照Google最新的设计，所有的导航都会被设置为抽屉显示，点击左上角菜单可以抽屉式弹出菜单项。

NavigationView就是和DrawerLayout搭配使用的神器。使用NavigationView只需要提供icon和title就可以生成专业的侧边导航。

#####Floating Action Button

Material Design引入了z轴，这个控件可以轻松制作出有阴影的悬浮圆形按钮。

#####Snackbar

底部栏消息提示控件，显示显示不再只有Toast和AlertDialog，还有Snackbar。

#####Tabs

在TabLayout出来之前，开发者通常手写Tab控件，并且需要手动控制UI。并且设计出来的Tab效果很一般。Google在2014年的iosched工程中推出了[SlidingTabLayout](https://github.com/google/iosched/blob/master/android/src/main/java/com/google/samples/apps/iosched/ui/widget/SlidingTabLayout.java)，这个组件后来也在众多项目中被开发者用到。但是开发者还是得添加额外的代码来控制UI。

TabLayout帮助开发者创建符合界面风格的Tab，并且简单易用。

#####CoordinatorLayout & Collapse Toolbar

更加优美的导航，导航可以相应页面滚动，也产生相应的滚动效果。可以参考Android 5.0版本默认的联系人应用。

##结语

目前 Material Design 在国内潜伏了一年，也没引起太大的市场反馈。互联网泡沫下，决策者的心思都花在了如何推广如何吸引第一批用户上了，难以负担App重新设计带来的时间成本，更不行因为重设计造成现有用户的流失。

当然，如果某一天所有的国内手机都可以升级为5.0系统，是不是会有许多软件开发者考虑对App做Material Design呢？Android系统深度定制化依然成为5.0难以在国产手机中推广的重要阻碍。

Material Design作为Android App设计风格的趋势，尝试在你的App中使用新的风格，也许可以给你的产品带来不一样的体验。


**参考**

[Google Materil Design](https://www.google.com/design/spec/material-design/introduction.html)

[Material Design for Developers](http://developer.android.com/training/material/index.html)

[Android Design Support Library](http://android-developers.blogspot.sg/2015/05/android-design-support-library.html)
