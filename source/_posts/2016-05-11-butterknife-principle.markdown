---
layout: post
title: "ButterKnife Principle"
date: 2016-05-11 15:15:25 +0800
description: ButterKnife原理
comments: true
categories: Android 

---

![ButterKnife 原理](http://upload-images.jianshu.io/upload_images/576124-831ee4dbd091aa30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/200)

最近在研究 Java 中的 Annotation，初衷是先了解注解，然后再了解下 Android 开发中常用框架的原理。经历的 Android 项目中，使用注解比较多的是：[ButterKnife](https://github.com/JakeWharton/butterknife), [Roboguice](https://github.com/roboguice/roboguice), [Dagger](https://github.com/roboguice/roboguice) 。ButterKnife 主要是 View Binding, 而 Roboguice 和 Dagger 则主要是做 DI(Dependency Injection)。本文主要研究 ButterKnife 的原理。

开始之前，还是应该向 ButterKnife 的作者 Jake Wharton 大神以及开源社区致敬。

<!-- More -->

## ButterKnife 初瞰

ButterKnife 到底是什么？看看官方的解释：
> Bind Android views and callbacks to fields and methods.

通过 ButterKnife [官方文档](http://jakewharton.github.io/butterknife/)，我们可以很快速的使用它。并且可以看到 ButterKnife 主要是实例化 View 或者给某个 View 添加各种事件 Listenters。

![ButterKnife View Binding](http://upload-images.jianshu.io/upload_images/576124-2be2b864707172b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

所以使用 ButterKnife 主要有几步：

1.Gradle 中引入 ButterKnife 框架：

```
compile 'com.jakewharton:butterknife:8.0.1'
```
2.ButterKnife 初始化:

```
@Override
protected void onCreate(Bundle savedInstanceState) {          
    super.onCreate(savedInstanceState);   
    setContentView(getLayoutResourceId());    
    ButterKnife.bind(this);
}}
```
3.声明 Field/Method

```
@Bind(R.id.btn_home)ImageView homeBtn;
@Bind(R.id.btn_explore)ImageView exploreBtn;
@OnClick(R.id.btn_home) void clickHomeBtn() {      
    // Handle the home btn click
}
```

## Java Annotation 

如果你对注解了解比较少，在进一步了解 ButterKnife 之前，有必要了解一下 Java Annotation 的基础知识。可以参考如下两份资料：

1. Mkyong [Java Custom Annotations](http://www.mkyong.com/java/java-custom-annotations-example/)
2. Trenia [Java Annotations](http://trinea.github.io/download/pdf/android/java-annotation.pdf)

通过上面的例子以及讲解，我们可以知道如何自定义注解。自定义注解我们需要知道两个基础的元注解`@Retention` 和 `@Target`.

### 元注解 @Retention

`@Retention` 保留时间：有三类值可以选择，`SOURCE`  `RUNTIME` `CLASS`。 

`SOURCE` 源码时保留：使用此类的注解多为标记注解，比如 `@Override`、`@Deprecated`、`@SuppressWarnings` 等。

`RUNTIME` 运行时保留：程序在运行过程中，使用这些 Annotation, 比如我们常用的 `@Test`。

`CLASS` 编译时保留：Java 文件在编译时由 apt 自动解析，需要自定义类继承自 AbstractProcessor 并重写 Process 函数。比如 ButterKnife 中使用的  `@BindView`, `@OnClick` 等就是声明为 CLASS 的。

### 元注解 @Target

Target 表示注解可以用来修饰哪些元素。可选值包括 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER 等。

## ButterKnifeProcessor

ButterKnife 中所有的注解都使用 Retention 为 CLASS 保留。所以在 ButterKnife 中，有个很重要的 ButterKnifeProcessor。当 java 文件进行编译时，ButterKnifeProcessor 的 process() 方法被调用，生成相关的 ViewBinder 类，用于将 View 或者 Listener 进行绑定。

ButterKnifeProcessor 的关键逻辑如下：

![ButterKnifeProcessor.process()](http://upload-images.jianshu.io/upload_images/576124-0fa0624d662090af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

process() 会找到所有 ButterKnife 相关的 annotation，并将这些相应的注解生成为 ViewBinder 类，一个 ViewBinder 示例如下：

![Screen Shot 2016-05-11 at 2.52.20 PM.png](http://upload-images.jianshu.io/upload_images/576124-a9f0fdf997e95ed8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

ViewBinder 通过 findViewById 实例化 Activity 中页面上的组件，并且能够设置各控件的点击时间，滑动事件等。

> 为什么 ButterKnife Annotation 的字段或者方法不能声明为 private?

bind() 方法中，通过 activity.homeBtn 对 View 赋值。因此所有 ButterKnife annotation 标记的 field 或者 method 都不能声明为 private, 因为 private 没办法在 ViewBinder 中直接访问。由此可见：

> ButterKnife 并不是依靠反射实现 View Injection 的！

## ButterKnife.bind(this)

在 ButterKnife 工作之前，我们一定得在设置 View  后调用 `ButterKnife.bind(Activity)` ，这样才能使 ButterKnife View 注入成功。那么 ButterKnife.bind(Activity) 是如何工作的呢？

![ButterKnife.bind(this)](http://upload-images.jianshu.io/upload_images/576124-dce842ddf87bb891.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

App 在打包时，ButterKnife Processor 就为所有使用 ButterKnife 注解的 Activity 生成 ViewBinder()。通过上述代码可以看到，ButterKnife 会尝试实例化当前 Activity 所关联的所有 ViewBinder 类。实例化之后再调用 `ViewBinder.bind(...)` 方法，bind() 方法是在 App 运行过程中才会被调用的。于是当前 Avtivity 的 View 被实例化赋值，如果有点击事件的注解，也会绑定相应的事件。

## 结语

ButterKnife 主要是做 View 注入的，使用起来比较简便。当然如果你想做一些 依赖注入，比如 Android MVP 架构中 Activity 与 Presenter 的依赖。你可能需要借助一些其他的库， 比如 Dagger。研究下 ButterKnife 的原理，才发现这个库做得如此的睿智。