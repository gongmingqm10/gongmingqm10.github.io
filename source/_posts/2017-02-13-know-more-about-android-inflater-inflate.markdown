---
layout: post
title: "Know more about Android Inflater.inflate()"
date: 2017-02-13 15:37:45 +0800
description: Android, Inflater, attachToRoot
comments: true
categories: Android

---

在 Android 开发中，有很多看起来 “约定俗成” 的代码，有时候不妨停下来想一想深层次的含义。最近我在给 Fragment 填充布局时，写到一句熟悉的代码：

```java
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.activity_main, container, false);
        return view;
    }
```

我又一次陷入了思索，其中的第三个参数 `false` 到底是什么意思，此处能不能变为 `true`。

<!-- more -->

## 探究

首页我们还是通过函数本身的注释来看各参数的作用，通过源码查看如下：

```
    /**
     * Inflate a new view hierarchy from the specified xml resource. Throws
     * {@link InflateException} if there is an error.
     * 
     * @param resource ID for an XML layout resource to load (e.g.,
     *        <code>R.layout.main_page</code>)
     * @param root Optional view to be the parent of the generated hierarchy (if
     *        <em>attachToRoot</em> is true), or else simply an object that
     *        provides a set of LayoutParams values for root of the returned
     *        hierarchy (if <em>attachToRoot</em> is false.)
     * @param attachToRoot Whether the inflated hierarchy should be attached to
     *        the root parameter? If false, root is only used to create the
     *        correct subclass of LayoutParams for the root view in the XML.
     * @return The root View of the inflated hierarchy. If root was supplied and
     *         attachToRoot is true, this is root; otherwise it is the root of
     *         the inflated XML file.
     */
    public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
	    ...
    }
   
```

上面的解释比较官方，但是不一定好理解。至少我就看了好几遍，最后结合网上的总结才明白其含义。

首先就 `LayoutInflater.inflate()`这个函数而言，功能是将一段 XML 资源文件加载成为 View。所以通常用于将 XML 文件实例化为 View。然后获取 View 上的组件最后操作之。

对于这个函数，我们通常容易引发歧义的地方在 `attachToRoot`的值，当这个值为 true/false 时，是由显著的区别的。

### attachToRoot = true

当设置 `attachToRoot = true` 时, 表示 xml 实例化的 View 会被添加到第二个参数 rootView 中。 所以此时相当于处理了两件事：

* 将 XML 实例化为 View；
* 将实例化的 View 添加到非空的 rootView 中；

经典的使用方法，对于不为空的 container，

```java
LayoutInflater.from(getContext()).inflate(R.layout.demo, container, true);

```

或者:

```java
LayoutInflater.from(getContext()).inflate(R.layout.demo, container);
```

当 ViewGroup 不为空时，第三个 true 参数可以直接省略。

### attachToRoot = false 

当设置 `attachToRoot = false` 时，表示直接将 xml 实例化为 View，而不用将 View 添加到指定的 ViewGroup 中。如果需要将 View 加入到 ViewGroup 中，还需要手动的调用 `ViewGroup.addView(view)`。

经典写法为：

```java
LayoutInflater.from(getContext()).inflate(R.layout.demo, container, false);
```

注意：对于上面的第二个参数 container，如果直接设置为 null，有可能导致 View 的位置或者大小等显示不正确。所以上面代码做了两件事：

* 将 XML 实例化为 View;
* 根据 ViewGroup 为子元素 View 设置正确的位置、大小等 LayoutParams 属性；

所以，实例化 View 时，如果已经明确知道父容器 ViewGroup，请直接传递 ViewGroup，避免出现子 View 显示时的位置、大小等异常情形。

### 比较

对于非空的 ViewGroup continer，以下两句是等效的：

```java
View buttonView = LayoutInflater.from(getContext()).inflate(R.layout.ugly_button, container, false);
container.addView(buttonView);
```

与

```java
View buttonView = LayoutInflater.from(getContext()).inflate(R.layout.ugly_button, container, true);
```

## 应用

通常将 XML 实例化为 View 涉及到以下常见情形：


### ListView/RecyclerView 中 Adapter.getView

```java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    ViewHolder viewHolder;

    if (convertView == null) {
        convertView = LayoutInflater.from(context).inflate(R.layout.ugly_button, parent, false);
        viewHolder = new ViewHolder();
        convertView.setTag(viewHolder);
    } else {
        viewHolder = (ViewHolder) convertView.getTag();
    }

    // ...
    return convertView;
}
```
此处应该设置 **attachToRoot = true**

因为 ListView 何时显示，以及如何显示。应该是由 ListView 决定的。Adapter.getView() 主要是根据指定的函数返回相应的 View 实例。显示逻辑则由 ListView 处理，因此设置 `attachToRoot = true`。

### 自定义 View

为了使得布局的层级足够浅，我们在布局时通常使用 `<merge></merge>` 定义 layout 的根元素，然后将 layout 填充到自定义的 View 中。

res/layout/bottom\_tab\_item.xml:

```
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:orientation="vertical">
        <ImageView
            android:id="@+id/tab_icon"
            android:layout_width="24dp"
            android:layout_height="24dp"
            android:paddingTop="2dp" />
        <TextView
            android:id="@+id/tab_label"
            android:layout_marginTop="1dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="14dp"
            tools:text="Tab"/>
    </LinearLayout>
</merge>
```

BottomTabItem.java:

```java
public class BottomTabItem extends RelativeLayout {
	...
	
	public void init() {
		LayoutInflater.from(getContext()).inflate(R.layout.bottom_tab_item, this, true);
	}
	...
}
```

对于自定义的 BottomTabItem，需要将定义的 XML 文件添加到其中。所以需要直接将 XML 实例化 View 作为子元素，添加到 BottonTabItem 中。

所以，对于自定义 View， 需要设置 `attachToRoot = true`。

### Fragment.onCreateView()

回顾我们最看我们最开始遇到的代码块：

```
@Nullable
@Override
public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.activity_main, container, false);
    return view;
}
```

onCreateView 函数直接反馈 View，可以看出函数只需要拿到实例化的 View，至于 View 是如何添加到 Fragnent 中，则是底层 Fragment 的逻辑。

如果在 Fragment 中不恰到的处理这个 attchToRoot 参数，会直接导致严重的崩溃性问题。而且通常出现这种问题时很不容易被发现。

所以，对于 Fragment 添加 layout，需要设置 `attachToRoot = false`。

### ListView headerView/footerView 等

对于 ListView.addHeaderView() / addFooterView() 也是我们经常使用 LayoutInflator.inflate() 很多的场合。

从 ListView.addHeaderView(View view) 函数定义可以看到，ListView 需要将实例化的 View 添加作为 header/footer view，所以显然需要的只是 View 的实例。添加或者删除操作通过 ListView 内部进行处理。

所以，对于 ListView 添加 headerView, footerVuew 等，需要设置 `attachToRoot = false`。

## 结语

知道 LayoutInflater.inflate 的细微知识点，可能会花费你一下午的时间。但是却可以让你对所写的代码更加自信，并且能够对事物认识更加深刻。何乐而不为呢？

## 参考

* [深入理解LayoutInflater.inflate()](http://blog.chengdazhi.com/index.php/110)
* [Understanding Android's LayoutInflater.inflate()](https://www.bignerdranch.com/blog/understanding-androids-layoutinflater-inflate/)





