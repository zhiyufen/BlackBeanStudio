# [Android笔记] Toolbar学习

## 1. gradle配置

```xml
	compile 'com.android.support:appcompat-v7:25.3.1'
	compile 'com.android.support:design:25.3.1'
```

## 2. 基础使用

a. 配置style让其去掉ActionBar；

```xml
    <style name="ToolbarStyle" parent="@style/Theme.AppCompat.Light">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
```

b. 把Activity的主题设置为上面去掉ActionBar的style;

```xml
 <activity android:name=".design_demo.ToolbarDemoActivity"
            android:theme="@style/ToolbarStyle" />
```

c. Activity的布局文件如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/content_parent"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimaryDark"
        app:title="标题"
        app:titleTextColor="#f34457"
        app:navigationIcon="@drawable/damai"
        app:subtitle="子标题"
        app:subtitleTextColor="#9f4457">
    </android.support.v7.widget.Toolbar>
</RelativeLayout>
```

## 3. Toolbar 组合其它Layout的效果使用

### 3.1 CoordinatorLayout 

&emsp;&emsp;CoordinatorLayout 作为一个 “super-powered FrameLayout”，主要有以下两个作用：作为顶层布局 和 作为协调子 View 之间交互的容器；

&emsp;&emsp;CoordinatorLayout 实现了一个NestedScrollingParent接口，所以它包含的子View只要实现NestedScrollingChild接口的，就很容易实现根据子View的滚动来控制CoordinatorLayout View的滚动了，如标题栏跟随视图的滚动。

		PS:Android NestedScrolling机制, 请参考大神文章：https://www.jianshu.com/p/aff5e82f0174

#### CoordinatorLayout 与 FloatingActionButton协同使用
&emsp;&emsp;CoordinatorLayout 提供了两个属性用来设置FloatingActionButton 的位置，CoordinatorLayout 会动态调整 FAB 的位置：				

	layout_anchor：设置 FAB 的锚点，我们熟悉的 PopupWindow 也有类似概念。
	layout_anchorGravity：设置相对锚点的位置，如bottom|right表示 FAB 位于锚点的右下角。

### 3.2 CoordinatorLayout与AppBarLayout
&emsp;&emsp;AppBarLayout 是一个垂直布局的 LinearLayout，它主要是为了实现 “Material Design” 风格的标题栏的特性；

&emsp;&emsp;AppBarLayout 与 CoordinatorLayout  配合使用时，可以通过设置app:layout_scrollFlags 属性来让其标题滚动到屏幕外：

| 属性值                 | 说明                                                                                                                                                                                                                                    |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| scroll              | 隐藏的时候，先整体向上滚动，直到 AppBarLayout 完全隐藏，再开始滚动 Scrolling View；显示的时候，直到 Scrolling View 顶部完全出现后，再开始滚动 AppBarLayout 到完全显示。                                                                                                                     |
| enterAlways         | 与 scroll 类似（scroll\|enterAlways），只不过向下滚动先显示 AppBarLayout 到完全，再滚动 Scrolling View。                                                                                                                                                      |
| enterAlwaysCollapse | 需要和 enterAlways 一起使用（scroll\|enterAlways\|enterAlwaysCollapsed），和 enterAlways 不一样的是，不会显示 AppBarLayout 到完全再滚动 Scrolling View，而是先滚动 AppBarLayout 到最小高度，再滚动 Scrolling View，最后再滚动 AppBarLayout 到完全显示；   注：需要定义 View 的最小高度（minHeight）才有效果： |
| exitUntilCollapsed  | 定义了 AppBarLayout 消失的规则。发生向上滚动事件时，AppBarLayout 向上滚动退出直至最小高度（minHeight），然后 Scrolling View 开始滚动。也就是，AppBarLayout 不会完全退出屏幕；                                                                                                               |

注：除了 scroll，其它取值，这些属性都必须与 scroll 一起使用 “|” 运算符

Demo:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/content_parent"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <android.support.design.widget.CoordinatorLayout
        android:id="@+id/settings_coordinator_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="标题"
                android:textSize="20sp"
                android:gravity="center"
                android:paddingTop="10dp"
                android:paddingBottom="10dp"
                android:minHeight="10dp"
                android:textColor="@android:color/white"
                android:background="@color/colorPrimary"
                app:layout_scrollFlags="scroll|exitUntilCollapsed"/>
        </android.support.design.widget.AppBarLayout>

        <android.support.v4.widget.NestedScrollView
            android:id="@+id/scrollView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">
            <!--@string/appbar_scrolling_view_behavior 表示使用android.support.design.widget.AppBarLayout$ScrollingViewBehavior来处理 NestedScrollView 与 AppBarLayout 的关系-->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/tv"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:textSize="200sp"
                    android:gravity="center_horizontal"
                    android:text="Tesing"/>
            </LinearLayout>
        </android.support.v4.widget.NestedScrollView>
    </android.support.design.widget.CoordinatorLayout>

</RelativeLayout>
```

&emsp;&emsp;把上面 的 TextView 改为Toolbar也是可以的，enterAlwaysCollapsed不需要加minHeight也有效果：
```xml
		<android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorPrimaryDark"
                app:title="标题"
                app:titleTextColor="#f34457"
                app:navigationIcon="@drawable/damai"
                app:subtitle="子标题"
                app:subtitleTextColor="#9f4457"
	            android:minHeight="10dp"
                app:layout_scrollFlags="scroll|enterAlwaysCollapsed">
            </android.support.v7.widget.Toolbar>
        </android.support.design.widget.AppBarLayout>

```

### 3.3 CoordinatorLayout 与 CollapsingToolbarLayout
&emsp;&emsp;CollapsingToolbarLayout 继承自 FrameLayout，它是用来实现 Toolbar 的折叠效果： 下拉时，标题栏会进行拉伸进而显示一些东西；一般它的直接子 View 是 Toolbar；

	app:layout_collapseMode的属性：
	scroll|enterAlwaysCollapsed： 上拉标题栏会隐藏；
	scroll|exitUntilCollapsed： 上拉标题栏会固定位；
		
&emsp;&emsp;layout_collapseMode除了使用 pin 固定住 View，还可以使用 parallax，视差的意思就是：移动过程中两个 View 的位置产生了一定的视觉差异。

Demo:
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/content_parent"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <android.support.design.widget.CoordinatorLayout
        android:id="@+id/settings_coordinator_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="200dp">
            <android.support.design.widget.CollapsingToolbarLayout
                android:id="@+id/collapsingToolbarLayout"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:contentScrim="@color/colorPrimary"
                app:layout_scrollFlags="scroll|exitUntilCollapsed">
                <ImageView
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:src="@drawable/mobike_bg"
                    android:scaleType="centerCrop"
                    app:layout_collapseParallaxMultiplier="0.9"
                    app:layout_collapseMode="parallax"/>
                                    <!-- android:layout_gravity="bottom"可以让 Toolbar 在ImageView的下面-->
                <android.support.v7.widget.Toolbar
                    android:id="@+id/toolbar"
                    android:layout_width="match_parent"
                    android:layout_height="?attr/actionBarSize"
	                                 android:layout_gravity="bottom"
                    app:title="标题"
                    android:minHeight="11dp">
                </android.support.v7.widget.Toolbar>
            </android.support.design.widget.CollapsingToolbarLayout>
        </android.support.design.widget.AppBarLayout>

        <android.support.v4.widget.NestedScrollView
            android:id="@+id/scrollView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">
            <!--@string/appbar_scrolling_view_behavior 表示使用android.support.design.widget.AppBarLayout$ScrollingViewBehavior来处理 NestedScrollView 与 AppBarLayout 的关系-->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/tv"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:textSize="500sp"
                    android:gravity="center_horizontal"
                    android:text="Tesingxxxxxxxxxxxxxxxxx"/>
            </LinearLayout>
        </android.support.v4.widget.NestedScrollView>
    </android.support.design.widget.CoordinatorLayout>

</RelativeLayout>
```

其它属性：

		contentScrim：表示 CollapsingToolbarLayout 折叠之后的“前景色”，我们看到 Toolbar 的变色，其实是因为前景色遮挡了而已。
		statusBarScrim：表示状态栏的“前景色”
		app:layout_scrollFlags=”scroll” 和介绍 AppBarLayout 时，给Toolbar 设置和配置一样。其效果上图已做展示。
		app:expandedTitleGravity=”center_horizontal” 从单词意思可以看出，这是展示后，title的位置。
		app:expandedTitleMarginStart=”50dp” 这是展示后，title 距离开始位置的边距。
		app:collapsedTitleGravity=”center” 这是收缩后，title 位置（测试发现，不好用）。
		app:expandedTitleTextAppearance=”@style/transparentText” 展开后标题文字的样式
		app:collapsedTitleTextAppearance=”@style/ToolbarTitle” 折叠后标题文字的样式

## 4. Toolbar的详细使用

### 4.1 组合menu配置文件来显示菜单

Menu XML
```xml
//toolbal_menu.xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_search"
        android:icon="@drawable/ic_search"
        android:title="搜索"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_notifications"
        android:icon="@drawable/card_category_icon_balance"
        android:title="通知"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_settings"
        android:icon="@mipmap/ic_launcher"
        android:orderInCategory="100"
        android:title="Settings"
        app:showAsAction="never" />
</menu>
```

Activity:
```java
public class ToolbarDemoActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.toolbar_demo_acitivity);

        Toolbar mToolBar = findViewById(R.id.toolbar);
        mToolBar.inflateMenu(R.menu.toolbal_menu);
        //setSupportActionBar(mToolBar);
    }
}
```

效果图(略)

&emsp;&emsp;这里可以和AppBarLayout的app:layout_scrollFlags组合各种折叠效果；

	app:layout_scrollFlags=”scroll”
	app:layout_scrollFlags=”scroll|enterAlways”
	app:layout_scrollFlags=”scroll|enterAlways|snap”
	app:layout_scrollFlags=”scroll|enterAlways|exitUntilCollapsed”
	app:layout_scrollFlags=”scroll|enterAlways|enterAlwaysCollapsed”


### Demo: 模仿Setting的ActionBar效果

Layout:
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/settings_coordinator_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.design.widget.AppBarLayout
        android:id="@+id/debug_settings_app_bar"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        style="@style/Widget.Design.AppBarLayout"
        app:elevation="0dp">
        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/debug_collapsing_app_bar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed|snap"
            android:background="#f2f2f2"
            style="@style/Widget.Design.CollapsingToolbar">
            <android.support.v7.widget.AppCompatTextView
                android:id="@+id/collapsing_bar_title"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:background="@android:color/transparent"
                android:gravity="bottom|center"
                android:alpha="0"
                android:includeFontPadding="false"
                android:maxLines="2"
                android:ellipsize="end"
                android:textColor="#252525"
                android:layout_marginLeft="20dp"
                android:layout_marginRight="20dp"
                android:textSize="35sp"/>
            <android.support.v7.widget.Toolbar
                android:id="@+id/debug_settings_toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:layout_gravity="bottom"
                app:layout_collapseMode="pin"
                android:theme="@style/SettingsExtendedAppBarStyle"
                app:contentInsetStartWithNavigation="0dp"
                app:contentInsetStart="0dp">
                <android.support.v7.widget.AppCompatTextView
                    android:id="@+id/toolbar_title"
                    android:layout_width="match_parent"
                    android:layout_height="?attr/actionBarSize"
                    android:alpha="0"
                    android:background="@android:color/transparent"
                    android:fontFamily="sec-roboto-regular"
                    android:gravity="center_vertical"
                    android:textDirection="locale"
                    android:singleLine="true"
                    android:textColor="#252525"
                    android:textSize="19sp"
                    android:layout_marginBottom="10dp" />
            </android.support.v7.widget.Toolbar>
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:id="@+id/scrollView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
            <TextView
                android:id="@+id/tv"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:textSize="500sp"
                android:gravity="center_horizontal"
                android:text="Tesingxxxxxxxxxxxxxxxxx"/>
        </LinearLayout>
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```

相应Style:
```xml
<style name="AppCompatActionBarStyle" parent="Widget.AppCompat.Light.ActionBar">
        <item name="background">@color/default_actionbar_background</item>
        <item name="icon">@android:color/transparent</item>
    </style>

    <style name="ActionBarStyle" parent="android:Widget.DeviceDefault.Light.ActionBar">
        <item name="android:background">@color/default_actionbar_background</item>
        <item name="android:icon">@android:color/transparent</item>
        <item name="android:titleTextStyle">@style/ActionBarTitleStyle</item>
    </style>

    <style name="SettingsExtendedAppBarStyle" parent="ActionBarStyle">
        <item name="android:homeAsUpIndicator">@drawable/action_bar_up_btn</item>
        <item name="actionBarStyle">@style/AppCompatActionBarStyle</item>
        <item name="android:textColor">@android:color/black</item>
    </style>

    <style name="ActionBarTitleStyle" parent="android:Widget.DeviceDefault.Light.TextView">
        <item name="android:textSize">17sp</item>
        <item name="android:fontFamily">sans-serif-condensed</item>
        <item name="android:textStyle">bold</item>
        <item name="android:textColor">#e6252525</item>
        <item name="android:textAllCaps">false</item>
    </style>

//其中 action_bar_up_btn.xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/tw_ic_ab_back_mtrl"
    android:width="24dp"
    android:height="24dp"
    android:autoMirrored="true"
    android:tint="#209861">
</bitmap>
```

ToolbarDemoActivity:
```java
package blackbean.rxjavademo.design_demo;

import android.app.ActionBar;
import android.content.Context;
import android.content.res.Configuration;
import android.content.res.TypedArray;
import android.graphics.Point;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.design.widget.AppBarLayout;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.util.Log;
import android.util.TypedValue;
import android.view.View;
import android.view.ViewGroup;
import android.view.WindowManager;
import android.widget.TextView;

import blackbean.rxjavademo.R;

public class ToolbarDemoActivity extends AppCompatActivity implements AppBarLayout.OnOffsetChangedListener{
    public static final String TAG = "ToolbarDemoActivity";
    private Toolbar mToolbar;
    private AppBarLayout mAppBarLayout;
    private TextView mTitleContainer;
    private TextView mTitle;

    private int mPrevOrientation = Configuration.ORIENTATION_UNDEFINED;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.toolbar_demo_acitivity);

        mToolbar = findViewById(R.id.debug_settings_toolbar);
        mAppBarLayout = findViewById(R.id.debug_settings_app_bar);
        mTitle = findViewById(R.id.toolbar_title);
        mTitleContainer = findViewById(R.id.collapsing_bar_title);
        mTitle.setText("我是标题");
        mTitleContainer.setText("我是标题");

        setSupportActionBar(mToolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        if (getActionBar() != null) {
            getActionBar().setDisplayOptions(
                    ActionBar.DISPLAY_HOME_AS_UP | ActionBar.DISPLAY_SHOW_TITLE);
        }
        mAppBarLayout.addOnOffsetChangedListener(this);
        mAppBarLayout.setExpanded(false);
        resetAppBarHeight();
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        if (mPrevOrientation != newConfig.orientation) {
            mPrevOrientation = newConfig.orientation;
            mAppBarLayout.setExpanded(false);
            resetAppBarHeight();
        }
    }

    @Override
    public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
        int totalScrollRange = appBarLayout.getTotalScrollRange();
        float offsetRatio = (float) Math.abs(verticalOffset) / (float) totalScrollRange;
        mTitleContainer.setAlpha(1.5f - (offsetRatio * 2));
        mTitle.setAlpha((offsetRatio * 2) - 1);
    }

    private void resetAppBarHeight() {
        ViewGroup.LayoutParams layoutParams = mAppBarLayout.getLayoutParams();
        int screenHeight = getWindowHeight(this);
        if (isLandscape()) {
            mTitleContainer.setVisibility(View.GONE);
            TypedArray array = null;
            try {
                array = getTheme().obtainStyledAttributes(new int[] {R.attr.actionBarSize});
                String string = array.getString(0);
                String number = string.replaceAll("[^[0-9|.]]", "");
                float dpValue = Float.parseFloat(number);
                float pxValue = dpToPx(dpValue);
                layoutParams.height = (int) pxValue;
            } catch (Exception e) {
                layoutParams.height =
                        (int) getResources().getDimension(R.dimen.action_bar_button_height);
            } finally {
                array.recycle();
            }
        } else {
            mTitleContainer.setVisibility(View.VISIBLE);
            layoutParams.height = (int) (screenHeight * 0.38f);
        }
        mAppBarLayout.setLayoutParams(layoutParams);
    }

    /**
     * Returns height of window display
     * @param context context
     * @return window display height value
     */
    private static int getWindowHeight(Context context) {
        int windowHeight = 0;
        try {
            WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
            if (wm != null) {
                Point point = new Point();
                wm.getDefaultDisplay().getSize(point);
                windowHeight = point.y;
            }
        } catch (Exception e) {
            Log.e(TAG, "cannot get window width");
        }
        return windowHeight;
    }

    /**
     * Check if the orientation is landscape mode
     */
    private boolean isLandscape() {
        Configuration configuration = getResources().getConfiguration();
        return configuration.orientation == Configuration.ORIENTATION_LANDSCAPE;
    }

    private int dpToPx(float dp) {
        return (int) TypedValue.applyDimension(
                TypedValue.COMPLEX_UNIT_DIP, dp, getResources().getDisplayMetrics());
    }
}
```

本文学习笔记大部分来自大神之作： https://www.jianshu.com/p/4a77ae4cd82f
