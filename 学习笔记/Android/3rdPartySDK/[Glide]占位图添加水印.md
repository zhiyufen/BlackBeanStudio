# 占位图添加水印

在Glide中对加载的图片进行绘制上我们自己内容，需要自定义Transformation， 我们这里直接继承BitmapTransformation类；

```kotlin
/**
 * 在图片中添加文字水印；
 * 目前仅支持在图片中心添加水印，如有需要，自行拓展；
 */
class WatermarkTransformation(private val markStr: String) : BitmapTransformation() {
    private var paint: Paint? = null

    constructor(markStr: String, paint: Paint) : this(markStr) {
        this.paint = paint
    }

    constructor(
        markStr: String,
        color: Int,
        textSize: Float,
        typeface: Typeface
    ) : this(markStr) {
        paint = Paint()
        paint?.apply {
            isAntiAlias = true
            setTextSize(textSize)
            textAlign = Paint.Align.CENTER
            setColor(color)
            setTypeface(typeface)
            style = Paint.Style.FILL
        }
    }

    override fun transform(
        pool: BitmapPool,
        toTransform: Bitmap,
        outWidth: Int,
        outHeight: Int
    ): Bitmap {
        val result = pool[outWidth, outHeight, Bitmap.Config.ARGB_8888]
        val canvas = Canvas(result)
        if (paint == null) {
            paint = getDefaultPaint()
        }
        paint?.let {
            val fontMetrics = it.fontMetrics
            canvas.drawBitmap(toTransform, 0f, 0f, Paint(Paint.ANTI_ALIAS_FLAG))
            canvas.drawText(
                markStr,
                outWidth / 2f,
                outHeight / 2f + (-fontMetrics.ascent - fontMetrics.descent) / 2,
                it
            )
        }
        return result
    }

    override fun updateDiskCacheKey(messageDigest: MessageDigest) {
        // do nothing
    }

    private fun getDefaultPaint(): Paint {
        val p = Paint()
        p.isAntiAlias = true
        p.textSize = ConvertUtils.dp2px(20f).toFloat()
        p.textAlign = Paint.Align.CENTER
        p.color = Color.parseColor("#FAFAFA")
        p.typeface = Typeface.create("sans-serif-light", Typeface.NORMAL)
        p.style = Paint.Style.FILL
        return p;
    }
}
```

图片转换已经实现好了，下面来看怎么适用到占位图上：

```kotlin
Glide.with(context)
.load(imgUrl)
.thumbnail(
    Glide.with(context)
          .load(R.drawable.image_mini_default_1)
    	  .apply(RequestOptions().transform(WatermarkTransformation("H"))
)
.into(imageView)
```

这样我们就可以在图片中心写上需要的文字了；