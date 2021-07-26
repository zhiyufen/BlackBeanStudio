### Picasso 使用笔记



picasso是Square公司开源的一个Android图形缓存库。可以实现图片下载和缓存功能。

特点：

1.在adapter中回收和取消当前的下载。 
2.使用最少的内存完成复杂的图形转换操作。
3.自动的内存和硬盘缓存。
4.图形转换操作，如变换大小，旋转等，提供了接口来让用户可以自定义转换操作。
5.加载载网络或本地资源。

#### 1. Gradle 配置

```groovy
implementation 'com.squareup.picasso:picasso:2.71828'
```

#### 2. 基本使用

```java
 View imageview=findViewById(R.id.activity_picass_imageview);
 String url="XXX";
 
 Picasso.get()//获取Picasso对象
        .load(url) //下载url对应的图片,url可以是Uri，File,ResId,path中一个 
        .into(imageview);//显示ImageView
```

获取Picasso对象时，我们还需要配置一些选项：

```java
    private static final String CACHE_DIR_NAME = "providerImages";
    private static volatile ImageLoader instance;
    private Picasso picasso;
	//内存缓存容量
    private static final int MIN_NORMAL_MEMORY_CACHE_SIZE = 4 * 1024 * 1024;
    //本地缓存容量
    private static final int MIN_NORMAL_DISK_CACHE_SIZE = 8 * 1024 * 1024;

    private ImageLoader(Context context){
        Picasso.Builder builder = new Picasso.Builder(context);
        //Debug版本相关
        if (BuildConfig.DEBUG && Build.TYPE.equalsIgnoreCase("eng")) {
            builder.loggingEnabled(true)
                    .indicatorsEnabled(true);
        }
        builder.memoryCache(new LruCache(MIN_NORMAL_MEMORY_CACHE_SIZE));
        File cacheDir = context.getDir(CACHE_DIR_NAME, Context.MODE_PRIVATE);
        // 使用OKHttp作为下载者来实现本地缓存
        builder.downloader(new OkHttp3Downloader(cacheDir, MIN_NORMAL_DISK_CACHE_SIZE)).build();
        picasso = builder.build();
        //设置Picasso全局单一实例
        Picasso.setSingletonInstance(picasso);
        if (Build.TYPE.equalsIgnoreCase("eng")) {
            picasso.setLoggingEnabled(true);
        }
    }
```



**加载sd卡本地图片：**

```java
imageview=findViewById(R.id.activity_picass_imageview);
File file = new File(Environment.getExternalStorageDirectory() + "/icon.png");
Picasso.get()
        .load(file)

```

**加载应用资源：**

```java
imageview=findViewById(R.id.activity_picass_imageview);
int resource = R.mipmap.za;
Picasso.get()
        .load(resource)

```

```java
imageview=findViewById(R.id.activity_picass_imageview);
String prefixurl="file:///android_asset/";
Picasso.get()
        .load(prefixurl+"tab_my_pressed.png")
        .into(imageview);
```

**加载Uri对象**

```java
imageview=findViewById(R.id.activity_picass_imageview);
Uri imageUri =XXX;
Glide.with(this)
      .load(imageUri)
      .into(imageview);
```

**添加占位符**

```java
imageview=findViewById(R.id.activity_picass_imageview);
String url="XXX";
Picasso.get()
       .load(url)
       //图片没加载时显示的图片
       .placeholder(R.drawable.ic_launcher_background)
       //图片加载失败时显示的图片
       .error(R.drawable.ic_launcher_background)
       .into(imageview);
```

**裁剪图片尺寸**

```
imageview=findViewById(R.id.activity_picass_imageview);
String url="XXX";
Picasso.get()
       .load(url)
       .resize(20,20)//将图片剪切成20*20大小的图片
       .into(imageview);
```

resize方法经常和centerCrop()，centerInside()等方法一起使用。但是后两者不可单独使用。

使用centerCrop()这个函数实现了拉伸截取中间部分，会铺满全屏，但是也许会将图片四周截去一部分；

如果想看清楚全貌，使用centerinside()方法，但是这样做有可能出现图片无法充满整个view的情况；

另外还可以使用.fit()方法来计算出最佳的大小及最佳的图片质量来进行图片展示 (  减少内存 )：

```java
imageview=findViewById(R.id.activity_picass_imageview);
String url="http://i.imgur.com/DvpvklR.png";
Picasso.get()
       .load(url)
       .fit()
       .into(imageview);
```



#### 3. 进阶使用

**加载自定义形状图片**

需要自定义**Transformation**实现类， 我们以圆角为例子：

```java
public class RoundTransform implements Transformation {
 
    private Context mContext;
 
    public RoundTransform(Context context) {
        mContext = context;
    }
 
    @Override
    public Bitmap transform(Bitmap source) {
        int widthLight = source.getWidth();
        int heightLight = source.getHeight();
        int radius = ActivityUtils.dip2px(mContext, 10); // 圆角半径
        Bitmap output = Bitmap.createBitmap(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(output);
        Paint paintColor = new Paint();
        paintColor.setFlags(Paint.ANTI_ALIAS_FLAG);
        RectF rectF = new RectF(new Rect(0, 0, widthLight, heightLight));
        canvas.drawRoundRect(rectF, radius, radius, paintColor);
        Paint paintImage = new Paint();
        paintImage.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP));
        canvas.drawBitmap(source, 0, 0, paintImage);
        source.recycle();
        return output;
    }
 
    @Override
    public String key() {
        return "roundcorner";
    }
 
}
```

效果应用：

```java
imageview=findViewById(R.id.activity_picass_imageview);
 String url="XXX";
 Picasso.get()
        .load(url)
        .transform(new RoundTransform(PicassoActivity.this))
        .into(imageview);
```

注意：设置圆角图片时，如果ImageView设置如下类型android:scaleType="centerCrop"则圆角没有效果???

**加载图片优先级**

因为Picasso是异步加载,所以多个图片时那个图片先加载出来是不一定的。如果用户想自定义图片加载的先后顺序。Picasso支持设置优先级,分为HIGH, NORMAL, 和 LOW,所有的加载默认优先级为NORMAL。

```java
imageview=findViewById(R.id.activity_picass_imageview);
String url="http://i.imgur.com/DvpvklR.png";
Picasso.get()
       .load(url)
       .priority(Picasso.Priority.HIGH)
       .into(imageview);
```

**旋转图片**

```java
 imageview=findViewById(R.id.activity_picass_imageview);
 String url="http://i.imgur.com/DvpvklR.png";
 Picasso.get()
        .load(url)
        .rotate(90)
        .into(imageview);
```

.rotate(90)：顺时针旋转90°。(默认圆心(0,0))。

.rotate(float degrees, float pivotX, float pivotY)：自定义圆心。

**Tag设置**

```
imageview=findViewById(R.id.activity_picass_imageview);
String url="http://i.imgur.com/DvpvklR.png";
Picasso.get()
       .load(url)
       .tag(1)
       .into(imageview);
```

标志该次请求的，该方法可拓展功能时使用；比如需要中途取消或暂停或恢复加载等：cancelTag(tag), pauseTag(tag),resumeTag(tag); 

**内存问题**

```

```



#### 4. 遇到问题点：

**需要加网络访问权限：**

```groovy
<uses-permission android:name="android.permission.INTERNET"/>
```

在Androd9 及更高版本上遇到 http访问返回错误码605的问题，那是因为Android的网络安全配置导致的，可使用下面方法：

1. 在res/xml目录下新建network-security-config.xml:

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
       <base-config cleartextTrafficPermitted="true" />
   </network-security-config>
   ```

   

2. 在Manifest.xml文件中使用配置：

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <manifest ... >
       <application android:networkSecurityConfig="@xml/network_security_config">
       </application>
   </manifest>
   ```

   但上面是允许所有http访问，也许会有问题, 可配置的, 比如：

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
       <base-config cleartextTrafficPermitted="true">
           <trust-anchors>
               <certificates src="system" />
           </trust-anchors>
       </base-config>
   </network-security-config>
   ```

   



#### 其它

**Picasso框架设计图**

![](https://img-blog.csdn.net/20180713110808486?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzczMDQ4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如上图，整个库分为 Dispatcher，RequestHandler 及 Downloader，PicassoDrawable 等模块。

Dispatcher 负责分发和处理 Action，包括提交、暂停、继续、取消、网络状态变化、重试等等。

简单的讲就是 Picasso 收到加载及显示图片的任务，创建 Request 并将它交给 Dispatcher，Dispatcher 分发任务到具体 RequestHandler，任务通过 MemoryCache 及 Handler(数据获取接口) 获取图片，图片获取成功后通过 PicassoDrawable 显示到 Target 中。

需要注意的是上面 Data 的 File system 部分，Picasso 没有自定义本地缓存的接口，默认使用 http 的本地缓存，API 9 以上使用 okhttp，以下使用 Urlconnection，所以如果需要自定义本地缓存就需要重定义 Downloader。

**源码：GitHub链接**

https://github.com/square/picasso

**几种图片框架对比图**

![](https://img-blog.csdn.net/20180716152341997?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzczMDQ4Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

本文笔记来自 https://blog.csdn.net/weixin_37730482/article/details/80901587