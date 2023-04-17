## 自定义View杂事

[TOC]

### Paint字体的使用

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

#### setAntiAlias(boolean aa)

设置画笔是否抗锯齿

#### setStrokeCap(Paint.Cap cap)

设置线冒样式，取值有

- Cap.ROUND(圆形线冒)、
- Cap.SQUARE(方形线冒)、
- Paint.Cap.BUTT(无线冒)
   注：冒多出来的那块区域就是线帽！就相当于给原来的直线加上一个帽子一样，所以叫线帽；
  因此使用Round 或 Square时，起点和终点在缩进半个线宽才能显示出来；

![](https://upload-images.jianshu.io/upload_images/2625999-a54a8a2ab7dd8567.png?imageMogr2/auto-orient/strip|imageView2/2/w/324/format/webp)



### Canvas

#### 画Drawable

```java

private Drawable runningDrawable;
runningDrawable = ResourcesCompat.getDrawable(getResources(), R.drawable.mute_mode, null);
//定义大小
runningDrawable.setBounds(0, 0, CmlUtils.convertDPToPixel("16dp"), CmlUtils.convertDPToPixel("16dp"));

 @Override
 protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
     canvas.save();
     //调整画图位置
     canvas.translate(400 , 23);
     runningDrawable.draw(canvas);
     canvas.restore();
 }
```



#### 自定义进度条

普通线性进度条，但进度条上面有一个图标跟随着进度前进；

```
/**
 * 专门用于统计步数的进度条
 */
public class StepProgressBar extends View {

    private int mMaxProgress = 100;
    private int mProgress = 1;

    private int mWidth, mHeight;
    private Paint progressBackGroundPaint;
    private Paint progressPaint;

    /**
     * 画笔的宽度
     */
    private int paintWidth;

    private Drawable runningDrawable;

    private float mProgressRate;

    public StepProgressBar(Context context) {
        super(context);
        init();
    }

    public StepProgressBar(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public StepProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        int backgroundColor = Color.parseColor("#33FC6E3F");
        int progressColor = Color.parseColor("#FC6E3F");
        CmlUtils.initialize(getResources());
        paintWidth = CmlUtils.convertDPToPixel("8dp");

        progressBackGroundPaint = new Paint();
        progressBackGroundPaint.setAntiAlias(true);
        progressBackGroundPaint.setStyle(Paint.Style.FILL);
        progressBackGroundPaint.setStrokeCap(Paint.Cap.ROUND);
        progressBackGroundPaint.setColor(backgroundColor);

        progressBackGroundPaint.setStrokeWidth(paintWidth);

        int drawableWidth = CmlUtils.convertDPToPixel("16dp");
        runningDrawable = ResourcesCompat.getDrawable(getResources(), R.drawable.mute_mode, null);
        if (runningDrawable != null) {
            runningDrawable.setBounds(0, 0, drawableWidth, drawableWidth);
            //runningDrawable.getIntrinsicWidth()
            runningDrawable.setTint(progressColor);
        }

        progressPaint = new Paint();
        progressPaint.setAntiAlias(true);
        progressPaint.setStyle(Paint.Style.FILL);
        progressPaint.setStrokeCap(Paint.Cap.ROUND);
        progressPaint.setColor(progressColor);
        progressPaint.setStrokeWidth(paintWidth);

        backGroundLineStartX = paintWidth/2;
        progressLineStartX = paintWidth/2;

        mProgressRate = (float) mProgress / mMaxProgress;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        mWidth = MeasureSpec.getSize(widthMeasureSpec);
        mHeight = MeasureSpec.getSize(heightMeasureSpec);
        Log.d("zhi","onMeasure: widthSize = "+ mWidth);
        Log.d("zhi","onMeasure: heightSize = "+ mHeight);
        mDrawableTranslateDx = mProgressRate * mWidth - CmlUtils.convertDPToPixel("16dp")/2;
        if (mDrawableTranslateDx < 0 ) {
            mDrawableTranslateDx = 0;
        }
        mDrawableTranslateDy = mHeight - CmlUtils.convertDPToPixel("16dp") - paintWidth;

        backGroundLineEndX = mWidth - paintWidth/2;
        backGroundLineStartY = backGroundLineEndY = mHeight - paintWidth/2;


        progressLineEndX = (int)((mWidth - paintWidth) * mProgressRate + paintWidth/2);
        progressLineStartY = progressLineEndY = mHeight - paintWidth/2;
    }

    private float mDrawableTranslateDx, mDrawableTranslateDy;
    private int backGroundLineStartX, backGroundLineEndX;
    private int backGroundLineStartY, backGroundLineEndY;
    private int progressLineStartX, progressLineEndX;
    private int progressLineStartY, progressLineEndY;

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Log.d("zhi","onDraw: "+ canvas);
        Log.d("zhi","backGroundLineStartX: "+ backGroundLineStartX);
        Log.d("zhi","backGroundLineStartY: "+ backGroundLineStartY);
        Log.d("zhi","backGroundLineEndX: "+ backGroundLineEndX);
        Log.d("zhi","mDrawableTranslateDx: "+ mDrawableTranslateDx);

        Log.d("zhi","progressLineStartX: "+ progressLineStartX);
        Log.d("zhi","progressLineStartY: "+ progressLineStartY);
        Log.d("zhi","progressLineEndX: "+ progressLineEndX);
        //Log.d("zhi","backGroundLineEndX: "+ backGroundLineEndX);
        Log.d("zhi","onDraw: "+ canvas);
        canvas.save();
        canvas.translate(mDrawableTranslateDx, mDrawableTranslateDy);
        runningDrawable.draw(canvas);
        canvas.restore();
        canvas.drawLine(backGroundLineStartX, backGroundLineStartY, backGroundLineEndX, backGroundLineEndY, progressBackGroundPaint);
        if (mProgressRate != 0) {
            canvas.drawLine(progressLineStartX, progressLineStartY, progressLineEndX, progressLineEndY, progressPaint);
        }
    }
}
```

