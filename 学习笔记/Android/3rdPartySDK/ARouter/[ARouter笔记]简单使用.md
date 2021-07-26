# [ARouter笔记]简单使用

## 一、 ARouter配置

### 1. 添加依赖和配置
```xml
	android {
		defaultConfig {
			...
			javaCompileOptions {
				//为了让ARouter知道这个模块来区别于其它模块的名字；
				annotationProcessorOptions {
					arguments = [AROUTER_MODULE_NAME: project.getName()]
				}
			}
		}
	}

	dependencies {
		// 替换成最新版本, 需要注意的是api
		// 要与compiler匹配使用，均使用最新版可以保证兼容
		compile 'com.alibaba:arouter-api:x.x.x'
		annotationProcessor 'com.alibaba:arouter-compiler:x.x.x'
		//implementation 'com.alibaba:arouter-api:1.5.0'
		//annotationProcessor 'com.alibaba:arouter-compiler:1.2.2'
        //implementation 'com.alibaba:fastjson:1.2.48'
		...
	}
```
### 2. 添加注解
```java
	// 在支持路由的页面上添加注解(必选)
	// 这里的路径需要注意的是至少需要有两级，/xx/xx
	@Route(path = "/test/activity")
	public class YourActivity extend Activity {
		...
	}
```

### 3. 初始化SDK

```java
	if (isDebug()) {           // 这两行必须写在init之前，否则这些配置在init过程中将无效
		ARouter.openLog();     // 打印日志
		ARouter.openDebug();   // 开启调试模式(如果在InstantRun模式下运行，必须开启调试模式！线上版本需要关闭,否则有安全风险)
	}
	ARouter.init(mApplication); // 尽可能早，推荐在Application中初始化
	发起路由操作

	// 1. 应用内简单的跳转(通过URL跳转在'进阶用法'中)
	ARouter.getInstance().build("/test/activity").navigation();

	// 2. 跳转并携带参数
	ARouter.getInstance().build("/test/1")
				.withLong("key1", 666L)
				.withString("key3", "888")
				.withObject("key4", new Test("Jack", "Rose"))
				.navigation();
```

### 4. 添加混淆规则(如果使用了Proguard)
```xml
-keep public class com.alibaba.android.arouter.routes.**{*;}
-keep public class com.alibaba.android.arouter.facade.**{*;}
-keep class * implements com.alibaba.android.arouter.facade.template.ISyringe{*;}

# 如果使用了 byType 的方式获取 Service，需添加下面规则，保护接口
-keep interface * implements com.alibaba.android.arouter.facade.template.IProvider

# 如果使用了 单类注入，即不定义接口实现 IProvider，需添加下面规则，保护实现
# -keep class * implements com.alibaba.android.arouter.facade.template.IProvider
```

### 5. 使用 Gradle 插件实现路由表的自动加载 (可选)
```jxml
apply plugin: 'com.alibaba.arouter'

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath "com.alibaba:arouter-register:?"
    }
}
```
可选使用，通过 ARouter 提供的注册插件进行路由表的自动加载(power by AutoRegister)， 默认通过扫描 dex 的方式 进行加载通过 gradle 插件进行自动注册可以缩短初始化时间解决应用加固导致无法直接访问 dex 文件，初始化失败的问题，需要注意的是，该插件必须搭配 api 1.3.0 以上版本使用！

### 6. 使用 IDE 插件导航到目标类 (可选)

在 Android Studio 插件市场中搜索 ARouter Helper, 或者直接下载文档上方 最新版本 中列出的 arouter-idea-plugin zip 安装包手动安装，安装后 插件无任何设置，可以在跳转代码的行首找到一个图标 (navigation) 点击该图标，即可跳转到标识了代码中路径的目标类



## 二、 基础使用

### 1. Activity 跳转

1. 在目标Activity中通过@Route注解来注册Activity的路径， 如：@Route(path = "/test/normalJumpActivity")
2. 通过ARouter对象进行设置跳转Activity路径等参数，调用navigation()进行跳转；

```java
ARouter.getInstance().build("/test/normalJumpActivity").navigation();
```

例子：
```java
public class ARouterActivity extends Activity {
    public final static int  REQUEST_CODE = 100;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.arouter_testing_layout_activity);
        ButterKnife.bind(this);
    }

    /**
     * testing ARouter start activity with parameter and testing start activity for result.
     */
    @SuppressWarnings("unused")
    @OnClick(R.id.arouter_jump_button)
    public void runningARouterTesting() {
		//应用内简单的跳转
		//ARouter.getInstance().build("/activity/normalJumpActivity").navigation();
		
		// 跳转并携带参数
        ARouter.getInstance() // 获取ARouter对象
                .build("/activity/normalJumpActivity") // 输入目标Activity的路径
                .withString("key1","HelloWord!") //传输参数
                .navigation(this, REQUEST_CODE); //进行跳转
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode){
            case REQUEST_CODE:
                if(resultCode == RESULT_OK){
                    String resultKey2 = data.getStringExtra("key2");
                    Log.d("yufen","返回结果为：" +  resultKey2);
                    Toast.makeText(
                            this, "返回结果为： " + resultKey2, Toast.LENGTH_LONG).show();
                }
        }
    }
}
```

目标Activity：
```java
@Route(path = "/activity/normalJumpActivity")
public class ARouterNormalJumpActivity extends Activity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.temp_main_layout);
        String resultKey1 = getIntent().getStringExtra("key1");
        Log.d("yufen","传过来的数据为：" +  resultKey1);
        Toast.makeText(
                this, "传过来的数据为： " + resultKey1, Toast.LENGTH_SHORT).show();

        //3秒后返回Activity结果及结束当前Activity；
        Handler handler = new Handler(getMainLooper());
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                Intent intent = new Intent();
                intent.putExtra("key2", "I like you!");
                setResult(RESULT_OK, intent);
                finish();
            }
        }, 3000);
    }
}
```

在测试app上， 点击arouter_jump_button按钮后， 输出的log如下：
```
2019-06-10 13:51:10.179 20751-20751/blackbean.rxjavademo D/yufen: 传过来的数据为：HelloWord!
2019-06-10 13:51:13.254 20751-20751/blackbean.rxjavademo D/yufen: 返回结果为：I like you!
```

### 2. URL 跳转
&emsp;&emsp;现在app为了让浏览器能跳转到app，一般都为实现类似Deeplink的功能，但ARouter同样提供这类的Activity跳转；

a. 实现统筹URL跳转的Activity(无界面)，通过该Activity来使用ARouter，从而实现ARouter的目标界面的跳转；

统筹URL跳转的Activity:
```java
// 新建一个Activity用于监听Scheme事件,之后直接把url传递给ARouter即可
public class SchemeFilterActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        Uri uri = getIntent().getData();
        ARouter.getInstance().build(uri).navigation();
        finish();
    }
}

```
AndroidManifest: 
```xml
        <activity android:name=".ARouter.SchemeFilterActivity">
            <intent-filter>
                <data
                    android:host="yufen.zhi.com"
                    android:scheme="arouter"
                    />
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
            </intent-filter>
        </activity>
```
通过上面，我们就可以实现URL跳转了。

b. 解析URL参数

&emsp;&emsp;为用于保存URL或者Intent传送过来的数据声明一个字段(命名需要和key相同)，并使用 @Autowired 标注，这样就可以ARouter.getInstance().inject(this)方法中自动帮你转换；另外你也可以在@Autowired中输入name作为key的命名，然后变量名，你就可以随意定；

&emsp;&emsp;如果需要传递自定义对象，那么模块中需要有一个类实现SerializationService,并使用@Route注解标注(方便用户自行选择序列化方式)；

```java
//JsonServiceImpl.java：
@Route(path = "/service/json")
	public class JsonServiceImpl implements SerializationService {
		@Override
		public void init(Context context) {
		}
		//使用该JSON，需要导入compile 'com.alibaba:fastjson:1.2.48'
		@Override
		public <T> T parseObject(String s, Type type) {
			return JSON.parseObject(s, type);
		}

		@Override
		public String object2Json(Object instance) {
			return JSON.toJSONString(instance);
		}

		//新版本已使用parseObject代替
		@Override
		public <T> T json2Object(String s, Class<T> aClass) {
			return null;
		}
	}
```
&emsp;&emsp; 我们来实现一个能解析URL参数的Activity：

```java
@Route(path = "/activity/UrlActivity")
public class ARouterUrlActivity extends Activity {
    @Autowired
    public String mName;
    @Autowired
    int age;
    @Autowired(name = "girl") // 通过name来映射URL中的不同参数
    boolean isBoy;

    /**
     * 如果需要传递自定义对象，新建一个类（并非自定义对象类）,
     * 然后实现 SerializationService,并使用@Route注解标注(方便用户自行选择序列化方式)
     */
    @Autowired
    TestObj obj;    // 支持解析自定义对象，URL中使用json传递

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ARouter.getInstance().inject(this);

        Log.d("yufen", ARouterUrlActivity.class.getSimpleName() + ": onCreate....");
        Log.d("yufen", "name=" + mName);
        Log.d("yufen", "age=" + age);
        Log.d("yufen", "isBoy=" + isBoy);
        if (obj != null)
            Log.d("yufen", "obj=" + obj.toString());

    }

    private static class TestObj {
        int id;
        String level;
        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }

        public String getLevel() {
            return level;
        }

        public void setLevel(String level) {
            this.level = level;
        }

        @NonNull
        @Override
        public String toString() {
            return "{id:\"" + id + "\", level:\"" + level + "\"}";
        }
    }
```
&emsp;&emsp; 编写一个简单静态网页会测试一下：

```xml
<html>
<body>
<br><br>
<div style="font-family:arial;color:red;font-size:60px;">
 <a href='arouter://yufen.zhi.com/activity/UrlActivity?mName=yufen&age=18&gril=false&obj=%7B%22level%22:%22top%22,%22id%22:123%7D'>ARouter Url</a>
</div>
</body>
</html>
 
```
&emsp;&emsp;其中arouter://yufen.zhi.com/test/activity2?mName=yufen&age=18&gril=false就是我们的URL，另外如果传递自定义对象，则需要把Json进行URL编码。如： {"level":"top","id":123}， 进行编码后URL应该是 arouter://yufen.zhi.com/activity/UrlActivity?mName=yufen&age=18&gril=false&obj=%7B%22level%22:%22top%22,%22id%22:123%7D

&emsp;&emsp;我们这里简略看一下为什么使用@Autowired注解 + ARouter.getInstance().inject(this) 组合就可以自动赋值的; 

&emsp;&emsp;首先在一个类中使用@Autowired注解变量时， Arouter的注解编译器会编译期间为生成一个包含inject()的类， 比如上面ARouterUrlActivity的注解：

```java
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY AROUTER. */
public class ARouterUrlActivity$$ARouter$$Autowired implements ISyringe {
  private SerializationService serializationService;

  @Override
  public void inject(Object target) {
    serializationService = ARouter.getInstance().navigation(SerializationService.class);
    ARouterUrlActivity substitute = (ARouterUrlActivity)target;
    substitute.mName = substitute.getIntent().getStringExtra("mName");
    substitute.age = substitute.getIntent().getIntExtra("age", substitute.age);
    substitute.isBoy = substitute.getIntent().getBooleanExtra("girl", substitute.isBoy);
    if (null != serializationService) {
      substitute.obj = serializationService.parseObject(substitute.getIntent().getStringExtra("obj"), new com.alibaba.android.arouter.facade.model.TypeWrapper<ARouterUrlActivity.TestObj>(){}.getType());
    } else {
      Log.e("ARouter::", "You want automatic inject the field 'obj' in class 'ARouterUrlActivity' , then you should implement 'SerializationService' to support object auto inject!");
    }
  }
}
```
&emsp;&emsp; 然后通过ARouter.getInstance().inject(this)方法会找到上面自动生成的类并调用其inject(), 从而从Intent中读取数据；

## 三、进阶使用

###  1. 拦截器

&emsp;&emsp;拦截器其实就是AOP的实现(面向切面编程)， 当某个模块编译入app时， 该模块的拦截器就会注册并起作用；所以在跳转某Activity时，会根据优先级先后经过所有注册的拦截器，如果某拦截器把该跳转事件拦截了，将无法跳转目标Activity;

&emsp;&emsp;自定义拦截器实现步骤：
1. 实现IInterceptor接口；
2. 在类上添加@Interceptor的注解，并传入priority的值（拦截器的优先级），该值越小，优先级越高；
3. 然后实现其接口方法：

```java
	init()//初始化方法，注意只调用一次；
	pocess()//判断事件是否需要拦截及其它操作(可调用InterceptorCallback.onInterrupt(exception)来拦截或InterceptorCallback.onContinue(postcard) 来放行)；
```

实现拦截器Demo:

```java
@Interceptor(priority = 5, name = "拦截器2号")
public class MyInterceptor2 implements IInterceptor {
    @Override
    public void process(Postcard postcard, InterceptorCallback callback) {
        Log.d("yufen","process: " + MyInterceptor2.class.getName());
        if("/activity/normalJumpActivity".equals(postcard.getPath())) {//判断其跳转的目标Activity是NormalJumpActivity就拦截它；
            callback.onInterrupt(new RuntimeException("被拦截跳转到Activity1了"));
        } else {
            callback.onContinue(postcard);
        }
    }

    @Override
    public void init(Context context) {
        Log.d("yufen","init: " + MyInterceptor2.class.getName());
    }
}
```

我们再实现一个跳转NormalJumpActivity的动作并实现NavigationCallback接口方法来监控一下跳转的情况：

```java
    @OnClick(R2.id.interceptor_button)
    public void startTestingInterceptor() {
        ARouter.getInstance().build("/activity/normalJumpActivity")
                .withString("key1","HelloWord!")
                .navigation(this, new NavigationCallback() {
                    @Override
                    public void onFound(Postcard postcard) {//表示目标Activity是存在的
                        Log.d(TAG,"找到了");
                    }

                    @Override
                    public void onLost(Postcard postcard) {//表示目标Activity不存在的
                        Log.d(TAG,"找不到");
                    }

                    @Override
                    public void onArrival(Postcard postcard) {//已跳转到目标Activity
                        Log.d(TAG,"跳转完了");
                    }

                    @Override
                    public void onInterrupt(Postcard postcard) {//中途被拦截了
                        Log.d(TAG,"被拦截了");
                    }
                });
    }
```
我们来看看其跳转动作的log：

```
2019-06-21 16:52:51.500 22435-22435/blackbean.rxjavademo D/ViewRootImpl@759c87e[ARouterActivity]: ViewPostIme pointer 0
2019-06-21 16:52:51.607 22435-22435/blackbean.rxjavademo D/ViewRootImpl@759c87e[ARouterActivity]: ViewPostIme pointer 1
2019-06-21 16:52:51.615 22435-22435/blackbean.rxjavademo D/yufen/ARouterActivity: 找到了
2019-06-21 16:52:51.618 22435-22456/blackbean.rxjavademo D/yufen: process: com.yufen.arouterlib2.interceptor.MyInterceptor2
2019-06-21 16:52:51.619 22435-22456/blackbean.rxjavademo D/yufen/ARouterActivity: 被拦截了
2019-06-21 16:52:51.621 22435-22456/blackbean.rxjavademo I/ARouter::: Navigation failed, termination by interceptor : 被拦截跳转到Activity1了[ ] 
```

&emsp;&emsp;而路由跳转的执行顺序为，先执行回调函数的onFound()，之后是拦截器2的process()，拦截器1的process()，最后执行回调函数的onArrival()。注意：拦截器是按照优先级的高低进行顺序执行的，优先级也高，越早执行。
&emsp;&emsp;上面就是关于拦截器的使用，你可以在process()中通过postcard的属性值进行判断，然后进行拦截处理，处理成功调用callback.onContinue()方法继续往下执行，失败则调用callback.onInterrupt()方法中断跳转。其中，postcard包含了路由节点的各种信息。

&emsp;&emsp;对于ARouter来说， 有一个“All In One”的概念，就是希望所有页面中的配置都能够浓缩到这一个页面中，也就是高内聚低耦合的思想，不希望页面的配置逃出页面，配置到像Manifest的其他地方。
&emsp;&emsp;因此在我们配置目标页面时，还有一个属性extras可以设置其值， 该值类型是int， 在Java中由4个字节实现，每个字节是8位，所以一共是32个标志位，去除掉符号位还剩下31个，也就是说转化成为二进制之后，一个int中可以配置31个1或者0，而每一个0或者1都可以表示一项配置即可；剩下的可以自行发挥，通过字节操作可以标识32个开关，通过开关标记目标页面的一些属性，在拦截器中可以拿到这个标记进行业务逻辑判断；

```java
//配置项定义
public final class InterceptConfigKey {
    public final static int INTERCEPT_VALUE_LOGIN = 1;
    public final static int INTERCEPT_VALUE_PAY_SDK = 1 << 1;
}

//拦截器针对该配置
@Interceptor(priority = 6, name = "拦截器1号")
public class MyInterceptor1 implements IInterceptor {
    @Override
    public void process(Postcard postcard, InterceptorCallback callback) {
        Log.d("yufen","process: " + MyInterceptor1.class.getName());
        if((postcard.getExtra() & (InterceptConfigKey.INTERCEPT_VALUE_LOGIN)) > 0) {
            if (true) { //假设判断本地手机没安装支付宝，需要启动H5 Activity来绑定支付宝账号
                ARouter.getInstance().build("/activity/normalJumpActivity")
                        .navigation();
            }
            callback.onInterrupt(new RuntimeException("被拦截跳转到NormalJumpActivity了"));
        }
        callback.onContinue(postcard);
    }

    @Override
    public void init(Context context) {
        Log.d("yufen","init: " + MyInterceptor1.class.getName());
    }
}
```

Log:
```
2019-06-21 19:38:04.059 5734-5734/blackbean.rxjavademo D/yufen/ARouterActivity: 找到了
2019-06-21 19:38:04.060 5734-5791/blackbean.rxjavademo D/yufen: process: com.yufen.arouterlib2.interceptor.MyInterceptor2
2019-06-21 19:38:04.060 5734-5791/blackbean.rxjavademo D/yufen: process: com.yufen.arouterlib2.interceptor.MyInterceptor1
2019-06-21 19:38:04.061 5734-5791/blackbean.rxjavademo D/yufen/ARouterActivity: 被拦截了     //被拦截后跳转NormalJumpActivity
2019-06-21 19:38:04.061 5734-5792/blackbean.rxjavademo D/yufen: process: com.yufen.arouterlib2.interceptor.MyInterceptor2
2019-06-21 19:38:04.062 5734-5792/blackbean.rxjavademo D/yufen: process: com.yufen.arouterlib2.interceptor.MyInterceptor1
2019-06-21 19:38:04.176 5734-5734/blackbean.rxjavademo D/yufen: 传过来的数据为：null
```
&emsp;&emsp; 对于这个拦截器，个人觉得可分为全局性和局部性的；比如extra的31个位中，可分16位为全局性的， 不管目标页面是什么，需要配置该位的，就进行拦截判断；另外剩下15个，可灵活目标页面来判断拦截；不然31位还是不太够用的。
如：
```java
 @Override
    public void process(Postcard postcard, InterceptorCallback callback) {
        if("/activity/normalJumpActivity1".equals(postcard.getPath()) 
                && (postcard.getExtra() & (InterceptConfigKey.INTERCEPT_VALUE_LOGIN)) > 0) {
            ....
            callback.onInterrupt(new RuntimeException("局部配置拦截"));
        }
        callback.onContinue(postcard);
    }
```

#### 注意事项
	· 定义多个拦截器的时候，priority的值不能定义一样的，只要其中两个拦截器的优先值一样，编译时会报错。
	· 在拦截器的process()方法中，如果对传入的 postcard对象设置了tag值，那么跳转会被当做拦截处理，通常来说，postcard的tag值会用来保存拦截处理过程中产生的异常对象: Postcard.setTag(new Object())。
	· 在拦截器的process()方法中，如果你即没有调用callback.onContinue(postcard)方法也没有调用callback.onInterrupt(exception)方法，那么不再执行后续的拦截器，需等待300s（默认值，可设置改变）的时间，才能抛出拦截器中断。
	· 拦截器的process()方法以及带跳转的回调中的onInterrupt(Postcard postcard)方法，均是在分线程中执行的，如果需要做一些页面的操作显示，必须在主线程中执行。

### 2. 降级策略

&emsp;&emsp; 当我们跳转某个目标页面时, 有可能跳转不了，这时候我们可能需要针对这种情况进行一定降级处理： 一般可分为 "单独降级"、“全局降级”；

#### 2.1 单独降级

&emsp;&emsp;只针对本次跳转失败时的降级处理，可实现 new NavigationCallback()接口，跳转失败时，会回调该onLost()方法；
```java
ARouter.getInstance().build("/test/activity12345")
                .withString("key1","HelloWord!")
                .navigation(this, new NavigationCallback() {
                    ...
                    @Override
                    public void onLost(Postcard postcard) {
                        Log.d("yufen","找不到");
                        //Do something
                    }
 
                    ...
                });
```

#### 2.2 全局降级

&emsp;&emsp;针对所有跳转失败时（没使用上面单独降级的跳转）的降级处理，可实现DegradeService接口，并配置任意的path进行注解即可：
```java
@Route(path = "/xxx/xxx")
public class DegradeServiceImpl implements DegradeService{
 
    @Override
    public void onLost(Context context, Postcard postcard) {
        Log.d("yufen", "DegradeServiceImpl onLost.....");
    }
 
    @Override
    public void init(Context context) {
        Log.d("yufen", "DegradeServiceImpl init.....");
    }
}

```
&emsp;&emsp;如果使用单独降级调用的话， 是不会再次降级到全局的；另外DegradeServiceImpl只有在第一次调用时， 才会初始化一次， 后面不再初始化；而拦截器是一开始就初始化的
&emsp;&emsp;经测试， 这个全局降级的在所有组件中只能定义一个，因此我们平时在一般组件(Base组件可以用这个)开发，不要使用这个全局降级，不然随时有不起作用；
&emsp;&emsp;常见用法： 降级后，通过跳转到第三方的H5的错误页面来解决的，因为APP不能够重复发布，但是H5是可以重复发布的，所以可以通过H5的方式解决降级问题，把去向的目标页面作为目标的参数传递到H5中


### 3. 服务管理

&emsp;&emsp;所谓的服务管理就是把不存在任何依赖的模块之间如何调用对方组件暴露的接口服务，从而实现组件之前不需要依赖和进行热插拔；实现原理是： IOC设计模式(控制反转)

&emsp;&emsp;实现步骤：

##### a. 定义需要暴露服务的接口
```java
package com.yufen.baselib.arouter.server_interface;

import com.alibaba.android.arouter.facade.template.IProvider;

/**
 * Created by Yufen Zhi on 2018/8/22.
 */
public interface ISayService extends IProvider {
    String say();
}
```
&emsp;&emsp; 该接口的定义地方一般会放在BasedLib或者CommonLib中， 因为这个接口的具体实现或者调用都需要导入该接口；

##### b. 实现暴露服务的接口

&emsp;&emsp;把上面接口进行具体的实现，以便其它组件进行调用；
```java
@Route(path = "/service/dog")
public class DogSayServiceImpl implements ISayService {

    @Override
    public String say() {
        return "汪汪汪汪汪汪";
    }

    @Override
    public void init(Context context) {
        Log.d("yufen","DogSayServiceImpl init.....");
    }
}

@Route(path = "/service/human")
public class HumanSayServiceImpl implements ISayService {

    @Override
    public String say() {
        return "我是Android人";
    }

    @Override
    public void init(Context context) {
        Log.d("yufen","HumanSayServiceImpl init.......");
    }
}

```
##### c. 调用服务

```java
    //当某接口只有一个实现时，可以精确找到实现体，但多个时，不确定；
    @Autowired
    ISayService humanSay1Service;
    //推荐)使用依赖注入的方式发现服务,通过注解标注字段,即可使用，无需主动获取
    @Autowired(name = "/service/dog")
    ISayService dogSayService;

    ISayService humanSayService2;
    ISayService humanSayService3;

    @OnClick(R2.id.server_test)
    public void testingServer() {
        Log.d("yufen","humanSay1Service = " + humanSay1Service.say());
        Log.d("yufen","dogSayService = " + dogSayService.say());

        // 2. 使用依赖查找的方式发现服务，主动去发现服务并使用，下面两种方式分别是byName和byType

        //byType， 同组件内可以这么用； 
        //humanSayService2 = ARouter.getInstance().navigation(HumanSayServiceImpl.class);

        //byName
        humanSayService3 = (ISayService)ARouter.getInstance().build("/service/human").navigation();
        if (humanSayService2 != null) {
            Log.d("yufen","humanSayService2 = " + humanSayService2.say());
        }
        if (humanSayService3 != null ) {
            Log.d("yufen","humanSayService3 = " + humanSayService3.say());
        }
    }
```
&emsp;&emsp; 综合上面多种调用方式来说， 推荐使用byName的方式来获取实际的实现类（dogSayService, humanSayService3）; 

#### 预处理服务

```java
/**
 * 实现 PretreatmentService 接口，并加上一个Path内容任意的注解即可
 */
@Route(path = "/xxx/xxx")
public class PretreatmentServiceImpl implements PretreatmentService {
    private final static String TAG = "PretreatmentServiceImpl";
    @Override
    public boolean onPretreatment(Context context, Postcard postcard) {
        // 跳转前预处理，如果需要自行处理跳转，该方法返回 false 即可
        Log.d(TAG, "onPretreatment");
        return false;
    }

    @Override
    public void init(Context context) {
        
    }
}
```
&emsp;&emsp;onPretreatment在刚启动时会调用一次， 每次跳转都会调用一次， 目前还没发现有什么实际上运用；

### 4. 其它

```java
// 跳转Activity时可指定Flag
ARouter.getInstance()
    .build("/home/main")
    .withFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
    .navigation();
				
// 获取Fragment
Fragment fragment = (Fragment) ARouter.getInstance().build("/test/fragment").navigation();
		
// 转场动画(常规方式)
ARouter.getInstance()
    .build("/test/activity2")
    .withTransition(R.anim.slide_in_bottom, R.anim.slide_out_bottom)
    .navigation(this);

// 转场动画(API16+)
ActivityOptionsCompat compat = ActivityOptionsCompat.
    makeScaleUpAnimation(v, v.getWidth() / 2, v.getHeight() / 2, 0, 0);
	
// 使用绿色通道(跳过所有的拦截器)
ARouter.getInstance().build("/home/main").greenChannel().navigation();

// 使用自己的日志工具打印日志
ARouter.setLogger();

// 使用自己提供的线程池
ARouter.setExecutor();

```


相关学习网址： 

	1， 开源最佳实践：Android平台页面路由框架ARouter： https://yq.aliyun.com/articles/71687?t=t1 
	3， GitHub的中文文档： https://github.com/alibaba/ARouter/blob/master/README_CN.md
	2， https://blog.csdn.net/weijianfeng1990912/article/details/66475978
	4， http://www.qingpingshan.com/rjbc/az/287674.html
	5,  深度好文：https://www.jianshu.com/p/c8d7b1379c1b
