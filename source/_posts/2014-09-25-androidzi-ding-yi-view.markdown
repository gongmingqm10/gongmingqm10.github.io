---
layout: post
title: "Android自定义View"
date: 2014-09-25 23:22:11 +0800
comments: true
categories: android
description: Android自定义View Android自定义控件

---
Android开发中经常用到各种各样的View，有时需要自定义View来满足当前的需求。这些自定义View主要是复写View绘制时的一些方法，从而产生新的View供项目中使用。  
## View的绘制流程  
自定义控件从最基础的View开始，View有几个重要的函数：`onMeasure()`, `onLayout()`, `onDraw()`，与触摸动作相关的还有`onTouchEvent()`，View也和Activity一样具有一定的生命周期，从View被创建开始到创建完成，主要经历了 `onMeasure` `onLayout` `onDraw()` 等过程，这些过程都是一步步完成的。也代表着View从声明到被用户看到的具体步骤。通过对这些中间步骤的了解与`Override`，我们可以创造出一些特殊的View。

```

	public class CustomView extends View {

    private static final String TAG = "gongmingqm10";

    public CustomView(Context context) {
        super(context);
    }

    public CustomView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Log.i(TAG, "--onDraw--");
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        Log.i(TAG, "--onLayout--" + left + " - " + top + " - " + right + " - " + bottom);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        Log.i(TAG, "--onMeasure--" + widthMeasureSpec + " - " + heightMeasureSpec);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.i(TAG, "--onTouchEvent--" + event.getX() + " - " + event.getY());
        return super.onTouchEvent(event);
    }
	}

```

在布局文件`main.xml`文件中这样使用自定义的`CustomView`


```

	<?xml version="1.0" encoding="utf-8"?>

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	android:orientation="vertical" android:layout_width="match_parent"
   		android:layout_height="match_parent">
	    <org.gongming.common.swipelist.CustomView
    	    android:layout_width="match_parent"
        	android:layout_height="600px"
	        android:background="#333666"
    	    />
	</FrameLayout>

```
Logcat输出结果为

	09-25 15:55:29.876    1535-1535/org.gongming.uikit I/gongmingqm10﹕ --onMeasure--1073742904 	- 1073742424
	09-25 15:55:29.876    1535-1535/org.gongming.uikit I/gongmingqm10﹕ --onMeasure--1073742904 - 1073742424
	09-25 15:55:29.876    1535-1535/org.gongming.uikit I/gongmingqm10﹕ --onLayout--0 - 0 - 1080 - 600
	09	-25 15:55:29.876    1535-1535/org.gongming.uikit I/gongmingqm10﹕ --onDraw--

通过上面的Logcat，我们可以看到在View的创建过程中， `onMeasure`被连续两次调用，调用完成之后紧接着`onLayout()`，最后进行`onDraw()`，于是一个View完成了从声明到创建的全过程。那么`onMeasure()` `onLayout()` `onDraw()`这几个方法究竟在绘制View的过程中起到了怎样的作用呢？

### 1. onMeasure(int widthMeasureSpec, int heightMeasureSpec)

从Log打印出来的信息来看，参数 widthMeasureSpec和heightMeasureSpec的值看起来没有特别实际的意义，但是Android本身提供了`MeasureSpec.getMode(measureSpec)`和`MeasudeSpec.getSize(measureSpec)`方法，这两个方法能够分别拿到int类型的 `mode` 和 `size`。`mode` 和 `size`主要描述了当前控件在父控件中的占位方式，以及根据占位方法计算出来的大小。我们可以根据 `mode` `size` 的值对控件的实际大小进行自定义控制。如果我们不使用`super.onMeasure(widthMeasureSpec, heightMeasureSpec)`，我们必须调用`setMeasureDimense(width, height)`来使这些设置生效。`width, height`分别表示当前这个控件真实的大小。

通过对`onMeasure()`函数的理解，我们基本知道`onMeasure()`的功能是告诉Android当前View的大小，并且此大小也是根据布局中父容器的约束生成的。当前我们也可以根据这些大小进行自定义设置当前View的大小。引用 [stackoverflow](http://stackoverflow.com/questions/12266899/onmeasure-custom-view-explanation)上的回答：

> onMeasure() is your opportunity to tell Android how big you want your custom view to be dependent the layout constraints provided by the parent; it is also your custom view's opportunity to learn what those layout constraints are (in case you want to behave differently in a match_parent situation than a wrap_content situation). These constraints are packaged up into the MeasureSpec values that are passed into the method.

对于MeasureSpec的`mode`主要有`EXACTLY, AT_MOST, UNSPECIFIED`三种值:

#####`EXACTLY`

字面意义是准确的，也就是我们的View是的宽度和高度是固定准确的，`mode`为这种值时，通常是我们设置了`layout_width`或者`layout_height`的值为一个给定的值，或者我们设置View的宽度或高度为`match_parent`，而父容器的宽度或高度固定。

```
<?xml version="1.0" encoding="utf-8"?>

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <org.gongming.common.swipelist.CustomView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#333666"
        />

</FrameLayout>

```
这里的CustomView宽度和高度都是 `match_parent`，而parent也是充满整个view页面，因此parent的大小是固定的，所以CustomView的大小也是固定的，这里取到的`mode`值就为`EXACTLY`；

如果把FrameLayout的宽度高度都设置成`wrap_content`，此时得到的`mode`同样都是`EXACTLY`，因为FrameLayout宽高收到屏幕大小的约束，其本身的大小是收到子view的影响，此时子View通过`match_parent`获得最大的空间，于是FrameLayout的宽高被撑到了最大值的水平。


#####`AT_MOST`

通过名词本身的描述，这种状态下意味着View存在一个最大值的约束，最大值约束一般来自父View，父View经过层层依赖，又会受到设备屏幕大小的约束。通过下面的例子看什么时侯View会呈现出`AT_MOST`状态：

```
<?xml version="1.0" encoding="utf-8"?>

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <org.gongming.common.swipelist.CustomView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#333666"
        />

</FrameLayout>
```

这种情况下CustomView的高度是`wrap_content`，自适应内容，没有固定的值。但是由于父容器FrameLayout的宽度和高度是固定的，因此FrameLayout的宽高将会约束CustomView的宽高。CustomView的高度此时会有个最大值约束。这是 CustomView高度的Mode是`AT_MOST`.  
如果把FrameLayout的`layout_height`设置为`wrap_content`，由于父容器自身的高度有个`AT_MOST`属性，子元素CustomView的最大值也不会超过父元素的最大值约束。所以此时CustomView的高度Mode仍然是`AT_MOST`。  
综合来看，只要View的当前宽度或者高度不是固定的，但是会存在一个最大值界限，则View 的Mode为`AT_MOST`。


#####`UNSPECIFIED`

`UNSPECIFIED`意味着高度或者宽度值是不明确不具体的，即Android系统本身都很难决定元素的宽度或者高度值，也没有对其宽度高度的约束值。一个典型情况是ScrollView，由于ScrollView的可以自滑动，因此其滑动方向上可以进行无限的伸缩，于是View在绘制时就很难确定其具体约束。

```
<?xml version="1.0" encoding="utf-8"?>

<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <org.gongming.common.swipelist.CustomView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#333666"
        />

</ScrollView>

```
`ScrollView`默认是竖直滑动，因此其高度值是难以确定的，位于其里面的CustomView如果没有提供明确的竖直，那么无论使用`match_parent`，还是`wrap_content`，高度值都是不能确定的。高度的Mode为`UNSPECIFIED`  
除了上述的MeasureSpec.getMode()外，在本函数中我们还可以通过MeasureSpec.getSize()得到View真实的值，这个真实值将会用来绘制当前View。我们自定义View的时侯也可以通过onMeasure进行恰当的重写，从而实现我们自己想要的功能。


### 2. onLayout(boolean changed, int left, int top, int right, int bottom)


`onLayout()`的参数中我们能够直接拿到当前view的位置，(left, top)描述了左上顶点的位置，而(right, bottom)确定了右下角的位置，在自定义View的时侯，可以根据父容器的位置，调用子View的layout()函数直接指定子view的位置。我们的LinearLayout, RelativeLayout等就是继承ViewGroup，然后根据子View设置的相关属性，从而确定子View应该被放在哪里，我们在app中也就看到了期望的界面。


### 3. onDraw(Canvas canvas)

Canvas是View的画布，有了canvasView才会真正的显示出来，才有我们看到的背景，图像，边框等元素。通过复写onDraw()，我们能够利用canvas做一些自定义的行为。比如我们通常看到的显示圆形ImageView头像等。


## 自定义View的实现

####自动换行控件-FlowLayout
这个Demo中主要通过继承ViewGroup实现自动换行控件，这种通常被用来放置TextView，对文字长度和数量位置的TextView来说，自动换行控件能够实现很好的布局效果，我们只需要将TextView加载到FlowLayout中，便可实现TextView组的自动换行。我们也可以批量给TextView增加自定义事件，进而实现我们想做的事情。

自定义ViewGroup主要参考Google的[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)，参照自定义控件写法，主要是人工对onMeasure()和onLayout进行重写。onMeasure()决定了控件本身的宽度和高度，而onLayout()则用来确定子类的位置摆放。子类在父类中的位置摆放通过layout(l, t, r, b)，这几个值表示控件的左上坐标和右下坐标，坐标是相对于父容器的位置来确定的。

首先看具体效果：  
<img src="/images/flow_layout_1.png" alt="FlowLayout" style="width: 250px;"/>


代码实现如下FlowLayout.java

```
package org.gongming.common;

import android.content.Context;
import android.util.AttributeSet;
import android.util.Log;
import android.view.View;
import android.view.ViewGroup;

public class FlowLayout extends ViewGroup {

    private static final int leftMargin = 6;
    private static final int rightMargin = 6;
    private static final int topMargin = 10;
    private static final int bottomMargin = 10;


    public FlowLayout(Context context) {
        super(context);
    }

    public FlowLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public FlowLayout(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int maxWidth = 0;
        int maxHeight = 0;
        int childState = 0;
        int calcHeight = 0;

        int totalWidth = MeasureSpec.getSize(widthMeasureSpec);
        int count = getChildCount();

        for (int index = 0; index < count; index++) {
            final View child = getChildAt(index);
            if (child.getVisibility() != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
                int calcWidth = child.getMeasuredWidth() + leftMargin + rightMargin;
                calcHeight = Math.max(calcHeight, child.getMeasuredHeight() + topMargin + bottomMargin);
                if (maxWidth + calcWidth > totalWidth) {
                    maxWidth = totalWidth;
                    maxHeight += calcHeight;
                } else {
                    maxWidth += calcWidth;
                }
                childState = combineMeasuredStates(childState, child.getMeasuredState());
            }
        }
        maxHeight += calcHeight;
        maxHeight = Math.max(maxHeight, getSuggestedMinimumHeight());
        maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());
        setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                resolveSizeAndState(maxHeight, heightMeasureSpec,
                        childState << MEASURED_HEIGHT_STATE_SHIFT));
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int totalWidth = getMeasuredWidth();
        int count = getChildCount();
        int width = 0;
        int height = 0;

        for (int index = 0; index < count; index++) {
            View view = getChildAt(index);
            int calcHeight = view.getMeasuredHeight() + topMargin + bottomMargin;
            int calcWidth = view.getMeasuredWidth() + leftMargin + rightMargin;
            if (width + calcWidth > totalWidth) {
                height += calcHeight;
                width = calcWidth;
            } else {
                width += calcWidth;
                height = Math.max(calcHeight, height);
            }
            view.layout(width - calcWidth + leftMargin, height - calcHeight + topMargin, width - rightMargin, height - bottomMargin);
        }
    }

    public void setOnItemClickListener(OnClickListener listener) {
        for (int index = 0; index < getChildCount(); index++) {
            View child = getChildAt(index);
            child.setOnClickListener(listener);
        }
    }
}

```

看看我们在xml中如果使用当前定义的View，主要通过包名访问：

```
<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:text="Root Swipe Layout"
        android:layout_gravity="center"
        />
    <org.gongming.common.FlowLayout
        android:layout_height="wrap_content"
        android:layout_width="match_parent"
        android:id="@+id/flowLayout"
        >
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="hahahah"
            android:background="#7E7EFF"
            />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="This is a amazing layout"
            android:background="#7E7EFF"
            />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Where is this layout"
            android:background="#7E7EFF"
            />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Oh, God. Please save me. I'm the cool layout"
            android:background="#7E7EFF"
            />
    </org.gongming.common.FlowLayout>
</LinearLayout>


```

在java代码里还可以定义item的单击事件，`flowLayout.setOnItemClickListener(OnClickListener listener)`， 这样就能对用户的单击行为进行响应。

####定比例长宽RelativeLayout-RatioRelativeLayout
Android设备的多样性使得我们要尽量做出适应多设备的可伸缩布局。在最佳实践中，我们谈到要尽量使用RelativeLayout相对布局。同样，我们在这里自定义了一种固定长宽比例的元素。通过自己指定的高度／宽度的比例值，最终确定View应该绘制的宽度和高度。定义如下，主要增加了一个ratio属性，以及重写了onMeasure()方法。  
res/values/attrs.xml

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="RatioLayout">
        <!--The ratio defines the value of height / width -->
        <attr name="ratio" format="float" />
    </declare-styleable>

</resources>

```

RatioRelativeLayout.java

```
package org.gongming.common;

import android.content.Context;
import android.content.res.TypedArray;
import android.util.AttributeSet;
import android.widget.RelativeLayout;

public class RatioRelativeLayout extends RelativeLayout {

    private float ratio = 0f;

    public RatioRelativeLayout(Context context) {
        super(context);
    }

    public RatioRelativeLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(attrs);
    }

    public RatioRelativeLayout(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    private void init(AttributeSet attrs) {
        TypedArray typedArray = getContext().obtainStyledAttributes(attrs, R.styleable.RatioLayout);
        ratio = typedArray.getFloat(R.styleable.RatioLayout_ratio, 0f);
        typedArray.recycle();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
        int measureHeight = MeasureSpec.getSize(heightMeasureSpec);

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        if (widthMode == MeasureSpec.EXACTLY && ratio != 0f) {
            measureHeight = (int) (measureWidth * ratio);
        } else if (heightMode == MeasureSpec.EXACTLY && ratio != 0f) {
            measureWidth = (int) (measureHeight / ratio);
        } else {
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        }
        setMeasuredDimension(measureWidth, measureHeight);
    }
}

```

在xml使用时使用该控件并给ratio设置值即可：

```
<?xml version="1.0" encoding="utf-8"?>

<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"

    >

    <org.gongming.common.RatioRelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/darker_gray"
        app:ratio="0.5"
        >

    </org.gongming.common.RatioRelativeLayout>

</FrameLayout>

```
最后在界面上我们便可以看到一个高度是宽度一半的自定义View。





















 
