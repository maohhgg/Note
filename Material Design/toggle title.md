---
title: ActionBar跟随滚动显示
date: 2016/9/9
tag:
  - Material Design
urlname: ActionBar-Scroll-display
---



隐藏ActionBar的方法有很多，可以通过代码的方式隐藏，也可以通过配置文件的方式，本片介绍简单的方式**通过配置文件**

先来介绍一下`CoordinatorLayout`，他可以作为顶层布局，也可以调度协调子布局。

`CoordinatorLayout`通过协调调度子布局的形式实现触摸影响布局的形式产生动画效果。`CoordinatorLayout`通过设置子View的 `app:layout_behavior`来调度子View。

在我们的项目使用`CoordinatorLayout`来同步滚动ActionBar和内容View。

使用`CoordinatorLayout`作为主要布局。需要关闭Theme自带的ActionBar，可以新建style建立新主题并加入
<!--more-->
```xml
<item name="windowActionBar">false</item>
<item name="windowNoTitle">true</item>
```

也可以设置Theme为`.NoActionBar`结尾的没有ActionBar的Theme

**注意** Android studio 2.1 自动生成的模板中 `style.xml v21` 增加了下面两行代码

```xml
<item name="android:windowDrawsSystemBarBackgrounds">true</item>
<item name="android:statusBarColor">@android:color/transparent</item>
```

在`CoordinatorLayout`标签中使用`android:fitsSystemWindows="true"`，会导致ActionBar滚动到顶部是占据前端，遮挡系统状态栏。 在`CoordinatorLayout`不使用`android:fitsSystemWindows="true"`又会实现`style.xml v21`中定义的条件，使得系统状态栏为白色等情况，和应用不协调。建议删除上面两行配置。

```xml
<android.support.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways|snap"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

</android.support.design.widget.AppBarLayout>
```

通常Toolbar如上定义，`Toolbar`使用`AppBarLayout`标签包裹。

> `AppBarLayout`继承自`LinearLayout`,可以认为是垂直布局的`LinearLayout`，`AppBarLayout`是在`LinearLayou`上加了一些材料设计的概念，它可以让你定制当某个可滚动View的滚动手势发生变化时，其内部的子View实现何种动作。

在需要滚动动作的标签上定义`app:layout_scrollFlags`属性。 `app:layout_scrollFlags`属性和对应动作

1. scroll 值设为scroll的View会跟随滚动事件一起发生移动。

2. enterAlways 值设为enterAlways的View,当ScrollView往下滚动时，该View会直接往下滚动。而不用考虑ScrollView是否在滚动。（对于滚动的绝对定位滚动）

3. exitUntilCollapsed 值设为exitUntilCollapsed的View，当这个View要往上逐渐"消逝"时，会一直往上滑动，直到剩下的的高度达到它的最小高度后，再响应ScrollView的内部滑动事件。

4. enterAlwaysCollapsed 是enterAlways的附加选项，一般跟enterAlways一起使用，它是指，View在往下"出现"的时候，首先是enterAlways效果，当View的高度达到最小高度时，View就暂时不去往下滚动，直到ScrollView滑动到顶部不再滑动时，View再继续往下滑动，直到滑到View的顶部结束。

当需要在AppBarLayout中引入图片时，`ToolBar`需要使用`CollapsingToolbarLayout`标签包裹起来。

```xml
<android.support.design.widget.CollapsingToolbarLayout
    android:id="@+id/toolbar_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    app:contentScrim="?attr/colorPrimary"
    app:expandedTitleMarginEnd="64dp"
    app:expandedTitleMarginStart="48dp"
    app:layout_scrollFlags="scroll|exitUntilCollapsed">
    <ImageView
        android:id="@+id/imageView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_collapseMode="parallax"
        app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
        android:adjustViewBounds="true"/>
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        app:layout_collapseMode="pin" />
</android.support.design.widget.CollapsingToolbarLayout>
```

向上滚动时`ImageView`高度减小，当`AppBarLayout`高度为最小时，`ImageView`隐藏，`AppBarLayout`展示的是`Toolbar`内容。所以`ImageView`标签必须在前，`Toolbar`在后。

`app:layout_collapseMode="parallax"`属性是使`ImageView`在滚动时有视差的效果，视差的不同效果可以用`app:layout_collapseParallaxMultiplier="0.5"`来设置参数范围`0.0`到`1.0`，默认`0.5`。

当使用`CollapsingToolbarLayout`标签时，`Toolbar.setTitle()`不再有效，标题必须设置在`CollapsingToolbarLayout` View上。

```java
CollapsingToolbarLayout collapsingToolbar =
                (CollapsingToolbarLayout) findViewById(R.id.toolbar_layout);
        collapsingToolbar.setTitle("title");
```

`AppBarLayout`子View不同时实现不同的动作可见[AndroidStudioProjects/Animation](https://github.com/maohhgg/AndroidStudioProjects/tree/master/Animation)中[MainActivity](https://github.com/maohhgg/AndroidStudioProjects/blob/master/Animation/app/src/main/java/com/example/mao/animation/MainActivity.java)和[DetailActiviy](https://github.com/maohhgg/AndroidStudioProjects/blob/master/Animation/app/src/main/java/com/example/mao/animation/DetailActivity.java)不同的滚动效果。
