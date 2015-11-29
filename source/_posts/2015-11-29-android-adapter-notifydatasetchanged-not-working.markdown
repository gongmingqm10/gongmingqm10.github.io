---
layout: post
title: "Android Adapter notifyDataSetChanged not working"
date: 2015-11-29 11:30:39 +0800
description: Android Adapter notifyDataSetChanged, not working
comments: true
categories: Android

---

刚开发 Android 开发时，总是遇到一个看似很简单的问题 “Android Adapter notifyDataSetChanegd 不生效”，而每次解决这个问题的方法总是简单粗暴，直接了当。大概是以前的时候做事以结果导向，所以对于这些奇怪的问题也总是没有深究。刚好最近在项目中再次遇到了这个问题，决心深究一下。

![notifyDataSetChanged not working](http://7xj9js.com1.z0.glb.clouddn.com/Its_not_working_anymore.png =500x)

<!-- more -->

### Adapter notifyDataSetChanged 做了什么事

要弄清楚 `adapter.notifyDataSetChanged` 为什么不工作，还是应该首先弄清楚它究竟做了什么事情。根据函数名的描述，主要是通知 “数据源改变”，从而刷新页面，这样新改变的数据就呈现在用户面前了。

Android Adapter 使用了观察者模式，每一个 ListView 可以实例化出一个 Observer，而每个 Adapter 则会实例化出一个 Subject。在 ListView setAdapter 的时候，Adapter 会将 ListView 中实例化的 Observer 注册入观察者列表中。当 Adapter 通过观察者更新时，则会调用 Observer 的 notifyChanged() 方法，如此重新刷新 ListView 的界面。

这个通知者模式的 UML 关系图大致可以描述如下：

![notifyDataSetChanged not working](http://7xj9js.com1.z0.glb.clouddn.com/adapter%20obervable%20pattern.png =500x)

继续查看 ListView.setAdapter 方法，可以看到在设置 Adapter 的时候，ListView 会将自己的 Observer 注册入 Subject:

```
    @Override
    public void setAdapter(ListAdapter adapter) {
        if (mAdapter != null && mDataSetObserver != null) {
            mAdapter.unregisterDataSetObserver(mDataSetObserver);
        }

		...

        super.setAdapter(adapter);

        if (mAdapter != null) {
			...
            mDataSetObserver = new AdapterDataSetObserver();
            mAdapter.registerDataSetObserver(mDataSetObserver);

			...
            }
        }
        
        ...
    }

```

### 为什么 notifyDataSetChanged 生效

既然 `notifyDataSetChanged` 能够促使 ListView 重绘界面，那么让其生效的方式就是保证 Adapter 中的数据源生效，这样在 ListView 重绘时才能使用新的数据刷新界面。看下面一段失效的 ListView 更新失效的问题。

```
        ListView itemList = (ListView) findViewById(android.R.id.list);
        data = new ArrayList<>();
        data.addAll(Arrays.asList("Item 1", "Item 2", "Item 3"));
        adapter = new ItemListAdapter(data);
        itemList.setAdapter(adapter);

        findViewById(R.id.fab).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 模拟请求 API 刷新重新获得数据
                data = new ArrayList<>();
                data.addAll(Arrays.asList("Item 1", "Item 2", "Item 3", "Item 4"));
                adapter.notifyDataSetChanged();
            }
        });
```

让我们分析失效的原因，首先将 ArrayList data 设置为 Adapter 的数据源，而 Java 是引用传值，所以在 onClick() 回调函数中，当我们对 data 重新赋值时，重新赋值后的 data 并不能和 Adapter 中的数据源产生关联。因此 Adapter 中的数据源还是 “Item 1”, “Item 2”, "Item 3"。所以这里无论怎样调用 notifyDataSetChanged 都是不能生效的。

### 如何让 notifyDataSetChanged 生效

让更新生效的方式是，让你的 Adapter 保持最新数据源。就上述例子来看，当 data 更新新值时，我们可以通过 List.add() 来更新 Adapter 中引用对应的值。实现代码如下：

```

        ListView itemList = (ListView) findViewById(android.R.id.list);
        data = new ArrayList<>();
        data.addAll(Arrays.asList("Item 1", "Item 2", "Item 3"));
        adapter = new ItemListAdapter(data);
        itemList.setAdapter(adapter);

        findViewById(R.id.fab).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                data.add("New Item " + view.toString());
                adapter.notifyDataSetChanged();
            }
        });
```
### End

Android 中有很多优秀的设计，当我们出现问题时，可以通过阅读源码的方式探索，并结合 Google 的相关资料，了解事物的根源，方能设计出更优秀的代码。



