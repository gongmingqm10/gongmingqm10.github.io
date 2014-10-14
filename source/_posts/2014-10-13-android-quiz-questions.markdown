---
layout: post
title: "Android quiz questions"
date: 2014-10-13 17:19:22 +0800
comments: true
categories: android

---
关于Android的几个常见的问题，记录如下，小问题看到本质也可以了解很多原理性的东西。

---

**Q: Android中Activity的生命周期？**

A: onCreate() -> onStart() -> onResume() -> onPause() -> onStop() ->onDestroy().

---

**Q：Activity的onCreate()等函数必须Override吗？如果没有onCreate()，Activity能否运行？**

A: 答案显然是Override不是必须的，没有onCreate()的Activity还是可以运行的，那么究竟会运行成什么样子呢。  
实例中试一试，从MainActivity中打开没有Override任何Method的Activity，发现界面上会出现一个白底界面的Activity，说明我们的Activity最终还是运行了的。只不过界面上没有任何元素，因为我们没有指定contentView。  

---

**Q: onStop()等在生命周期中是否一定会被调用?**

A: onStart(), onResume(), onPause(), onStop()都可以不被调用。因为一旦在onCreate()中我们直接手动调用finish()，相当于告诉Activity结束，于是系统会直接调用onDestroy()来销毁当前的Activity。

---


**Q: onCreate()中的参数savedInstanceState意义是什么，会在什么情况下用到？**

A：按照官方的解释：
>      * @param savedInstanceState If the activity is being re-initialized after
     *     previously being shut down then this Bundle contains the data it most
     *     recently supplied in {@link #onSaveInstanceState}.  <b><i>Note: Otherwise it is null.</i></b>
     * 

savedInstanceState表示当Activity在前一个关闭再度初始化的时候会保存的信息。举例来讲，在我们的APP运行时，如果系统内存不足，系统会把Activity栈中一些比较古老的Activity给终结掉，此时系统会调用`onSaveInstanceState()`方法，此方法默认保存此Intent的一些信息，当这个Activity再次被调用时，这些保存的信息会传递到onCreate的savedInstanceState中。我们如果需要自定义的保存某些很重要的信息，可以复写`onSaveInstanceState()`方法把某些重要信息放到bundle中，也可以把那些信息写到本地文件中或者数据库中进行保存。方便下次进入时能够获取到。

---

**Q: Activity与Fragment的区别**

A: Activity相信大家都不陌生，其实可以简单的看着一个页面，跳转到另外一个页面一般就进入新的Activity。那么Android从API11开始使用Fragment的原因呢？其实主要是为了适配Android的多屏（特别是手机与平板）问题。一般情况下，如果按照普通的设计思路，我们会同时维护两份代码，并且UI上会有比较大的区别。对于快速升级的互联网产品而言，简直就是一个灾难。这时Google想了一个好的办法，在代码中引入Fragment，Fragment相当于一个module，我们所有的UI可以以模块的形式放到Fragment中。这时候的Fragment能够被多个Activity复用，我们的Activity此时做的事情就是组装这些Fragment。在平板中，我可以在ActivityA中放入FragmentA和FragmentB；在手机中，我则会在ActivityA中放入FragmentA，而在ActivityB中放入FragmentB。所以平板和手机的适配问题变成了简单的组装。从而我们的View不需要进行各种大的变动。这能够为开发者节省很多时间。  
具体参考：[Android Fragment Component](http://developer.android.com/guide/components/fragments.html)  

---

**Q: UI线程你知道多少？**

A：UI线程就是我们常说的主线程，UI线程负责着与用户的交互操作，会处理用户的触摸滑动点击等事件，并把这些事件分发给相应的组件来处理。如果我们尝试在UI线程中进行一个耗时的操作，那么我们的APP会出现ANR（Application not responsing）异常。在UI线程中直接进行耗时操作就会造成主线程阻塞，阻塞到达一定时间则会产生ANR。  
为了防止这种UI错误，我们进行的处理是将耗时操作放到另外一个线程中处理，耗时操作完成后，我们不能在非UI线程中进行UI的操作，这样是不安全的。因为我们可以借助Handler把更新的消息发给主线程，然后在主线程中更新UI操作。下面是几种耗时操作后更新UI线程的方法：    

* View.post(Runnable runnable)
* Activity.runOnUiThread(Runnable runnable)
* AsyncTask
* Hanlder + Thread

事实上前面三种都是使用java封装好的其他线程中操作UI线程的方法，这些方法都是为了方便开发人员而进行的一些处理。具体用法，大家可以Google之。








