## Android View组件的那些事

[TOC]

### 1. ProgressBar

ProgressBar是Android下的进度条，也是为数不多的直接继承于View类的控件,直接子类有AbsSeekBar和ContentLoadingProgressBar，其中AbsSeekBar的子类有SeekBar和RatingBar.

rogressBar有两个进度，一个是android:progress，另一个是android:secondaryProgress。后者主要是为缓存需要所涉及的,比如看视频时缓存的进度是多少；

ProgressBar分为确定的和不确定的，不确定的ProgressBar是由属性android:indeterminate来控制的，设置为true的，那么ProgressBar就可能是圆形的滚动条或者水平的滚动条（由样式决定）。默认情况下，如果是水平进度条，那么就是确定的。

其样式有：

```java
style="@android:style/Widget.ProgressBar"
style="@android:style/Widget.ProgressBar.Large"	//大环形进度条
style="@android:style/Widget.ProgressBar.Small"	//小环形进度条
style="@android:style/Widget.ProgressBar.Inverse"	//普通大小的环形进度条
style="@android:style/Widget.ProgressBar.Large.Inverse"	//大环形进度条
style="@android:style/Widget.ProgressBar.Small.Inverse"	//小环形进度条
style="@android:style/Widget.ProgressBar.Small.Title"	//标题栏环形进度条
style="@android:style/Widget.ProgressBar.Horizontal"	//水平进度条
```

也可以使用系统样式：

```java
style="?android:attr/progressBarStyle"
style="?android:attr/progressBarStyleHorizontal" //横向
style="?android:attr/progressBarStyleInverse"
style="?android:attr/progressBarStyleLarge"  //大圆形
style="?android:attr/progressBarStyleLargeInverse"
style="?android:attr/progressBarStyleSmall" //小圆形
style="?android:attr/progressBarStyleSmallInverse"
style="?android:attr/progressBarStyleSmallTitle"
```

其它属性：

```java
android:max="100"	//进度条最大值
android:progress="0"	//进度条初始值
android:secondaryProgress="0"	//二级进度值，值介于0到max。该进度在主进度和背景之间。比如用于网络播放视频时，二级进度用于表示缓冲进度，主进度用于表示播放进度。
android:animationResolution	//超时的动画帧之间的毫秒 ；必须是一个整数值,如“100”。（已经被舍弃了，现在都不用了。）
android:progressDrawable=""	//设置进度条轨道对应的drawable对象
//下面为不确定时设置
android:indeterminate=""	//是否允许使用不确定模式，该属性设置为true，表示设置进度条不精确显示进度，在不确定模式下，进度条动画无限循环
android:indeterminateDrawable=""	//定义不确定模式是否可拉
android:indeterminateDuration=""	//时间不定的动画
android:indeterminateBehavior=""	//定义当进度达到最大时，不确定模式的表现；该值必须为repeat或者cycle，repeat表示进度从0重新开始；cycle表示进度保持当前值，并且回到0
android:indeterminateOnly=""	//限制为不定模式
android:indeterminateTint="" //进度条的颜色
android:indeterminateTintMode="" //颜色模式
```

比如：

```xml
        <ProgressBar
            android:id="@+id/retry_progress"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:progressDrawable="@drawable/progress_bar_bg"
            android:indeterminate="true"
            android:indeterminateTint="#fafafa" //修改进度条的颜色
            android:indeterminateTintMode="src_atop" //颜色模式
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"/>
```

更多属性参考官方文档： https://developer.android.com/reference/android/widget/ProgressBar

查看系统中水平进度条的风格文件：

```xml
<style name="Widget.ProgressBar.Horizontal">
    <item name="indeterminateOnly">false</item>
    <item name="progressDrawable">@drawable/progress_horizontal</item>
    <item name="indeterminateDrawable">@drawable/progress_indeterminate_horizontal</item>
    <item name="minHeight">20dip</item>
    <item name="maxHeight">20dip</item>
    <item name="mirrorForRtl">true</item>
</style>
```

其中@drawable/progress_horizontal就是设置进度条背景：

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <-- 进度条背景色 -->
    <item android:id="@android:id/background"> 
        <shape>
            <corners android:radius="5dip" />
            <gradient
                    android:startColor="#ff9d9e9d"
                    android:centerColor="#ff5a5d5a"
                    android:centerY="0.75"
                    android:endColor="#ff747674"
                    android:angle="270"
            />
        </shape>
    </item>
     <-- 第二进度条 -->
    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                        android:startColor="#80ffd300"
                        android:centerColor="#80ffb600"
                        android:centerY="0.75"
                        android:endColor="#a0ffcb00"
                        android:angle="270"
                />
            </shape>
        </clip>
    </item>
    <-- 第一进度条 -->
    <item android:id="@android:id/progress"> 
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                        android:startColor="#ffffd300"
                        android:centerColor="#ffffb600"
                        android:centerY="0.75"
                        android:endColor="#ffffcb00"
                        android:angle="270"
                />
            </shape>
        </clip>
    </item>
    
</layer-list>
```

也就是我们可以模仿上面来自定义自己的进度条；

### 2. Shape 背景图

在Android开发中，使用shape可以很方便的帮我们画出想要的背景，相对于png图片来说，使用shape可以减少安装包的大小，而且能够更好的适配不同的手机。

Shape的属性，官网说明：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    <--分别是：矩形（rectangle）、椭圆或圆（oval）、线（line）、圆环（ring）-->
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <-- 圆角大小 -->
    <corners
        android:radius="integer" // 4个角的圆角大小(都相同时)
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <-- 渐变-->
    <gradient
        android:angle="integer" // 渐变角度：0:左到右;90:下到上;180:右到左;270:上到下
        android:centerX="integer" // 表示渐变的X轴起始位置，范围0-1，0.5表示圆心。
        android:centerY="integer" // 表示渐变的Y轴起始位置，范围0-1，0.5表示圆心。
        android:centerColor="integer" //
        android:endColor="color" // 渐变结束颜色
        android:gradientRadius="integer" //
        android:startColor="color" // 渐变起始颜色
        android:type=["linear" | "radial" | "sweep"] // 渐变类型, 
        // linear 线性渐变，默认的渐变类型
        // radial 放射渐变，设置该项时，android:gradientRadius也必须设置
        // sweep 扫描性渐变
        android:useLevel=["true" | "false"] />
    <-- 内边距设置 -->
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <-- 大小 -->
    <size
        android:width="integer"  // 宽度
        android:height="integer" // 高度
        /> 
    <-- 填充颜色 -->
    <solid
        android:color="color" />
    <-- 边框设置-->
    <stroke
        android:width="integer" // 边框宽度
        android:color="color" // 边框颜色
        android:dashWidth="integer" // 虚线宽度
        android:dashGap="integer" // 虚线间距宽度
        />
</shape>
```

更多属性，请前往官网： https://developer.android.com/guide/topics/resources/drawable-resource.html#Shape

代码动态设置Shape的：

```java
GradientDrawable mItemContainViewBackground =
                        (GradientDrawable) mItemContainView.getBackground();
mItemContainViewBackground.setColor(ContextCompat.getColor(
                                mItemContainView.getContext(),
                                R.color.news_feed_selected_channel_not_editable));
```



可与 layer-list结合使用，组成不同的效果图：https://www.cnblogs.com/tianzhijiexian/p/3889770.html

### 3. Ripple(波纹效果)

为了button会自带有Ripple点击效果。但是往往开发者需要修改点击效果，从而修改**android:backgroud**, 这时候加入Ripple效果就会改变Button点击时背景带有波纹效果。所以使用Ripple的关键就在**android:backgroud**中设置。

```
android:background="?android:attr/selectableItemBackground" //有边界波纹
android:background="?android:attr/selectableItemBackgroundBorderless" //超出边界波纹（圆形）API要求21以上
```

设置ripple 标签的drawble:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"        
       android:color="?android:colorPrimaryDark">
</ripple>
```

那边我们在实现一个带圆角时的Button时，怎么让其带波纹效果：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:attr/colorControlHighlight">
    <item>
        <shape android:shape="rectangle">
            <solid android:color="#EDEDED"/> //Button的固有的颜色
            <corners android:radius="22dp"/> //圆角率
            <size
                android:width="92dp"
                android:height="36dp"/>
        </shape>
    </item>
</ripple>
```



### 4. TextView

| xml属性                         | 说明                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| android:autoLink                | 设置是否当文本为URL链接/email/电话号码/map时，文本显示为可点击的链接。可选值(none/web /email/phone/map/all) |
| android:autoTex                 | 如果设置，将自动执行输入值的拼写纠正。此处无效果，在显示输入法并输入的时候起作用。 |
| android:bufferType              | 指定getText()方式取得的文本类别。选项editable 类似于StringBuilder可追加字符，也就是说getText后可调用append方法设置文本内容。spannable 则可在给定的字符区域使用样式，参见这里1、这里2。 |
| android:capitalize              | 设置英文字母大写类型。此处无效果，需要弹出输入法才能看得到，参见EditView此属性说明。 |
| android:textIsSelectable=”true” | 设置让TextView文本可被选择                                   |
| android:editable=”true”         | 设置是否可编辑。                                             |
| android:gravity                 | 设置文本位置，如设置成“center”，文本将居中显示。             |
| android:includeFontPadding      | 设置文本是否包含顶部和底部额外空白，默认为true。             |
| android:maxLength               | 限制显示的文本长度，超出部分不显示。                         |
| android:lineSpacingExtra        | 设置行间距。                                                 |
| android:lineSpacingMultiplier   | 设置行间距的倍数。如”1.2”                                    |
| android:scrollHorizontally      | 设置文本超出TextView的宽度的情况下，是否出现横拉条。         |
| android:selectAllOnFocus        | 如果文本是可选择的，让他获取焦点而不是将光标移动为文本的开始位置或者末尾位置。 TextView中设置后无效果,在EditText可查。 |
| android:shadowColor             | 指定文本阴影的颜色，需要与shadowRadius一起使用               |
| android:shadowDx                | 设置阴影横向坐标开始位置                                     |
| android:shadowDy                | 设置阴影纵向坐标开始位置                                     |
| android:shadowRadius            | 设置阴影的半径。设置为0.1就变成字体的颜色了，一般设置为3.0的效果比较好。 |
| android:singleLine              | 设置单行显示。如果和layout_width一起使用，当文本不能全部显示时，后面用“…”来表示。如果不设置singleLine或者设置为false，文本将自动换行 |
| android:text                    | 设置显示文本。                                               |
| android:textAppearance          | 设置文字外观。如 “?android:attr/textAppearanceLargeInverse”这里引用的是系统自带的一个外观，?表示系统是否有这种外观，否则使用默认的外观。可设置的值如下：<br />/textAppearanceButton<br />/textAppearanceInverse<br />/textAppearanceLarge<br />/textAppearanceLargeInverse<br />/textAppearanceMedium<br />/textAppearanceMediumInverse<br />/textAppearanceSmall<br />/textAppearanceSmallInverse |
| android:textColor               | 设置文本颜色                                                 |
| android:textColorHighlight      | 被选中文字的底色，默认为蓝色                                 |
| android:textColorHint           | 设置提示信息文字的颜色，默认为灰色。与hint一起使用。         |
| android:textColorLink           | 文字链接的颜色.                                              |
| android:textScaleX              | 设置文字之间间隔，默认为1.0f。                               |
| android:textSize                | 设置文字大小，推荐度量单位”sp”，如”15sp”                     |
| android:textStyle               | 设置字形[bold(粗体) 0, italic(斜体) 1, bolditalic(又粗又斜) 2] 可以设置一个或多个，用“ |
| android:typeface                | 设置文本字体，必须是以下常量值之一：normal 0, sans 1, serif 2, monospace(等宽字体) 3] |
| android:height                  | //设置文本区域的高度，支持度量单位：px(像素)/dp/sp/in/mm(毫米) |
| android:maxHeight               | 设置文本区域的最大高度                                       |
| android:minHeight               | 设置文本区域的最小高度                                       |
| android:width                   | 设置文本区域的宽度，支持度量单位：px(像素)/dp/sp/in/mm(毫米)，与layout_width 的区别看这里。 |
| android:maxWidth                | 设置文本区域的最大宽度                                       |
| android:minWidth                | 设置文本区域的最小宽度                                       |
|                                 |                                                              |

更多属性可参考： [TextView属性详解](https://blog.csdn.net/jaycee110905/article/details/8762238)

### 5. ConstrainLayout用法 

https://www.jianshu.com/p/106e4282a383

### 6. View 设置外形轮廓

```java
getCurrentView().setOutlineProvider(new ViewOutlineProvider() {
            @Override
            public void getOutline(View view, Outline outline) {
                //card_common_background_radius = 32dp
                float bottom = view.getContext().getResources().getDimension(R.dimen.card_common_background_radius);
                outline.setRoundRect(0, 0, view.getWidth(), view.getHeight(), bottom);
            }
        });
```

### 7. AlertDialog

简单用法：

```java
private AlertDialog.Builder buildPermissionAlertDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        View layout = LayoutInflater.from(getContext()).inflate(R.layout.content_view, null);
        TextView content = layout.findViewById(R.id.textview);
        content.setText(Html.fromHtml(getString(R.string.content_popup)));
        content.setMovementMethod(LinkMovementMethod.getInstance());
         builder.setView(layout);
            builder.setPositiveButton(R.string.turn_on, (dialog, which) -> //do something);
            builder.setNegativeButton(R.string.btn_cancel, (dialog, which) -> {
                // doNothing.
            });
            return builder;
        }
```

XML:

```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingTop="@dimen/dialog_margin_top"
    android:paddingBottom="@dimen/dialog_margin_top">

    <TextView
        android:id="@+id/textview"
        style="@style/WinsetDescriptionTextStyle"
        android:layout_marginTop="0dp"
        android:layout_marginBottom="0dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:clickable="true"
        android:focusable="true"
        android:gravity="left"
        android:scrollbars="vertical" />
</ScrollView>
```

### 8. 图层 layer-list

`LayerDrawable` 是管理其他可绘制对象阵列的可绘制对象。列表中的每个可绘制对象均按照列表顺序绘制，列表中的最后一个可绘制对象绘于顶部。每个可绘制对象由单一 `` 元素内的 `` 元素表示。

layer-list 的大致原理类似 RelativeLayout（或者FrameLayout） ，也是一层层的叠加 ，后添加的会覆盖先添加的。在 layer-list 中可以通过 控制后添加图层距离最底部图层的 左上右下的四个边距等属性，得到不同的显示效果。

因 layer-list 创建出来的也是 drawable 资源，所以，同 shape selector 一样，都是定义在 res 中的 drawable 文件夹中，也是一个 xml 文件。使用的时候，同shape selector , 布局文件中使用 @drawable/ xxx 引用, 代码中使用 R.drawable.xxx 引用。

**语法：**

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@[package:]drawable/drawable_resource" //可绘制对象资源
        android:id="@[+][package:]id/resource_name" // id
        android:top="dimension" //顶部偏移（像素）
        android:right="dimension" // 右边偏移（像素）
        android:bottom="dimension" //底部偏移（像素）
        android:left="dimension" //左边偏移（像素）。
     />
</layer-list>
```

默认情况下，所有可绘制项都会缩放以适应包含视图的大小。因此，将图像放在图层列表中的不同位置可能会增大视图的大小，并且有些图像会相应地缩放。为避免缩放列表中的项目，请在 `` 元素内使用 `` 元素指定可绘制对象，并且对某些不缩放的项目（例如 `"center"`）定义重力。例如，以下  < Bitmap> 定义缩放以适应其容器视图的项目：

可缩放图片：

```xml
<item android:drawable="@drawable/image" />
```

为避免缩放，以下示例使用重力居中的 < Bitmap> 元素：

```xml
<item>
  <bitmap android:src="@drawable/image"
          android:gravity="center" />
</item>
```

### 9. 自带自缀的EditText

```kotlin
/**
 * A custom edit text which have pre fixed text;
 * ex: we need always show "￥" when input amount of money.
 */
class PrefixedEditText(context: Context, attrs: AttributeSet?, defStyleAttr: Int)
    : AppCompatEditText(context, attrs, defStyleAttr) {
    private var mPrefixString: String? = null
    private var mOriginPaddingLeft = -1

    init {
        val ta = context.obtainStyledAttributes(attrs, R.styleable.PrefixedEditText)
       mPrefixString = ta.getString(R.styleable.PrefixedEditText_pretext)
       ta.recycle()
    }

    constructor(context: Context) : this(context, null)
    constructor(context: Context, attrs: AttributeSet?): this(context, attrs, androidx.appcompat.R.attr.editTextStyle)

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        calculatePrefixTextPadding()
    }

    private fun calculatePrefixTextPadding() {
        if (mPrefixString.isNullOrEmpty() || mOriginPaddingLeft >= 0) {
            return
        } else {
            val length = mPrefixString!!.length
            if (length <= 0) return
            val paddingLeft = paint.measureText(mPrefixString).toDouble()
            mOriginPaddingLeft = compoundPaddingLeft
            setPadding((mOriginPaddingLeft + paddingLeft).toInt(), paddingTop, paddingRight, paddingBottom)
        }
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        canvas.drawText(mPrefixString!!, mOriginPaddingLeft.toFloat(), getLineBounds(0, null).toFloat(), paint)
    }
}
```

### 10. Paint字体的使用

在Android SDK中使用Typeface类来定义字体，能够经过经常使用字体类型名称进行设置，如设置默认黑体： canvas;

  * Typeface.DEFAULT //常规字体类型 字体
  * Typeface.DEFAULT_BOLD //黑体字体类型 spa
  * Typeface.MONOSPACE //等宽字体类型 code
  * Typeface.SANS_SERIF //sans serif字体类型 对象
  * Typeface.SERIF //serif字体类型 图片

```java
Paint mp = new paint();
mp.setTypeface(Typeface.DEFAULT_BOLD)
```

除了字体类型设置以外，还能够为字体类型设置字体风格，如设置粗体：

```java
Paint mp = new Paint();
//String familyName = “宋体”;
//Typeface font = Typeface.create(familyName,Typeface.BOLD);
Typeface font = Typeface.create(Typeface.SANS_SERIF, Typeface.BOLD);
p.setTypeface( font );
```

经常使用的字体风格名称还有： 

  * Typeface.BOLD //粗体 字符串

  * Typeface.BOLD_ITALIC //粗斜体

  * Typeface.ITALIC //斜体

  * Typeface.NORMAL //常规j

可是有时上面那些设置在绘图过程当中是不起做用的，因此还有以下设置方式:

```
Paint mp = new Paint();
mp.setFakeBoldText(true); //true为粗体，false为非粗体
mp.setTextSkewX(-0.5f); //float类型参数，负数表示右斜，整数左斜
mp.setUnderlineText(true); //true为下划线，false为非下划线
mp.setStrikeThruText(true); //true为删除线，false为非删除线
```



相关例子，请参考：

https://blog.csdn.net/android_cmos/article/details/80033784

https://blog.csdn.net/HJsir/article/details/82153428