## [Android]TabLayout+ViewPager+Fragment

[TOC]

### 1. TabLayout

TabLayout提供了一个水平的布局用来展示Tabs, 一般都是配合ViewPager使用的，ViewPager里的Fragment随着顶部的Tab一起联动

#### 1）配置

```groovy
compile 'com.android.support:design:25.0.0'
//Androidx
implementation 'com.google.android.material:material:1.0.0'
```

### 2）TabLayout的基本属性

> android.support.design:tabBackground — 设置的背景。
>  android.support.design:tabContentStart — 相对起始位置tab的Y轴偏移量。
>  android.support.design:tabGravity — tab的布局方式，两个值GRAVITY_CENTER （内容中心显示） 和 GRAVITY_FILL （内容尽可能充满TabLayout）。
>  android.support.design:tabIndicatorColor — 设置tab指示器(tab的下划线)的颜色。
>  android.support.design:tabIndicatorHeight — 设置tab指示器(tab的下划线)的高度。
>  android.support.design:tabMaxWidth — 设置tab选项卡的最大宽度。
>  android.support.design:tabMinWidth — 设置tab选项卡的最小宽度。
>  android.support.design:tabMode — 设置布局中tab选项卡的行为模式，两个常量MODE_FIXED （固定的tab）和 MODE_SCROLLABLE（滑动的tab）。
>  android.support.design:tabPadding — 设置tab的内边距（上下左右）。
>  android.support.design:tabPaddingBottom — 设置tab的底部内边距。
>  android.support.design:tabPaddingEnd — 设置tab的右侧内边距。
>  android.support.design:tabPaddingStart — 设置tab的左侧内边距。
>  android.support.design:tabPaddingTop — 设置tab的上方内边距。
>  android.support.design:tabSelectedTextColor — 设置tab被选中时的文本颜色。
>  android.support.design:tabTextColor — 设置tab默认的文本颜色。
>  [android.support.design:tabTextAppearance](android.support.design:tabTextAppearance) — 设置tab的TextAppearance样式的引用，可以引用另一个资源，形式为“@ [+]
>  [package：] type / name”或主题属性，格式为“？[package：] type / name”。。

使用：

```xml
 <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabIndicatorColor="#eb5461"
        app:tabIndicatorHeight="0dp"
        app:tabMode="fixed"
        app:tabMaxWidth="0dp"
        app:tabGravity="fill"
        app:tabPaddingTop="0dp"
        app:tabPaddingStart="0dp"
        app:tabPaddingEnd="0dp" />
```

### 3）公共方法

> addOnTabSelectedListener（TabLayout.OnTabSelectedListener listener）
>  添加一个TabLayout.OnTabSelectedListener监听事件，当tab选择更改时，它将被调用。
>
> addTab（TabLayout.Tab tab，boolean setSelected）
>  向此布局添加选项卡。
>
> addTab（TabLayout.Tab tab，int position）
>  向此布局添加选项卡。
>
> addTab（TabLayout.Tab tab）
>  向此布局添加选项卡。
>
> addTab（TabLayout.Tab tab，int position，boolean setSelected）
>  向此布局添加选项卡。
>
> addView（View child，int index）
>  添加子视图到指定位置。
>
> addView（View child）
>  添加子视图。
>
> addView（View child，ViewGroup.LayoutParams params）
>  添加具有指定布局参数的子视图。
>
> addView（View child，int index，ViewGroup.LayoutParams params）
>  添加具有指定布局参数的子视图。
>
> clearOnTabSelectedListeners（）
>  删除所有以前添加的TabLayout.OnTabSelectedListeners。
>
> FrameLayout.LayoutParams generateLayoutParams（AttributeSet attrs）
>  根据提供的属性集返回一组新的布局参数。
>
> int getSelectedTabPosition（）
>  返回当前所选标签的位置。
>
> TabLayout.Tab getTabAt（int index）
>  返回指定位置的tab。
>
> int getTabCount（）
>  返回当前在操作栏中注册的选项卡数。
>
> int getTabGravity（）
>  返回当前的标签tab的布局方式，GRAVITY_CENTER （内容中心显示） 和 GRAVITY_FILL （内容尽可能充满TabLayout）。
>
> int getTabMode（）
>  返回tab选项卡的行为模式,MODE_FIXED* （固定的tab）和 MODE_SCROLLABLE（滑动的tab）。
>
> ColorStateList getTabTextColors（）
>  获取用于选项卡的不同状态（正常，已选择）的文本颜色。
>
> TabLayout.Tab newTab ()
>  创建并返回一个新的TabLayout.Tab。
>
> removeAllTabs（）
>  从操作栏中删除所有选项卡，并取消选择当前选项卡。
>
> removeOnTabSelectedListener（TabLayout.OnTabSelectedListener listener）
>  删除以前通过addOnTabSelectedListener（OnTabSelectedListener）添加的给定
>  TabLayout.OnTabSelectedListener，tab选中监听器。
>
> removeTab（TabLayout.Tab tab）
>  从布局中删除选项卡。
>
> removeTabAt（int position）
>  从布局中删除选项卡。
>
> setOnTabSelectedListener（TabLayout.OnTabSelectedListener listener）
>  API方法24.0.0中已弃用此方法。使用addOnTabSelectedListener（OnTabSelectedListener）和removeOnTabSelectedListener（OnTabSelectedListener）。
>
> setScrollPosition（int position，float positionOffset，boolean updateSelectedText）
>  设置选项卡的滚动位置，当标签tab显示为滚动容器（如ViewPager）的一部分时，此功能非常有用。
>  参数:
>  位置int：当前滚动位置
>  positionOffset float：表示从位置偏移的[0, 1)的值。
>  updateSelectedText boolean：是否更新文本的选择状态。。
>
> setSelectedTabIndicatorColor（int color）
>  设置选中的tab的指示器（下划线）颜色。
>
> setSelectedTabIndicatorHeight（int height）
>  设置选中的tab的指示器的高度。
>
> setTabGravity（int gravity）
>  设置TabLayout的布局方式，GRAVITY_CENTER （内容中心显示） 和 GRAVITY_FILL （内容尽可能充满TabLayout）。。
>
> setTabMode（int mode）
>  设置tab选项卡的行为模式,MODE_FIXED* （固定的tab）和 MODE_SCROLLABLE（滑动的tab）。
>
> setTabTextColors（int normalColor，int selectedColor）
>  设置用于选项卡的不同状态（常规，选定）的文本颜色。
>
> setTabTextColors（ColorStateList textColor）
>  设置用于选项卡的不同状态（常规，选定）的文本颜色。
>
> setTabsFromPagerAdapter（PagerAdapter adapter）
>  API方法23.2.0中已弃用此方法。使用setupWithViewPager（ViewPager）将TabLayout与ViewPager链接在一起。当使用该方法时，当更改PagerAdapter时，TabLayout将自动更新。
>
> setupWithViewPager（ViewPager viewPager，boolean autoRefresh）
>  将TabLayout与ViewPager链接在一起，当更改PagerAdapter时，TabLayout是否更新由autoRefresh决定。
>
> setupWithViewPager（ViewPager viewPager）
>  将TabLayout与ViewPager链接在一起。
>
> shouldDelayChildPressedState（）
>  如果此ViewGroup的子代或子孙后代按下的状态应该被延迟，则返回true。 一般来说，应该对可以滚动的容器（如List）进行此操作。 这防止当用户实际上尝试滚动内容时出现按压状态。 由于兼容性原因，默认实现返回true。 不滚动的子类通常会覆盖此方法并返回false。











### 4. TabLayout+ViewPager+Fragment

##### layout

```xml
<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/sesl_round_and_bgcolor_light">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginEnd="16dp"
        app:tabIndicatorColor="#eb5461"
        app:tabIndicatorHeight="0dp"
        app:tabMode="fixed"
        app:tabMaxWidth="0dp"
        app:tabGravity="fill"
        app:tabPaddingTop="0dp"
        app:tabPaddingStart="0dp"
        app:tabPaddingEnd="0dp" />

    <androidx.viewpager.widget.ViewPager
        android:id="@+id/pager"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"/>
</LinearLayout>
```

Activity:

```kotlin
/**
 * Activity for handler view pager, tab layout, fragment.
 */
class MyActivity: AppCompatActivity() {
    companion object {
        private const val TAG = "my_bill"
    }

    var mPagerIndex = Constant.PAGE_INDEX_REPAYMENT

    private val mViewPager by lazy {
        findViewById<ViewPager>(R.id.pager)
    }
    private val mTabLayout by lazy {
        findViewById<TabLayout>(R.id.tab_layout)
    }
    private val mPagerAdapter by lazy {
        MyBillPageAdapter(supportFragmentManager)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.my_bill_main_layout)
        // actionbar
        supportActionBar?.apply {
            setDisplayHomeAsUpEnabled(true)
            setDisplayShowHomeEnabled(false)
            setHomeButtonEnabled(true)
        }

        initPager()
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == android.R.id.home) {
            finish()
        }
        return super.onOptionsItemSelected(item)
    }

    private fun initPager() {
        mViewPager.adapter = mPagerAdapter
        mTabLayout.setupWithViewPager(mViewPager)
        mViewPager.currentItem = mPagerIndex
        initTabLayout()
    }

    private fun initTabLayout() {
        for (i in 0 until mTabLayout.tabCount) {
            val tabView = layoutInflater.inflate(R.layout.tab_item, mTabLayout, false)
            tabView.findViewById<TextView>(R.id.tab_title).setText(Constant.TAB_TITLE_RES[i])
            tabView.layoutParams = ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT)
            mTabLayout.getTabAt(i)?.apply {
                customView = tabView
            }
        }
        //mTabLayout.tabMode = TabLayout.MODE_FIXED
        mTabLayout.addOnTabSelectedListener(object: TabLayout.OnTabSelectedListener {
            override fun onTabSelected(tab: TabLayout.Tab?) {
                tab?.apply {
                    SAappLog.dTag(TAG, "position = $position")
                }
            }
            override fun onTabReselected(tab: TabLayout.Tab?) {
            }

            override fun onTabUnselected(tab: TabLayout.Tab?) {
            }
        })
    }
}


/**
 * Adapter for my bill pager.
 */
class MyBillPageAdapter(fm: FragmentManager): FragmentPagerAdapter(fm) {
    override fun getItem(position: Int): Fragment? {
        return MyFragment.newInstance(position)
    }

    override fun getCount(): Int {
        return Constant.TAB_TITLE_RES.size
    }

    override fun getPageTitle(position: Int): CharSequence? {
        return SReminderApp.getInstance().getString(Constant.TAB_TITLE_RES[position])
    }
}

class MyBillFragment: Fragment() {

    companion object {
        fun newInstance(tabType: Int): MyBillFragment {
            val args = Bundle()
            args.putInt(MyBillConstant.ARG_TAB_TYPE, tabType)
            val fragment = MyBillFragment()
            fragment.arguments = args
            return fragment
        }
    }
    ...
}
```

