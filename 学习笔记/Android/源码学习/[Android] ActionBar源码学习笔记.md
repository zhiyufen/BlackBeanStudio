# ActionBar源码学习笔记

[TOC]

最近遇到一个ActionBar的菜单弹出其选项时，需要把其选项设置为圆角的，但使用popupBackGround属性后，发现，圆角是改好的，但其点击时的波纹效果是超出其选项圆角背景的，但尝试各种方法都没无效；看来得好看看其一下其源码了； 

本文是基于Androidx appcompat1.3.1的；

## XXX

### 找到ActionBar

从AppCompatActivity中：getSupportActionBar()

```java
    @Nullable
    public ActionBar getSupportActionBar() {
        return this.getDelegate().getSupportActionBar();
    }
```

Delegate对应实现类是AppCompatDelegateImpl：

```java
//AppCompatDelegate.java
	@NonNull
    public AppCompatDelegate getDelegate() {
        if (this.mDelegate == null) {
            this.mDelegate = AppCompatDelegate.create(this, this);
        }

        return this.mDelegate;
    }
    
    @NonNull
    public static AppCompatDelegate create(@NonNull Activity activity, @Nullable AppCompatCallback callback) {
        return new AppCompatDelegateImpl(activity, callback);
    }

```

那我们找到：

```java
//AppCompatDelegateImpl.java
    public ActionBar getSupportActionBar() {
        this.initWindowDecorActionBar();
        return this.mActionBar;
    }

    private void initWindowDecorActionBar() {
        this.ensureSubDecor();
        if (this.mHasActionBar && this.mActionBar == null) {
            if (this.mHost instanceof Activity) {
                this.mActionBar = new WindowDecorActionBar((Activity)this.mHost, this.mOverlayActionBar);
            } else if (this.mHost instanceof Dialog) {
                this.mActionBar = new WindowDecorActionBar((Dialog)this.mHost);
            }

            if (this.mActionBar != null) {
                this.mActionBar.setDefaultDisplayHomeAsUpEnabled(this.mEnableDefaultActionBarUp);
            }

        }
    }
```

另：从setSupportActionBar方法，我们可以看到Toolbar的实现类是ToolbarActionBar.java，并直接代替 mActionBar，所以两者只能使用其中一个；

### WindowDecorActionBar的初始化

```java
    public WindowDecorActionBar(Activity activity, boolean overlayMode) {
        mActivity = activity;
        Window window = activity.getWindow();
        View decor = window.getDecorView();
        init(decor);
        if (!overlayMode) {
            mContentView = decor.findViewById(android.R.id.content);
        }
    }
```

其Init（）:

```java

    private void init(View decor) {
        mOverlayLayout = (ActionBarOverlayLayout) decor.findViewById(R.id.decor_content_parent);
        if (mOverlayLayout != null) {
            mOverlayLayout.setActionBarVisibilityCallback(this);
        }
        mDecorToolbar = getDecorToolbar(decor.findViewById(R.id.action_bar));
        mContextView = (ActionBarContextView) decor.findViewById(
                R.id.action_context_bar);
        mContainerView = (ActionBarContainer) decor.findViewById(
                R.id.action_bar_container);

        if (mDecorToolbar == null || mContextView == null || mContainerView == null) {
            throw new IllegalStateException(getClass().getSimpleName() + " can only be used " +
                    "with a compatible window decor layout");
        }

        mContext = mDecorToolbar.getContext();

        // This was initially read from the action bar style
        final int current = mDecorToolbar.getDisplayOptions();
        final boolean homeAsUp = (current & DISPLAY_HOME_AS_UP) != 0;
        if (homeAsUp) {
            mDisplayHomeAsUpSet = true;
        }

        ActionBarPolicy abp = ActionBarPolicy.get(mContext);
        setHomeButtonEnabled(abp.enableHomeButtonByDefault() || homeAsUp);
        setHasEmbeddedTabs(abp.hasEmbeddedTabs());

        final TypedArray a = mContext.obtainStyledAttributes(null,
                R.styleable.ActionBar,
                R.attr.actionBarStyle, 0);
        if (a.getBoolean(R.styleable.ActionBar_hideOnContentScroll, false)) {
            setHideOnContentScrollEnabled(true);
        }
        final int elevation = a.getDimensionPixelSize(R.styleable.ActionBar_elevation, 0);
        if (elevation != 0) {
            setElevation(elevation);
        }
        a.recycle();
    }
```

从这里看，init已经根据xml布局初始化各个View组件了，但xml对应是哪里？

在ActionBar的构造方法，我们知道view是来自于：window的getDecorView方法， 通过查看Activity的源码发现Window的真正实现是PhoneWindow：

```java
//PhoneWindow.java
    @Override
    public final @NonNull View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor();
        }
        return mDecor;
    }
```

```java
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            //获取根view： DecorView
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
			...
        }
        ..
    }
```

其实类DecorView是FrameLayout的派生类，也就是说是一个ViewGroup；然后调用generateLayout()获取布局文件:

```java

    protected ViewGroup generateLayout(DecorView decor) {
        // Apply data from current theme.

        TypedArray a = getWindowStyle();
		...
        if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {
            requestFeature(FEATURE_NO_TITLE);
        } else if (a.getBoolean(R.styleable.Window_windowActionBar, false)) {
            // Don't allow an action bar if there is no title.
            requestFeature(FEATURE_ACTION_BAR);
        }
 		...

        // Inflate the window decor.

        int layoutResource;
        int features = getLocalFeatures();
        .,.
        } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
            // If no other features and not embedded, only need a title.
            // If the window is floating, we need a dialog layout
            if (mIsFloating) {
                TypedValue res = new TypedValue();
                getContext().getTheme().resolveAttribute(
                        R.attr.dialogTitleDecorLayout, res, true);
                layoutResource = res.resourceId;
            } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
                layoutResource = a.getResourceId(
                        R.styleable.Window_windowActionBarFullscreenDecorLayout,
                        R.layout.screen_action_bar);
            } else {
                layoutResource = R.layout.screen_title;
            }
            // System.out.println("Title!");
        }
		...
        mDecor.startChanging();
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
		ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        if (contentParent == null) {
            throw new RuntimeException("Window couldn't find content container view");
        }
		...

        return contentParent;
    }
```

从上面代码可以看出，根据window中不同的features加载不同的布局文件，比如当features=FEATURE_ACTION_BAR时，加载的布局文件com.android.internal.R.layout.screen_action_bar，即screen_action_bar.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.android.internal.widget.ActionBarOverlayLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/decor_content_parent"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:splitMotionEvents="false"
    android:theme="?attr/actionBarTheme">
    <FrameLayout android:id="@android:id/content"
                 android:layout_width="match_parent"
                 android:layout_height="match_parent" />
    <com.android.internal.widget.ActionBarContainer
        android:id="@+id/action_bar_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        style="?attr/actionBarStyle"
        android:transitionName="android:action_bar"
        android:touchscreenBlocksFocus="true"
        android:keyboardNavigationCluster="true"
        android:gravity="top">
        <!-- ActionBar的布局-->
        <com.android.internal.widget.ActionBarView
            android:id="@+id/action_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            style="?attr/actionBarStyle" />
        <com.android.internal.widget.ActionBarContextView
            android:id="@+id/action_context_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:visibility="gone"
            style="?attr/actionModeStyle" />
    </com.android.internal.widget.ActionBarContainer>
    <com.android.internal.widget.ActionBarContainer android:id="@+id/split_action_bar"
                  android:layout_width="match_parent"
                  android:layout_height="wrap_content"
                  style="?attr/actionBarSplitStyle"
                  android:visibility="gone"
                  android:touchscreenBlocksFocus="true"
                  android:keyboardNavigationCluster="true"
                  android:gravity="center"/>
</com.android.internal.widget.ActionBarOverlayLayout>

```

从这个布局文件中可以看到，整个window的根布局为DecorView，然后DecorView里又分为ActionBar区域、Content区域和SplitActionBar区域，其中ActionBar区域放置ActionBarView或者ActionBarContextView，Content区域放置的就是我们使用Activity时自己设计的Layout，SplitActionBar区域放置的是Split形式的ActionBar；整个结构如下图：
![image](https://img-blog.csdn.net/20150318200403609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSm9iX0hlc2M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

