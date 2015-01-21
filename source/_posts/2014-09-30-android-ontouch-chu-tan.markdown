---
layout: post
title: "Android onTouch() 初探"
date: 2014-09-30 17:28:56 +0800
comments: true
categories: [android]
description: Android touch 事件机制
---

Android的touch对于我来说是个既熟悉又陌生的话题，熟悉之处在于onTouch太常用了，从系统的自定义的ListView的滑动到我们自定义的可以滑动的View，onTouch直接与用户进行相关的Interation，所以onTouch无处不再。想像下如果Android某一天不能相应我们的touch事件，那我们现在的触屏手机基本废了，那我们的手机还得还原到以前的字母键手机的状态。学习总是得自己逼迫自己，没有博客的驱动我是怎样也不会来从头开始研究onTouch的作用机制的，博客当作一个自我学习的过程，坚持再坚持。

废话不说，步入正题，从用户开始，触屏事件被屏幕传感器截获，截获后会将该触摸数据传到我们的View上面，然后View再进行相应的处理。从用户触摸到数据被View感知都是有Android底层完成的，我们这里关心的只是Android的view会如何响应这些触摸行为呢？了解了触摸行为相关的原理才可以更好的利用这些特性从而实现我们自定义的各种交互生动的组件。

下面我们会循序渐进，逐步研究一些与Touch相关的特性，最后我们会通过几个小demo来展示怎么通过重写onTouch()自定义一些实用的组件。

###1. boolean onTouch() 返回值的意义
直接贴出View.OnTouchListener的源码：

```
    public interface OnTouchListener {
        /**
         * Called when a touch event is dispatched to a view. This allows listeners to
         * get a chance to respond before the target view.
         *
         * @param v The view the touch event has been dispatched to.
         * @param event The MotionEvent object containing full information about
         *        the event.
         * @return True if the listener has consumed the event, false otherwise.
         */
        boolean onTouch(View v, MotionEvent event);
    }
```
这是官方给出的定义，对于return返回值的意义，解释是`return True if the listener has consumed the event, false otherwise`，如果返回True，表示View消费了这次事件，否则的话，表示View并没有消费本次Touch事件。  
<img src="/images/touch-simple.png" alt="main_activity_layout.xml"/>  
在XML中，我们声明了A，B，C 三个Layout，其中C包含B，而B包含A，对于ABC三个View，我们都为其指定相应的触摸事件，然后再观察触摸事件是如何被响应并传递的。  
MainActivty中定义如下：    
<img src="/images/touch-simple-main.png" alt="MainActivity.java"/>    
在onTouch()方法中，我们`return false`，说明当前的View并没有消化触摸事件，它会将触摸事件继续`向上`传递，所谓`向上`指的是View会向它的父元素传递，父元素又会根据其定义的onTouch事件继续将事件传递。触摸A元素，产生的Log如下：  
  
```
10-01 16:20:44.640    1471-1471/org.gongming.uikit I/gongmingqm10﹕ --onTouch()--son
10-01 16:20:44.640    1471-1471/org.gongming.uikit I/gongmingqm10﹕ --onTouch()--parent
10-01 16:20:44.640    1471-1471/org.gongming.uikit I/gongmingqm10﹕ --onTouch()--grantParent
```
此时是`onTouch() return false` 的情况，触摸A的时候，首先触摸事件会有A处理，A处理完之后，`return false`导致触摸事件向A的父容器B传递，同时B又会继续向C传递，直至结束。  
当我们把`onTouch()`返回结果改为`return true`时，再次触摸A，这时的Log如下：  

```
10-01 16:32:29.008    1612-1612/org.gongming.uikit I/gongmingqm10﹕ --onTouch()--son
10-01 16:32:29.016    1612-1612/org.gongming.uikit I/gongmingqm10﹕ --onTouch()--son
```
这时的`return true`表示A自己消化了触摸事件，所以触摸事件不会向上传播。通过`onTouch`中的MotionEvent可以得到当前触摸的x和y坐标，借此实现一些复杂的功能。  
MotionEvent除了提供触摸的坐标外，还通过`MotionEvent.getEventAction()`判断当前触摸的类型，主要分为`ACTION_POINTER_DOWN` `ACTION_POINTER_UP` `ACTION_DOWN` `ACTION_MOVE` `ACTION_UP` `ACTION-CANCEL`，以下是MotionEvent中对这些Action的定义说明：
  
```
    /**
     * Constant for {@link #getActionMasked}: A pressed gesture has started, the
     * motion contains the initial starting location.
     * <p>
     * This is also a good time to check the button state to distinguish
     * secondary and tertiary button clicks and handle them appropriately.
     * Use {@link #getButtonState} to retrieve the button state.
     * </p>
     */
    public static final int ACTION_DOWN             = 0;
    
    /**
     * Constant for {@link #getActionMasked}: A pressed gesture has finished, the
     * motion contains the final release location as well as any intermediate
     * points since the last down or move event.
     */
    public static final int ACTION_UP               = 1;
    
    /**
     * Constant for {@link #getActionMasked}: A change has happened during a
     * press gesture (between {@link #ACTION_DOWN} and {@link #ACTION_UP}).
     * The motion contains the most recent point, as well as any intermediate
     * points since the last down or move event.
     */
    public static final int ACTION_MOVE             = 2;
    
    /**
     * Constant for {@link #getActionMasked}: The current gesture has been aborted.
     * You will not receive any more points in it.  You should treat this as
     * an up event, but not perform any action that you normally would.
     */
    public static final int ACTION_CANCEL           = 3;
    
    /**
     * Constant for {@link #getActionMasked}: A movement has happened outside of the
     * normal bounds of the UI element.  This does not provide a full gesture,
     * but only the initial location of the movement/touch.
     */
    public static final int ACTION_OUTSIDE          = 4;

    /**
     * Constant for {@link #getActionMasked}: A non-primary pointer has gone down.
     * <p>
     * Use {@link #getActionIndex} to retrieve the index of the pointer that changed.
     * </p><p>
     * The index is encoded in the {@link #ACTION_POINTER_INDEX_MASK} bits of the
     * unmasked action returned by {@link #getAction}.
     * </p>
     */
    public static final int ACTION_POINTER_DOWN     = 5;
    
    /**
     * Constant for {@link #getActionMasked}: A non-primary pointer has gone up.
     * <p>
     * Use {@link #getActionIndex} to retrieve the index of the pointer that changed.
     * </p><p>
     * The index is encoded in the {@link #ACTION_POINTER_INDEX_MASK} bits of the
     * unmasked action returned by {@link #getAction}.
     * </p>
     */
    public static final int ACTION_POINTER_UP       = 6;

    /**
     * Constant for {@link #getActionMasked}: A change happened but the pointer
     * is not down (unlike {@link #ACTION_MOVE}).  The motion contains the most
     * recent point, as well as any intermediate points since the last
     * hover move event.
     * <p>
     * This action is always delivered to the window or view under the pointer.
     * </p><p>
     * This action is not a touch event so it is delivered to
     * {@link View#onGenericMotionEvent(MotionEvent)} rather than
     * {@link View#onTouchEvent(MotionEvent)}.
     * </p>
     */
    public static final int ACTION_HOVER_MOVE       = 7;

    /**
     * Constant for {@link #getActionMasked}: The motion event contains relative
     * vertical and/or horizontal scroll offsets.  Use {@link #getAxisValue(int)}
     * to retrieve the information from {@link #AXIS_VSCROLL} and {@link #AXIS_HSCROLL}.
     * The pointer may or may not be down when this event is dispatched.
     * <p>
     * This action is always delivered to the window or view under the pointer, which
     * may not be the window or view currently touched.
     * </p><p>
     * This action is not a touch event so it is delivered to
     * {@link View#onGenericMotionEvent(MotionEvent)} rather than
     * {@link View#onTouchEvent(MotionEvent)}.
     * </p>
     */
    public static final int ACTION_SCROLL           = 8;

```
通过以下语句，可以可以判断当前触摸处于何种状态，进而进行相关的操作：  

```
switch (event.getActionMasked()) {
	case MotionEvent.ACTION_DOWN:
		...
		break;
	case MotionEvent.ACTION_MOVE:
		...
		break;
	...
}

```

但是实际上事情到这里远没有终止，`onTouch`中我们可以通过`getAction`获取响应的触摸行为。但是当我们运行demo的时候发现，默认情况下只有ACTION_DOWN事件会响应，`ACTION_MOVE`竟然不响应，原来是onTouch()默认条件下`return false`，意味着View本身调用一次`ACTION_DOWN`函数后，就将触摸事件继续向其父类View继续传递，所以`ACTION_MOVE`不能响应。如果要获得响应，那我们需要告诉系统由`我`来处理触摸事件，`return true`就是View对外界释放的信号，此时事件就可以被View继续处理。

###2. boolean onInterceptTouchEvent(MotionEvent ev)

除了onTouch外，开发中我们还可以经常遇到onInterceptTouchEvent，从字面意思来看是拦截触摸事件的作用，为了更权威，摘录官方解释如下：

```
For as long as you return false from this function, each following
     * event (up to and including the final up) will be delivered first here
     * and then to the target's onTouchEvent().
     *  If you return true from here, you will not receive any
     * following events: the target view will receive the same event but
     * with the action {@link MotionEvent#ACTION_CANCEL}, and all further
     * events will be delivered to your onTouchEvent() method and no longer
     * appear here.
```
查看源代码可以看到，这个方法被定义在ViewGroup中，我们可以猜到这个方法可能会和View的子View有关系。结合官方解释，如果onInterceptTouchEvent()返回为true，那么触摸事件不会被分发到子类中。

###3. View.dispatchTouchEvent()

dispatchTouchEvent()是View类中的方法，用于将当前View接收到的触摸事件进行分发或者进行自身的相应处理。触摸事件从用户点击屏幕时就开始被封装为TouchEvent开始了自己的旅程：

1. Activity.dispatchTouchEvent(), Activity通过系统本身的传感器接收到封装为MotionEvent的触摸事件，开始通过dispatchTouchEvent()把事件向下分发给Root ViewGroup。

2. ViewGroup.dispatchTouchEvent(), ViewGroup从Activity中拿到MotionEvent后，首先通过自己的dispatchTouchEvent再次向自己的子View分发。在dispatchTouchEvent函数中，ViewGroup会调用自己的onInterceptTouchEvent，如果intercept返回true，那么ViewGroup就停止对子View进行事件分发，一旦有些子View还有一些处理中的触摸事件，ViewGroup会发送ACTION_CANCEL事件给子View，然后ViewGroup判断自己是否有onTouchListener, 有的话就执行之，没有的话就会执行自身的onTouchEvent()。如果intercept返回false，ViewGrou将会根据触摸的位置和子View的位置判断是否将MotionEvent分发给子View.

3. 在触摸点范围内的子View会根据继续进行类似2的处理。如果子View也有自己的children，则继续按照步骤2中的逻辑进行分发。这里假设子View没有children。如果自己的onTouchListener存在的话，则首先会执行listener中的onTouch()方法或者直接去执行自身的onTouchEvent()方法。 执行这些方法如果返回true，则表示View已经消耗了这个触摸事件，事件传递结束。如果onTouch()返回false，则冒泡向上传递MotionEvent，直到Activity.onTouchEvent()，是触摸事件的终点。

###More. SimpleOnGestureListener  
`onTouch是`View中所有触摸事件的入口，通过上面的了解我们可以看到MotionEvent中通过提供触摸的位置和触摸的类型方便我们进行各种判断，但是我们的真实需求往往比这复杂，我们有时需要自己处理双击事件，长按事件，滑动事件等等。如果这些由我们自己处理，那我们需要在onTouch()中进行一些逻辑判断，这无形中增加了我们的开发难度和代码量。好在Android通过GestureDetector提供了更好的支持。对于一些常规的事件，我们可以直接通过GestureDetector捕获，从而降低了我们的开发成本，开发人员可以把更多的时间放在业务上。通过`GestureDetector.SimpleOnGestureListener`的实现，我们能够复写一些基础的事件，从而完成我们的一些业务逻辑。   
<img src="/images/touch-gesture-1.png" alt="SimpleOnGestureListener"/>   
通过以上代码，我们主要可以看到在`View.setOnTouchListener`中将MotionEvent委托给GestureDetector来处理，而自身返回的true或者false将决定父容器能否响应触摸事件。  

###Sample. SwipeLayout

了解了touch事件的机制以及流程，我们能够更加灵活的自定义与手势操作结合起来的控件。在这个例子中，我们自定义SwipeLayout，实现竖直翻动效果。

代码源地址为：[SwipeLayout源码](https://github.com/gongmingqm10/AndroidUikit/blob/master/library/src/main/java/org/gongming/common/SwipeLayout.java)

###Reference
1. [Android: Difference between onInterceptTouchEvent and dispatchTouchEvent?](http://stackoverflow.com/questions/9586032/android-difference-between-onintercepttouchevent-and-dispatchtouchevent)
2. [Managing Touch Events in a ViewGroup](http://developer.android.com/training/gestures/viewgroup.html)
3. [How Android Handles Touches](http://www.youtube.com/watch?v=EZAoJU-nUyI)




