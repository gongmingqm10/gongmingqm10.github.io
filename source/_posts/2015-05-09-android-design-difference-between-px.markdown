---
layout: post
title: "Android Design - Difference between px sp and dp"
date: 2015-05-09 11:32:59 +0800
description: Android dp, sp, px difference
comments: true
categories: Android

---

Android自兴起以来，开发者不断增加，与此同时市场上参差不齐的设备也不断增加，随之而来的则是开发者需要适配众多机型而带来的困扰与抱怨。并且随着不同厂家定制化不同的ROM，Android设备的碎片化现象日益严重。这篇文章主要讲述如何设计中严格遵从设计图标准的Android页面。

## 1. px, dp, sp区别

###什么是px

px就是我们通常所说的像素的单位，在css里面，我们用px比较多。px主要指用户在屏幕上看到的事物的实际大小单位。

###什么是dp

在看手机或者电脑参数时，我们经常说到一个关键词，屏幕分辨率(resolution)，分辨率被表示成每一个方向上的像素数量，比如800x480分辨率

<!-- more -->

{% img /images/retina_one_screen.jpg Mac retina resolution %}

上图中，我们可以看到15寸的retina pro分辨率是2880x1800，而60寸的HDTV分辨率却只有1920x1080。可见如果60寸的HDTV也要达到Retina的显示效果，其分辨率大概至少为11520x7200。

既然已经有了分辨率的概念，那么我们就可以了解 dpi(dot per inch)的概念了。dpi本意是每英寸点的数量，但是也经常和ppi(pixel per inch)混用。指每英寸的像素数量，也被我们称为屏幕密度，dpi越大，图像从视觉上看起来越清晰。

Google官方对dp的解释如下：

> A virtual pixel unit that you should use when defining UI layout, to express layout dimensions or position in a density-independent way.
The density-independent pixel is equivalent to one physical pixel on a 160 dpi screen, which is the baseline density assumed by the system for a "medium" density screen. At runtime, the system transparently handles any scaling of the dp units, as necessary, based on the actual density of the screen in use. The conversion of dp units to screen pixels is simple: px = dp * (dpi / 160). For example, on a 240 dpi screen, 1 dp equals 1.5 physical pixels. You should always use dp units when defining your application's UI, to ensure proper display of your UI on screens with different densities.

有了密度的概念，我们可以更好的理解dp(density independent pixel)了。dp是Android中用来定义UI布局中表达元素尺寸或位置的一个虚拟的像素单位，dp的存在是为了页面元素位置不受屏幕密度所影响。以设计师设计一个APP页面为例，设计师希望用户视觉上看起来如此，也就是设计师设计时通常以px作为基础单位。而在其他不同屏幕密度的设备中，也希望能够进行一定的伸缩，类似于维持一个相对一致的百分比。所以开发者通常需要以dp为单位，以适应不同密度的屏幕。

所以dp和px之间有一套换算标准：`px = dp * (dpi / 160)`。假设在dpi为320的屏幕上，1dp = 2px。

### 什么是sp

sp是Android中专业为字体而设置的单位。使用sp作为字体单位不仅能够使字体大小受屏幕密度影响，并且能够使字体受用户系统设置的影响。Android提供字体大小的设置选项。一旦使用sp作为字体，根据用户设置字体的不同，App中的字体大小也会显示不同。但是在正常字体设置下，1dp ＝ 1sp。

## 2. 设计师视角

Android的多屏适配存在这么多的尺寸，自然需要设计师和工程师之间找到沟通的桥梁，设计师如何规范设计才能够更加符合Android的设计标准呢。设计师与工程师对于页面的尺寸需要有一个共识性的标准，测试人员才可能进行更有效的测试，才不致于因为不同机型显示不同效果而引起不必要的工作量。

首先我们需要选择一个相对通用的屏幕尺寸，可以参考现有的一些主流机型的尺寸。为了便于计算，我们选择Nexus 4作为设计图的原版标准。那么所有的页面都是基于Nexus4而产生的。Nexus4尺寸如下:

`主屏尺寸4.7，主屏分辨率：1280x768像素，屏幕像素密度320ppi。`

一般来说，我们选择市场上最常见的机型来设计，这里我们为了计算方便，选择320ppi的机型，设计图中尺寸将是标准图中的2倍。假设在设计图中，页面内边距为32px, 按钮的高度为96px, icon大小为48x48。

通过`px＝dp*(dpi/160)`的标准，设计师给开发者标注图时可标注页边距为16dp，按钮高度为48dp，icon则分别出四套mdpi, hdpi, xhdpi, xxhdpi标准。

## 3. 开发者视角

### margin, padding, height, width

从开发者角度来看，对于控件的尺寸，高度等，只需要按照转化后的dp设置即可。

```
android:layout_padding="16dp"

android:layout_height="48dp"

```

以dp为单位能够保证不同分辨率屏幕上显示不同大小的字体。并且能够保持相对大小，这样也更符合设计的初衷。

### 图片资源

对于png制作出来的图片，一般大小固定，没办法像尺寸一样自动的伸缩。所以Android会存在至少4个资源文件夹。当App运行过程中，系统能够根据当前设备的屏幕密度，自动选择使用哪种尺寸的图片资源。

ldpi | mdpi | hdpi | xhdpi | xxhdpi | xxxhdpi
------------ | ------------- | ------------ | ------------ | ------------
0.75 | 1  | 1.5 | 2 | 3 | 4
120 | 160 | 240 | 320 | 480 | 640
18x18 | 24x24 | 36x36 | 48x48 | 72x72 | 96x96


## 4. 更多

附上Android主流机型：

Android：主流机型主要为 480x800, 480x854, 540x960, 720x1280, 800x1280 这五种。
（非主流机型还包括：240x320, 320x480, 640x960 这三种，其中两种都与 iPhone 一致。）
iOS: 主流机型主要为 320x480, 640x960, 640x1136, 1024x768, 2048x1536, 这五种。
WP：主流机型主要为 480x800，720x1280, 768x1280 这三种

Photoshop制图时，字体大小通常是pt这个单位。pt是长度单位， 1pt = 1/72英寸， px = pt * dpi/72。

另附上豆瓣关于这些单位更[具体的阐释](http://www.douban.com/note/155032221/)。
