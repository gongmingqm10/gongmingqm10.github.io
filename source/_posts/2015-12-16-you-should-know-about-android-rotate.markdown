---
layout: post
title: "You should know about Android rotate"
date: 2015-12-16 15:39:43 +0800
description: Android屏幕旋转，Fragment生命周期, 状态保存
comments: true
categories: Android
published: false

---

在 Android 开发或面试过程中，屏幕旋转是一个容易让人忽视的知识点。在我之前经历的项目中，App 通常是为竖屏状态设置的，所以通常我们会对每个页面都设置竖屏方向，这时候我们不需要考虑旋转屏问题。但是最近项目中，我们的 App 是为平板设计的，而横竖屏旋转是属于客户的一个需求，当然平板上横竖屏的确比较常用。所以就借此机会研究了下 Android 横竖屏问题。

![Android 横竖屏](http://speakinbytes.com/wp-content/uploads/2014/01/portrait_landscape.png =500x)

横竖屏之所以需要引起开发者的注意，是因为 App 在横竖屏切换的过程中，页面会重绘，那么页面上已有的数据（比如登录页面已经输入的用户名）如何保存成为了一个问题。按照官方的推荐，Activity本身的确有处理旋转屏事件的函数，但是当一个页面中需要保存的数据很多的时候（比如很多 EditText），还是手工处理，就显得有些繁琐了。下面我们将循序渐进地探索 Android 屏幕旋转之最佳实践。

## Activity 旋转中的保存与恢复

在解决问题或者探索最佳实践的时候，我们可以从简单的问题入手，慢慢衍生至复杂的情形，最后再抽象出一些比较通用的解决方案。这一步我们单纯的探索一个 Activity（没有 Fragment）的数据保存与恢复。

### Activity 的生命周期

Activity 从创建到呈现在用户面前到消亡，有着自己完整的生命周期：

![Activity 生命周期](http://7xj9js.com1.z0.glb.clouddn.com/activity_lifecycle.png =500x)

旋转屏幕时，打印相关 Log 如下：

![Activity 旋转中的生命周期](http://7xj9js.com1.z0.glb.clouddn.com/activity_rotate_cycle.png =600x)

从 Log 可以看到，在屏幕旋转时，原来的 Activity 调用了 onDestroy，随后重新实例化了一个 MainActivity。重新实例化的 MainActivity 也会经历 “onCreate -> onStart -> onResume” 的生命周期。

### Activity 中窗口保存与恢复

为了进一步探索 Activity 在旋转屏过程中的数据保存及恢复的逻辑，我们构造了一个具有用户名和密码的登录界面。当用户在用户名中输入用户名时，这时候旋转屏幕，此时期望的操作应该是输入的用户名仍然能够保留：

![Activity Login 页面](http://7xj9js.com1.z0.glb.clouddn.com/activity_login.png =400x)

在不添加任何额外代码的情况下，我们可以看到输入框中的数据在旋转屏后仍然能够保留，这些控件基本状态的值是如何被保留的呢？

Activity 中方法 `protected void onSaveInstanceState(Bundle outState) {...} ` 主要做状态保存相关的处理，如果我们有需要特地保存的变量等，我们可以在 `onSaveInstanceState` 中保存，保存后的 bundle 以 outState Bundle 的格式保存。当 Activity 再次被初始化时，`onCreate(Bundle savedInstanceState)` 会将保存的 bundle 传递给 Activity 主页面，Activity 主页面接收到这些状态保存的数据后，能够根据保存中的控件的ID信息，状态数据等对页面进行自动的初始化。当我们转屏时，会主动触发 `onSaveInstanceState` 被调用。Log 打印如下：

![Activity onSaveInstanceState & onRestoreInstanceState](http://7xj9js.com1.z0.glb.clouddn.com/activity_onrestore_data.png =600x)

根据 Log 很明确看出在旋转屏之后，随着 onPause 的执行，`onSaveInstanceState` 也被执行。当 Activity 再次初始化时，`onCreate(Bundle savedInstanceState)` 会传递回一个非空的 savedInstanceState（而当 Activity 第一次初始化时此值为空），同时 `onRestoreInstanceState` 也会被调用，用来将保存的窗口状态信息重新应用：

生命周期：`onCreate` -> `onStart` -> `onResume` -> Running 转屏 -> `onPause` -> `onSaveInstanceState` -> `onStop` -> `onDestroy` -> `onCreate` -> `onStart` -> `onRestoreInstanceState` -> `onResume`;

![Android savedInstanceState](http://7xj9js.com1.z0.glb.clouddn.com/activity_savedInstanceState.png =600x)

通过对 `savedInstanceState` 的查看，发现 savedInstanceState 中包含有 Key 为 `android:viewHierarchyState` 的 bundle 数据，此 bundle 数据具体内容如下：

```
Bundle[{android:views={
		16908290=android.view.AbsSavedState$1@347440f8, 		2131492927=android.view.AbsSavedState$1@347440f8, 		2131492928=android.view.AbsSavedState$1@347440f8, 		2131492929=android.support.v7.widget.Toolbar$SavedState@391b69d1, 		2131492930=android.view.AbsSavedState$1@347440f8, 		2131492944=TextView.SavedState{138fec36 start=8 end=8 text=gongming}, 		2131492945=TextView.SavedState{16043737 start=0 end=0 text=}, 		2131492946=android.view.AbsSavedState$1@347440f8
	}, 
	android:focusedViewId=2131492944}
]
```
可以看到 username 的输入框 EditText，其 ID 以及当前的值都已经保存至 savedInstanceState 中。

那么，对于 Activity.java 的而言，最重要的保存数据和恢复数据的源代码如下：

```
    private static final String WINDOW_HIERARCHY_TAG = "android:viewHierarchyState";
    
    protected void onSaveInstanceState(Bundle outState) {
        outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
        Parcelable p = mFragments.saveAllState();
        if (p != null) {
            outState.putParcelable(FRAGMENTS_TAG, p);
        }
        getApplication().dispatchActivitySaveInstanceState(this, outState);
    }
    
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        if (mWindow != null) {
            Bundle windowState = savedInstanceState.getBundle(WINDOW_HIERARCHY_TAG);
            if (windowState != null) {
                mWindow.restoreHierarchyState(windowState);
            }
        }
    }

```
这就是为什么我们没有做过多的处理却可以让 App 在旋转屏幕时仍然自动保存恢复输入框中的文字。

### Activity 中数据保存与恢复

既然 Activity 本身对窗口（控件）的状态信息进行了保存及恢复处理。那么我们在屏幕切换时最应该关心的就是页面数据的保存与恢复。页面数据主要有两种：

* API 请求的数据：在横竖屏切换时对API数据进行保存及恢复能够防止 API 的重复调用；
* 页面中的状态值：在 Activity 运行过程中被改变的状态值，在恢复时需要手动保存及恢复，以便不影响状态值相关的页面逻辑；

在 Activity 中进行变量的存储及读取时，使用 `onSaveInstanceState` 进行数据的存储，使用 `onRestoreInstanceState` 进行数据的读取：

```
    private boolean shouldSaveData = false;
    
    private static final String KEY_SHOULD_SAVE_DATA = "save_data_key";

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        outState.putBoolean(KEY_SHOULD_SAVE_DATA, shouldSaveData);
        super.onSaveInstanceState(outState);

        Log.i(TAG, "@@Activity onSaveInstanceState@@" + this.toString());
    }

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        shouldSaveData = savedInstanceState.getBoolean(KEY_SHOULD_SAVE_DATA);
        Log.i(TAG, "@@Activity onRestoreInstanceState@@" + this.toString());
    }
```

### Activity 中弹框的状态保存及恢复

在 Activity 中经常会遇到 Dialog，甚至像下面具有输入框的 Dialog：

![Android Dialog](http://7xj9js.com1.z0.glb.clouddn.com/activity_signature_dialog.png =400x)

当用户点击 Login 按钮，弹出弹框需要用户签名，这时如果屏幕不小心进行了一次旋转，那么这个弹出的 Dialog 便消失，随之消失的还有用户输入了一半的输入框文字。Activity 在旋转过程中对于 Dialog 自身的生命周期进行很好的管理，如果为了达到更好的用户体验，转屏时也需要保存输入框状态，那么此处我们强烈推荐用户使用 [DialogFragment](http://developer.android.com/reference/android/app/DialogFragment.html) 代替 Dialog。

由于 Fragment 也具有生命周期，使用 DialogFragment 之后，我们结合 Activity 与 Fragment 的生命周期，查看整个过程经历了哪些流程。

![Frgament 生命周期](http://7xj9js.com1.z0.glb.clouddn.com/fragment_lifecycle.png =300x)

结合 DialogFragment 在 Activity 中旋转的重新初始化及数据恢复，我们可以看到执行顺序如下：

![Android Fragment 执行顺序](http://7xj9js.com1.z0.glb.clouddn.com/activity_fragment_rotate.png =500x)

在转屏时，Activity 和 Fragment 都会重新实例化，并且都通过 `onSaveInstanceState` 进行状态保存。值得注意的是不同于 Activity, Fragment 并没有 `onRestoreInstanceState` 方法，Fragment 的状态恢复在 `onActivityCreated` 方法中。查看 DialogFragment 源码，我们可以看到如下调用：

```
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
		...
        if (savedInstanceState != null) {
            Bundle dialogState = savedInstanceState.getBundle(SAVED_DIALOG_STATE_TAG);
            if (dialogState != null) {
                mDialog.onRestoreInstanceState(dialogState);
            }
        }
    }
    
    ...
    
    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        if (mDialog != null) {
            Bundle dialogState = mDialog.onSaveInstanceState();
            if (dialogState != null) {
                outState.putBundle(SAVED_DIALOG_STATE_TAG, dialogState);
            }
        }
		...
    }
    
```

由于 DialogFragment 其实就是展示一个 Dialog，而 DialogFragment 对 Dialog 的状态保存及恢复使得 Dialog 的状态得以保存。

## Fragment 状态的保存及恢复

