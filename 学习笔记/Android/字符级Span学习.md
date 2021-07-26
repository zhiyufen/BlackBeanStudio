字符级Span学习

[TOC]

本文笔记大部分来自：

https://www.jianshu.com/p/be0d79b9d5e6

https://blog.csdn.net/wbwjx/article/details/53179209

http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0305/2535.html



给TextView设置字符级别的Span。如果一个Span想要影响段落层次的文本格式，则需要继承CharacterStyle。

### 1. 主要规则：

- 如果一个Span影响字符级的文本格式，则继承[CharacterStyle](https://developer.android.com/reference/android/text/style/CharacterStyle.html)。
- 如果一个Span影响段落层次的文本格式，则实现[ParagraphStyle](https://developer.android.com/reference/android/text/style/ParagraphStyle.html)
- 如果一个Span修改字符级别的文本外观，则实现[UpdateAppearance](https://developer.android.com/reference/android/text/style/UpdateAppearance.html)
- 如果一个Span修改字符级文本度量|大小，则实现[UpdateLayout](https://developer.android.com/reference/android/text/style/UpdateLayout.html)

#### 1.1 CharacterStyle

CharacterStyle是个抽象类，字符级别的Span都需要继承这个类，这个类里面有一个抽象方法：

```java
public abstract void updateDrawState(TextPaint tp);
```

通过实现该方法来改变TextPaint的属性来得到不同的展现形式；

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333Mb9-0.png)

一个CharacterStyle类型的Span对象只能给一个Spaned片段使用，如果想这个Span给多个片段使用可以使用wrap方法来复制属性相同的新对象。

```java
    /**
     * A given CharacterStyle can only applied to a single region of a given
     * Spanned.  If you need to attach the same CharacterStyle to multiple
     * regions, you can use this method to wrap it with a new object that
     * will have the same effect but be a distinct object so that it can
     * also be attached without conflict.
     */
    public static CharacterStyle wrap(CharacterStyle cs) {
        if (cs instanceof MetricAffectingSpan) {
            return new MetricAffectingSpan.Passthrough((MetricAffectingSpan) cs);
        } else {
            return new Passthrough(cs);
        }
    }
```

通过看Passthrough，其实只代理一下CharacterStyle而已，可以理解成伪复制；

#### 1.2 UpdateAppearance

如果一个Span修改字符级别的文本外观，则实现UpdateAppearance。

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333H3B-2.png)

#### 1.3 UpdateLayout

如果一个Span修改字符级文本度量|大小，则实现[UpdateLayout](https://developer.android.com/reference/android/text/style/UpdateLayout.html)；

在Android源码中，只有MetricAffectingSpan实现了UpdateLayout接口。

![](http://www.jcodecraeer.com/uploads/20150305/1425485348424809.png)

#### 1.4 ParagraphStyle

如果一个Span影响段落层次的文本格式，则实现[ParagraphStyle](https://developer.android.com/reference/android/text/style/ParagraphStyle.html)

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333I032-1.png)



CharacterStyle, ParagraphStyle, UpdateAppearance和UpdateLayout都是一个空接口.我们要实现自定义Span,则只有继承自其子类,一般是MetricAffectingSpan,ReplacementSpan,LineBackgroundSpan; 

### 2. 工作原理

#### 2.1 布局（Layout）

当你给一个TextView设置文本时，它使用基类[布局](https://developer.android.com/reference/android/text/Layout.html)去管理文本的渲染。
布局类包含一个布尔`mSpannedText`:真，当文本是一个[Spanned](https://developer.android.com/reference/android/text/Spanned.html)的实例时（[SpannableString](https://developer.android.com/reference/android/text/SpannableString.html)实现[Spanned](https://developer.android.com/reference/android/text/Spanned.html)）。这个类只处理[ParagraphStyle](https://developer.android.com/reference/android/text/style/ParagraphStyle.html) Spans。
[draw](https://developer.android.com/reference/android/text/Layout.html#draw(android.graphics.Canvas, android.graphics.Path, android.graphics.Paint, int)方法调用了其它两个方法：

- **drawBackground**
  对于文本的每一行，如果有一个[LineBackgroundSpan](https://developer.android.com/reference/android/text/style/LineBackgroundSpan.html)用于当前行，[LineBackgroundSpan#drawBackground](https://developer.android.com/reference/android/text/style/LineBackgroundSpan.html#drawBackground(android.graphics.Canvas, android.graphics.Paint, int, int, int, int, int, java.lang.CharSequence, int, int, int))方法将被调用。
- **drawText**
  对于文本的每一行，它计算[LeadingMarginSpan](https://developer.android.com/reference/android/text/style/LeadingMarginSpan.html)和[LeadingMarginSpan2](https://developer.android.com/reference/android/text/style/LeadingMarginSpan.LeadingMarginSpan2.html)，并调用[LeadingMarginSpan＃drawLeadingMargin](https://developer.android.com/reference/android/text/style/LeadingMarginSpan.html#drawLeadingMargin(android.graphics.Canvas, android.graphics.Paint, int, int, int, int, int, java.lang.CharSequence, int, int, boolean, android.text.Layout))方法当它是必要的时候。这也用于确定文本对齐。最后，如果当前行是跨行的，布局将调用 TextLine#draw方法（每一行都会创建一个TextLine对象）。

#### 2.2 文本行(TextLine)

[android.text.TextLine](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/TextLine.java)的文档这么说：代表一行样式的文字，用于测量视觉顺序和为了渲染。
TextLine类包含3个Spans的集合：

- MetricAffectingSpan set
- CharacterStyle set
- ReplacementSpan set

其中有趣的方法：TextLine#handleRun，这也是所有的Spans用来渲染文本的。相对于Span的类型，TextLine调用：

- [CharacterStyle#updateDrawState](http://flavienlaurent.com/blog/2014/01/31/spans/)方法更改MetricAffectingSpan和CharacterStyle两个Spans的TextPaint配置。
- TextLine#handleReplacement方法处理ReplacementSpan。它调用[Replacement#getSize](https://developer.android.com/reference/android/text/style/ReplacementSpan.html#getSize(android.graphics.Paint, java.lang.CharSequence, int, int, android.graphics.Paint.FontMetricsInt))得到replacement的宽度，如果它需要更新字体规格最终会调用[Replacement#draw](https://developer.android.com/reference/android/text/style/ReplacementSpan.html#draw(android.graphics.Canvas, java.lang.CharSequence, int, int, float, int, int, int, android.graphics.Paint)

#### 2.3 字体规格（Font Metrics）

看图说话：

![](https://img-blog.csdn.net/20161115224034363)

![](http://www.jcodecraeer.com/uploads/20150305/1425485349832234.png)

![](https://upload-images.jianshu.io/upload_images/5734256-64f218ab8594bb3a.png?imageMogr2/auto-orient/strip|imageView2/2/w/839/format/webp)

`FontMetrics`是`Paint`的内部类,里面包含了一些关于字体的常量.

其中`Baseline(基线)`,`ascent（上坡度)`,`descent（下坡度)`,`leading（行间距)`,这些常量集合`canvas`使得我们的绘制工作变得更加的自由; 

上图三，似乎ascent包含了descent的值？

而我们绘制文字的时候一般使用的是`TextPaint`这个继承自`Paint`的类

```java
//获取文本宽度
TextPaint textPaint = new TextPaint();
paint.setTextSize(size);//设置字体大小
paint.setTypeface(Typeface.xx);//设置字体
float width = Layout.getDesiredWidth(str,textPaint);
```



### 3. 各个Span介绍

#### 3.1 BulletSpan

BulletSpan影响段落层次的文本格式。它可以给段落的开始处加上项目符号。

```java
/**
* gapWidth:项目符号和文本之间的间隙
* color: 项目符号的颜色，默认为透明
*/
//创建一个黑色的BulletSpan，间隙为15px
span = new BulletSpan(15, Color.BLACK);
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333L944-3.png)

#### 3.2 QuoteSpan

QuoteSpan影响段落层次的文本格式。它可以给一个段落加上垂直的引用线。

```
/**
* color: 垂直的引用线颜色，默认是蓝色
*/
//创建一个红色的引用
span = new QuoteSpan(Color.RED);
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333JT8-4.png)

#### 3.3 AlignmentSpan.Standard

AlignmentSpan.Standard影响段落层次的文本格式。它可以把段落的每一行文本按正常、居中、相反的方式对齐。

```java
/**
* align: 对齐方式
*/
//居中对齐的段落
span = new AlignmentSpan.Standard(Layout.Alignment.ALIGN_CENTER);
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333I019-5.png)

#### 3.4 UnderlineSpan

UnderlineSpan影响字符级的文本格式。它可以为字符集加上下划线，归功于[Paint#setUnderlineText(true)](https://developer.android.com/reference/android/graphics/Paint.html#setUnderlineText(boolean))。

```java
//下划线
span = new UnderlineSpan();
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333H040-6.png)

#### 3.5 StrikethroughSpan

StrikethroughSpan影响字符级的文本格式。它可以给字符集加上删除线，归功于[Paint#setStrikeThruText(true))](https://developer.android.com/reference/android/graphics/Paint.html#setStrikeThruText(boolean)。

```java
//删除线
span = new StrikethroughSpan();)
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333K4S-7.png)

#### 3.6 SubscriptSpan

SubscriptSpan影响字符级的文本格式，它可以通过减小[TextPaint#baselineShift](https://developer.android.com/reference/android/text/TextPaint.html#baselineShift)给字符集加下标。

```
//下标
span = new SubscriptSpan();
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333KH2-8.png)

#### 3.7 SuperscriptSpan

SuperscriptSpan影响字符级的文本格式。它可以通过增加[TextPaint#baselineShift ](https://developer.android.com/reference/android/text/TextPaint.html#baselineShift)给字符集加上标。

```java
//上标
span = new SuperscriptSpan();
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333JJ3-9.png)

#### 3.8 BackgroundColorSpan

[android.text.style.BackgroundColorSpan](https://developer.android.com/reference/android/text/style/BackgroundColorSpan.html)
BackgroundColorSpan影响字符级的文本格式。它可以给字符集加上背景颜色。

```java
/**
* color: 背景颜色
*/
//设置字符背景颜色
span = new BackgroundColorSpan(Color.GREEN);
```

![BackgroundColorSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333M2P-10.png)

#### 3.9 ForegroundColorSpan

[android.text.style.ForegroundColorSpan](https://developer.android.com/reference/android/text/style/ForegroundColorSpan.html)
ForegroundColorSpan影响字符级的文本格式，它可以设置字符集的前景颜色也即文字颜色。

```java
/**
* color: 前景颜色
*/
//设置红色的前景
span = new ForegroundColorSpan(Color.RED);
```

![ForegroundColorSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333I164-11.png)

ForegroundColorSpan的效果图

#### 3.10 ImageSpan

[android.text.style.ImageSpan](https://developer.android.com/reference/android/text/style/ImageSpan.html)
ImageSpan影响字符级的文本格式。它可以生成图像字符。这是为数不多的文档齐全的Span所以enjoy it!

```java
/**
* Context: 上下文
* resourceId: 图像资源id
*/
//用一个小图像代替字符
span = new ImageSpan(this, R.drawable.pic1_small);
```

![ImageSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333M256-12.png)

ImageSpan的效果图

#### 3.11 StyleSpan

[android.text.style.StyleSpan](https://developer.android.com/reference/android/text/style/StyleSpan.html)
StyleSpan影响字符级的文本格式，它可以给字符集设置样式（blod、italic、normal）。

```java
//设置bold+italic的字符样式
span = new StyleSpan(Typeface.BOLD | Typeface.ITALIC);
```

![StyleSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333M014-13.png)

StyleSpan的效果图

#### 3.12 TypefaceSpan

[android.text.style.TypefaceSpan](https://developer.android.com/reference/android/text/style/TypefaceSpan.html)
TypefaceSpan影响字符级的文本格式。它可以给字符设置字体集（monospace、serif等）。

```java
//设置serif family
span = new TypefaceSpan("serif");
```

![TypefaceSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333KM5-14.png)



#### 3.13 TextAppearanceSpan

[android.text.style.TextAppearanceSpan](https://developer.android.com/reference/android/text/style/TextAppearanceSpan.html)
TextAppearanceSpan影响字符级的文本格式。它可以给字符集设置外观（appearance）。

```java
/**
* TextAppearanceSpan(Context context, int appearance, int colorList)
*                     context: 上下文
*                     appearance：appearance资源id（例如：android.R.style.TextAppearance_Small）
*                     colorList：文本的颜色资源id（例如：android.R.styleable.Theme_textColorPrimary）
*
* TextAppearanceSpan(String family, int style, int size, ColorStateList color, ColorStateList linkColor)
* family：字体family
* style：描述样式（例如：android.graphics.Typeface）
* size：文字大小
* color：文字颜色
* linkColor：连接文本的颜色
*/
//设置serif family
span = new TextAppearanceSpan(this/*a context*/, R.style.SpecialTextAppearance);
```

```xml
<-- style.xml -->
<style name="SpecialTextAppearance" parent="@android:style/TextAppearance">
<item name="android:textColor">@color/color1</item>
<item name="android:textColorHighlight">@color/color2</item>
<item name="android:textColorHint">@color/color3</item>
<item name="android:textColorLink">@color/color4</item>
<item name="android:textSize">28sp</item>
<item name="android:textStyle">italic</item>
</style>
```

![TextAppearanceSpan效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333H230-15.png)



#### 3.14 AbsoluteSizeSpan

[android.text.style.AbsoluteSizeSpan](https://developer.android.com/reference/android/text/style/AbsoluteSizeSpan.html)
AbsoluteSizeSpan影响字符级的文本格式。它可以设置一个字符集的绝对文字大小。

```java
/**
* size: 大小
* dip: false，size单位为px，true，size单位为dip（默认为false）。
*/
//设置文字大小为24dp
span = new AbsoluteSizeSpan(24, true);
```

![AbsoluteSizeSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333H394-16.png)



#### 3.16 RelativeSizeSpan

[android.text.style.RelativeSizeSpan](https://developer.android.com/reference/android/text/style/RelativeSizeSpan.html)
RelativeSizeSpan影响字符水平的文本格式。它可以设置字符集的文本大小。

```java
//设置文字大小为大2倍
span = new RelativeSizeSpan(2.0f);
```

![RelativeSizeSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333H936-17.png)



#### 3.17 ScaleXSpan

[android.text.style.ScaleXSpan](https://developer.android.com/reference/android/text/style/ScaleXSpan.html)
ScaleXSpan印象字符集的文本格式。它可以在x轴方向上缩放字符集。

```java
//设置水平方向上放大3倍
span = new ScaleXSpan(3.0f);
```

![ScaleXSpan的效果图](http://www.jcodecraeer.com/uploads/allimg/150305/00333K463-18.png)



#### 3.18 MaskFilterSpan

[android.text.style.MaskFilterSpan](https://developer.android.com/reference/android/text/style/MaskFilterSpan.html)
MaskFilterSpan影响字符集文本格式。它可以给字符集设置[android.graphics.MaskFilter](https://developer.android.com/reference/android/graphics/MaskFilter.html)。
**警告：BlurMaskFilter不支持硬件加速**

```
//模糊字符集
span = new MaskFilterSpan(new BlurMaskFilter(density*2, BlurMaskFilter.Blur.NORMAL));
//浮雕字符集
span = new MaskFilterSpan(new EmbossMaskFilter(new float[] { 1, 1, 1 }, 0.4f, 6, 3.5f));
```

MaskFilterSpan的效果图: BlurMaskFilter

![MaskFilterSpan的效果图: BlurMaskFilter](http://www.jcodecraeer.com/uploads/allimg/150305/00333G233-19.png)



MaskFilterSpan的效果图: EmbossMaskFilter

![MaskFilterSpan的效果图: EmbossMaskFilter](http://www.jcodecraeer.com/uploads/allimg/150305/00333KT5-20.png)

### 4. Spans 进阶

#### 4.1 动画

##### 4.1.1 变色动画

以修改前景色为例进行添加动画：

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333La8-21.gif)

ForegroundColorSpan里的mColor为只读，所以只能编写MutableForegroundColorSpan来继承它；

```java
public class MutableForegroundColorSpan extends ForegroundColorSpan
{
    private int mAlpha = 255;
    private int mForegroundColor;
    public MutableForegroundColorSpan(int alpha, int color)
    {
        super(color);
        mAlpha = alpha;
        mForegroundColor = color;
    }
    public MutableForegroundColorSpan(Parcel src)
    {
        super(src);
        mForegroundColor = src.readInt();
        mAlpha = src.readInt();
    }
    public void writeToParcel(Parcel dest, int flags)
    {
        super.writeToParcel(dest, flags);
        dest.writeInt(mForegroundColor);
        dest.writeFloat(mAlpha);
    }
    @Override
    public void updateDrawState(TextPaint ds)
    {
        ds.setColor(getForegroundColor());
    }
    /**
    * @param alpha from 0 to 255
    */
    public void setAlpha(int alpha)
    {
        mAlpha = alpha;
    }
    public void setForegroundColor(int foregroundColor)
    {
        mForegroundColor = foregroundColor;
    }
    public float getAlpha()
    {
        return mAlpha;
    }
    @Override
    public int getForegroundColor()
    {
        return Color.argb(mAlpha, Color.red(mForegroundColor), Color.green(mForegroundColor), Color.blue(mForegroundColor));
    }
}
```

现在，我们可以在同一个实例改变透明度和前景色了。但是，当你设置这些属性，它并不会刷新视图，你必须通过重新设置SpannableString才能刷新视图。

```java
MutableForegroundColorSpan span = new MutableForegroundColorSpan(255, Color.BLACK);
spannableString.setSpan(span, 0, text.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
textView.setText(spannableString);
//黑色完全不透明（译者注：上面代码的效果）
span.setAlpha(100);
span.setForegroundColor(Color.RED);
//到这一步文字没有变化
textView.setText(spannableString);
//最后，文字变为红色和透明
```

现在我们要前景色的动画。我们可以自定义[android.util.Property](https://developer.android.com/reference/android/util/Property.html)。

```java
private static final Property<MutableForegroundColorSpan, Integer> MUTABLE_FOREGROUND_COLOR_SPAN_FC_PROPERTY =
new Property<MutableForegroundColorSpan, Integer>(Integer.class, "MUTABLE_FOREGROUND_COLOR_SPAN_FC_PROPERTY") {
    @Override
    public void set(MutableForegroundColorSpan span, Integer value) {
        span.setForegroundColor(value);
    }
    @Override
    public Integer get(MutableForegroundColorSpan span) {
        return span.getForegroundColor();
    }
};
```

最后，我们使用属性动画（[ObjectAnimator](https://developer.android.com/reference/android/animation/ObjectAnimator.html)）让自定义属性动起来。不要忘记更新视图。

```java
MutableForegroundColorSpan span = new MutableForegroundColorSpan(255, Color.BLACK);
mSpannableString.setSpan(span, 0, text.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
ObjectAnimator objectAnimator = ObjectAnimator.ofInt(span, MUTABLE_FOREGROUND_COLOR_SPAN_FC_PROPERTY, Color.BLACK, Color.RED);
objectAnimator.setEvaluator(new ArgbEvaluator());
objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        //refresh
        mText.setText(mSpannableString);
    }
});
objectAnimator.start();
```

##### 4.1.2 ActionBar”烟火”

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333K1M-22.gif)

“烟 火”动画是让文字随机淡入。首先，把文字切断成多个spans（例如，一个character的span），淡入spans后再淡入其它的spans。用 前面介绍的MutableForegroundColorSpan，我们将创建一组特殊的span对象。在span组调用对应的setAlpha方法，我 们随机设置每个span的透明度。

```java
private static final class FireworksSpanGroup {
    private final float mAlpha;
    private final ArrayList<MutableForegroundColorSpan> mSpans;
    private FireworksSpanGroup(float alpha) {
        mAlpha = alpha;
        mSpans = new ArrayList<MutableForegroundColorSpan>();
    }
    public void addSpan(MutableForegroundColorSpan span) {
        span.setAlpha((int) (mAlpha * 255));
        mSpans.add(span);
    }
    public void init() {
        Collections.shuffle(mSpans);
    }
    public void setAlpha(float alpha) {
        int size = mSpans.size();
        float total = 1.0f * size * alpha;
        for(int index = 0 ; index < size; index++) {
            MutableForegroundColorSpan span = mSpans.get(index);
            if(total >= 1.0f) {
                span.setAlpha(255);
                total -= 1.0f;
            } else {
                span.setAlpha((int) (total * 255));
                total = 0.0f;
            }
        }
    }
    public float getAlpha() { return mAlpha; }
}
```

我们创建一个自定义属性动画的属性去更改FireworksSpanGroup的透明度

```java
private static final Property<FireworksSpanGroup, Float> FIREWORKS_GROUP_PROGRESS_PROPERTY =
new Property<FireworksSpanGroup, Float>(Float.class, "FIREWORKS_GROUP_PROGRESS_PROPERTY") {
    @Override
    public void set(FireworksSpanGroup spanGroup, Float value) {
        spanGroup.setProgress(value);
    }
    @Override
    public Float get(FireworksSpanGroup spanGroup) {
        return spanGroup.getProgress();
    }
};
```

最后，我们创建span组并使用一个ObjectAnimator给其加上动画。

```java
final FireworksSpanGroup spanGroup = new FireworksSpanGroup();
//初始化包含多个spans的grop
//spanGroup.addSpan(span);
//给ActionBar的标题设置spans
//mActionBarTitleSpannableString.setSpan(span, start, end, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
spanGroup.init();
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(spanGroup, FIREWORKS_GROUP_PROGRESS_PROPERTY, 0.0f, 1.0f);
objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener()
{
    @Override
    public void onAnimationUpdate(ValueAnimator animation)
    {
        //更新标题
        setTitle(mActionBarTitleSpannableString);
    }
});
objectAnimator.start();
```

#### 4.2 自定义Span

首先，我们要创建一个继承[ReplacementSpan](https://developer.android.com/reference/android/text/style/ReplacementSpan.html)抽象类的自定义Span（如果你想画一个自定义的背景，你可以实现[LineBackgroundSpan](https://developer.android.com/reference/android/text/style/LineBackgroundSpan.html),这是影响段落级的文本格式。）。

必须实现2个方法：

- [getSize](https://developer.android.com/reference/android/text/style/ReplacementSpan.html#getSize(android.graphics.Paint, java.lang.CharSequence, int, int, android.graphics.Paint.FontMetricsInt))：这个方法返回新的你更换后的size： 宽度；
  text：Span管理的文本
  start：文本开始处
  end：文本结尾处
  fm：字体规格，**可以为空**
  
  **目前遇到一个问题，但该TextView(加入Bitmap时)的Width不变时， 其fm会为空。**
- [draw](https://developer.android.com/reference/android/text/style/ReplacementSpan.html#draw(android.graphics.Canvas, java.lang.CharSequence, int, int, float, int, int, int, android.graphics.Paint))：可以使用Canvas绘制。
  x：绘制文本的x坐标
  top：线（line）的顶部（译者注：line的定义参看前面*字体规格*这一节）
  y：基线
  bottom：线的底部、

画一个包围文本的蓝色矩形：*FrameSpan.java*

```
@Override
public int getSize(Paint paint, CharSequence text, int start, int end, Paint.FontMetricsInt fm)
{
    //将返回相对于Paint画笔的文本
    mWidth = (int) paint.measureText(text, start, end);
    return mWidth;
}
@Override
public void draw(Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, Paint paint)
{
    //使用自定义的画笔绘制在画布上
    canvas.drawRect(x, top, x + mWidth, bottom, mPaint);
}
```

![](http://www.jcodecraeer.com/uploads/allimg/150305/00333L352-23.png)

#### 5. 其它

#### 5.1 SetSpan方法

```
void setSpan (Object what, int start, int end, int flags);
```

参数说明:

- what ： 文本格式，可以设置成前景色，背景色，下划线，中划线，模糊等
- start ： 字符串设置格式的起始下标
- end ： 字符串设置格式结束下标
- flags ： 标识

flags: 包含四种情况,用四个常量控制

- `Spanned.SPAN_INCLUSIVE_EXCLUSIVE` 从起始下标到结束下标，包括起始下标不包含结束坐标
- `Spanned.SPAN_EXCLUSIVE_EXCLUSIVE` 从起始下标到结束下标，但都不包括起始下标和结束下标
- `Spanned.SPAN_INCLUSIVE_INCLUSIVE` 从起始下标到终了下标，同时包括起始下标和结束下标
- `Spanned.SPAN_EXCLUSIVE_INCLUSIVE` 从起始下标到终了下标，包括结束下标不包含起始坐标