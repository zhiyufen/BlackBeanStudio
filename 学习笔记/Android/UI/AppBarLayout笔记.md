## AppBarLayout笔记

[TOC]

### 概念

AppBarLayout是一个垂直的LinearLayout，实现了Material Design中app bar的scrolling gestures特性。AppBarLayout是在LinearLayou上加了一些材料设计的概念，它可以让你定制当某个可滚动View的滚动手势发生变化时，其内部的子View实现何种动作(通过layout_scrollFlags属性或是setScrollFlags()方法来指定)。

AppBarLayout只有作为CoordinatorLayout的直接子View时才能正常工作；

AppBarLayout的设计精髓在于，与导航栏（也就是Toolbar）、顶部标签栏（即TabLayout）一起使用，来达到MD风格中App Bar的一些滚动交互设计效果。

一般组合使用：

```
CoordinatorLayout+AppBarLayout+Toolbar

CoordinatorLayout+AppBarLayout+CollapsingToolbarLayou
```



### 特性

#### 设置动作模式

##### scroll 模式

子View会跟随滚动事件一起发生移动而滚出或滚进屏幕。

> 1. 其它动作模式必须带上它，不然无效：app:layout_scrollFlags="scroll|enterAlways"
> 2. 如果在这个子View前面的任何其他子View没有设置这个值，那么这个子View的设置将失去作用。

```xml
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/tb_toolbar"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:background="@color/colorPrimary"
            app:layout_scrollFlags="scroll"
            app:title="@string/app_name" />

    </android.support.design.widget.AppBarLayout>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

</android.support.design.widget.CoordinatorLayout>
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e54ccef9b274ea7a178815441923e6d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

##### enterAlways模式

和scroll相比较，其实就是向下滚动时优先级问题，scroll首先滑动的是列表，列表的数据全部滚动完毕，才开始toolbar滑动。而scroll | enterAlways首先滑动的是toolbar ，然后再去滑动其他的view。只是优先级先后的问题。

```xml
app:layout_scrollFlags="scroll|enterAlways"
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eae880898721401c8196d83a53b9d65f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

##### enterAlwaysCollapsed模式

enterAlwaysCollapsed是enterAlways的附加标志，这里涉及到子View的高度和最小高度，向下滚动时，子View先向下滚动最小高度值，然后Scrolling View开始滚动，到达边界时，子View再向下滚动，直至显示完全。

```xml
.... 
<android.support.v7.widget.Toolbar
            android:id="@+id/tb_toolbar"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:minHeight="50dp"
            android:background="@color/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
            app:title="@string/app_name" />
....
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a62a7ce8ea8e46fc98aedc2aaa51a89d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### exitUntilCollapsed模式

发生向上滚动事件时，子View向上滚动直至最小高度，然后Scrolling View开始滚动。也就是，子View不会完全退出屏幕。

```xml
...
<android.support.v7.widget.Toolbar
            android:id="@+id/tb_toolbar"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:minHeight="50dp"
            android:background="@color/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            app:title="@string/app_name" />

...
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f1e9d07dc3f4b8e83feb50165b289a3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

##### snap 模式

子View滚动比例的吸附效果。也就是说子View不会存在局部显示的情况，滚动到子View的部分高度，当我们松开手指时，子View要么向上全部滚出屏幕，要么向下全部滚进屏幕。

```xml
...
 <android.support.v7.widget.Toolbar
            android:id="@+id/tb_toolbar"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:background="@color/colorPrimary"
            app:layout_scrollFlags="scroll|snap"
            app:title="@string/app_name" />
...
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3848167cb8324e7d8adc5b0c34564ae9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

##### snapMargins模式

snapMargins是必须配合snap一起使用的额外的flag。如果设置的话，这个View将会被snap到它的顶部外边距和它的底部外边距的位置，而不是这个View自身的上下边缘。

```xml
...
        <android.support.v7.widget.Toolbar
            android:id="@+id/tb_toolbar"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:layout_marginStart="10dp"
            android:layout_marginTop="200dp"
            android:layout_marginEnd="10dp"
            android:layout_marginBottom="10dp"
            android:background="@color/colorAccent"
            app:layout_scrollFlags="scroll|snap|snapMargins"
            app:title="@string/app_name" />
...
```

Margin生效了，滑动必须超过Toolbar的高度以及上下Margin就会继续滑动，否则就恢复。



> 内容来自:
>
> https://juejin.cn/post/6939814211522396190