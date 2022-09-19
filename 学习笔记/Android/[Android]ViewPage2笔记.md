

## ViewPage2笔记

[TOC]

### 概念

ViewPager2是ViewPager的升级版。

1. ViewPager2内部实现是RecyclerView，所以ViewPager2的性能更高。
2. ViewPager2可以实现竖向滑动，ViewPager只能横向滑动。
3. ViewPager2只有一个adapter，FragmentStateAdapter继承自RecyclerView.Adapter<FragmentViewHolder>。
    而ViewPager有两个adapter,FragmentStatePagerAdapter和FragmentPagerAdapter，均是继承PagerAdapter。FragmentStatePagerAdapter和FragmentPagerAdapter两者的区别是FragmentStatePagerAdapter不可以缓存，FragmentPagerAdapter可以缓存。
4. ViewPager2模式实现了懒加载，默认不进行预加载。内部是通过Lifecycle 对 Fragment 的生命周期进行管理。ViewPager会进行预加载，懒加载需要我们自己去实现

### 使用

#### 添加依赖

```groovy
dependencies {
  implementation "androidx.viewpager2:viewpager2:1.0.0"
}
```

#### 基础使用

##### ViewPage2布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

##### Adapter

ViewPager2是基于RecyclerView的，所以它使用的Adapter也是RecyclerView的Adapter。

```kotlin
class MyAdapter :RecyclerView.Adapter<MyAdapter.MHolder>(){

    private var mColorsList = ArrayList<Int>()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MHolder {
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item_page,parent,false)
        return MHolder(itemView)
    }

    override fun onBindViewHolder(holder: MHolder, position: Int) {
        holder.view.setBackgroundColor(mColorsList[position])
    }

    override fun getItemCount(): Int {
        return mColorsList.size
    }

    fun setData(colors:List<Int>){
        mColorsList.clear()
        if(colors?.isNotEmpty()){
            mColorsList.addAll(colors)
        }
        notifyDataSetChanged()
    }

    class MHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val view = itemView.findViewById<View>(R.id.view_item)
    }
}
```

item_page.xml内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent">

    <View
        android:id="@+id/view_item"
        android:background="@color/colorAccent"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

##### 绑定Adapter

```kotlin
class ViewPager2TestActivity :AppCompatActivity(){

    private val list = arrayListOf<Int>(Color.RED,Color.YELLOW,Color.BLUE,Color.GREEN)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_viewpager2test)

        val adapter = MyAdapter()
        adapter.setData(list)
        view_pager.adapter = adapter
    }
}
```

#### 基本特性

##### 支持竖直方向

除了传统的水平分页之外，ViewPager2 还支持垂直分页。您可以通过设置 ViewPager2 元素的 android:orientation 属性为其启用垂直分页：

```xml
<androidx.viewpager2.widget.ViewPager2
   xmlns:android="http://schemas.android.com/apk/res/android"
   android:id="@+id/pager"
   android:orientation="vertical" />
```

或代码设置：

```kotlin
view_pager.orientation = ViewPager2.ORIENTATION_VERTICAL
```

##### 支持从右到左

持从右到左 (RTL) 分页。系统会根据语言区域在适当的情况下自动启用 RTL 分页，不过您也可以通过设置 ViewPager2 元素的 android:layoutDirection 属性为其手动启用 RTL 分页：

```xml
<androidx.viewpager2.widget.ViewPager2
   xmlns:android="http://schemas.android.com/apk/res/android"
   android:id="@+id/pager"
   android:layoutDirection="rtl" />
```

或代码设置：

```kotlin
view_pager.layoutDirection = LayoutDirection.RTL
```

##### 支持滑动监听

使用registerOnPageChangeCallback方法注册监听：

```kotlin
view_pager.registerOnPageChangeCallback(object :ViewPager2.OnPageChangeCallback(){
    override fun onPageSelected(position: Int) {
        super.onPageSelected(position)
        Toast.makeText(this@ViewPager2TestActivity, "ccm== $position", Toast.LENGTH_SHORT).show()
    }
})
```

##### 禁止滑动

在使用ViewPager的时候想要禁止用户滑动需要重写ViewPager的onInterceptTouchEvent。而ViewPager2，我们可以直接通过setUserInputEnabled为false禁止滑动：

```kotlin
view_pager.isUserInputEnabled = false
```

##### 切换到某个page

```kotlin
view_pager.currentItem = 1
```

##### 模拟滑动切换page

ViewPager2新增了一个fakeDragBy的方法，我们可以通过fakeDragBy来移动，fakeDragBy(-10),负数是说明向下一个页面，正数表示向前一个页面滑动，但使用fakeDragBy前需要先调用beginFakeDrag方法才能生效。可以使用endFakeDrag停止；

```kotlin
btn_fake.setOnClickListener {
   view_pager.beginFakeDrag()
   view_pager.fakeDragBy(-500f)
}
btn_fake.setOnClickListener {
   view_pager.beginFakeDrag()
   if(view_pager.fakeDragBy(-310f)) view_pager.endFakeDrag()
}
```

##### Page预加载

ViewPager2中offScreenPageLimit的默认值被设置为了-1，而当offScreenPageLimit为-1的时候，使用的是RecyclerView的缓存机制。而当offScreenPageLimit大于1时，才会去实现预加载。 

```kotlin
view_pager.setOffscreenPageLimit(2)
```

#### PageTransformer

ViewPager2的Transformer功能有了很大的扩展。ViewPager2不仅可以通过PageTransformer用来设置页面动画，还可以用PageTransformer设置页面间距以及同时添加多个PageTransformer。

##### 设置页面间距

```kotlin
view_pager.setPageTransformer(MarginPageTransformer(30))
```

##### 设置页面跳转动画