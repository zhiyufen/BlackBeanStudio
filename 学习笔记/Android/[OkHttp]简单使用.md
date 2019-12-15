# [OkHttp]简单使用

***
## 概念
&emsp;&emsp;Okhttp对Http相关操作进行封装的一个Jar包（支持HTTP 和 HTTP/2），其API设计轻巧、可支持同步请求，也可支持异步请求；设计缓存处理，请求的数据可缓存到本地，再次请求时，如果数据还在有效期内，可直接使用，避免过多重复请求服务器；如果通过HTTP/2协议，则可以客户端到同一服务器的所有请求都同用一个Socket连接。但使用HTTP/1.x协议，其okhttp也在内部使用连接池，对连接进行重用，减少延迟。

&emsp;&emsp;Android Studio的Gradle中进行配置okhttp：
```
dependencies {
    compile 'com.squareup.okhttp3:okhttp:3.10.0'
}
```
&emsp;&emsp;okhttp的源码请参考GitHub： https://github.com/square/okhttp

***
## 常用类

### okhttp3.OkHttpClient;
&emsp;&emsp;该类表示Http请求的客户端类。一般来说，为了所有的请求都可以共用Response缓存，线程池，以及连接池，我们会自定义HttpUtil类，该类对OkHttpClient只进行一次实例化（new OkHttpClient()）,然后所有Http请求都通过HttpUtil来共同使用这个OkHttpClient对象。
```
OkHttpClient client = new OkHttpClient()
```
&emsp;&emsp;需要通过OkHttpClient.Builder来设置OkHttpClient的参数：如连接超时时间，读取超时时间，缓存目录，代理等。
```
//HttpUtil.java（共用）
OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(3000, TimeUnit.MILLISECONDS)
                .readTimeout(3000, TimeUnit.MILLISECONDS)
                .cache(cache)
                .proxy(proxy)
                .authenticator()
                .build();
				
//当你需要对某个特殊网络请求设置额外的参数，可通过newBuilder()进行设置新的（重复的话，会覆盖原来的）
OkHttpClient newClient = client.newBuilder()
                .connectTimeout(600, TimeUnit.MICROSECONDS)
                .build();

```
### okhttp3.Request; 
&emsp;&emsp;Request类封装了请求报文信息：请求的Url地址、请求的方法（如GET、POST等）、各种请求头（如Content-Type、Cookie）以及可选的请求体。一般通过内部类Request.Builder的链式调用生成Request对象。
```
Request request = new Request.Builder()
           .url("http://publicobject.com/helloworld.txt")
            .get()//设置为“GET”
            .header("Accept", "application/json; q=0.5")
            .build();
```
Request的相关API：
```
//请求方法参数设置
get()//“GET”
delete()//“DELETE”
post(requestBody)//“POST”
put(requestBody)//”PUT“
patch()//”PATCH“

//设置Url
url(HttpUrl url/String url/URL url)

//设置请求头的name和value,但不能重复
header(String name, String value)
//但想添加多个name相同的请求头
addHeader(String name, String value)

```

### okhttp3.Call; 
&emsp;&emsp;Call代表了一个实际的HTTP请求，通过OkHttpClient对象的newCall(Request)方法获得Call对象。再通过该Call对象进行同步或异步获取数据。
```
//同步
try {
    new OkHttpClient.Builder().build().newCall(new Request.Builder().build()).execute();
} catch (IOException e) {
    e.printStackTrace();
}
//异步
new OkHttpClient.Builder().build().newCall(new Request.Builder().build()).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {

    }
});
```
### okhttp3.Response 
&emsp;&emsp;Response类封装了响应报文信息：状态吗（200、404等）、响应头（Content-Type、Server等）以及可选的响应体。可以通过Call对象的execute()方法获得Response对象，异步回调执行Callback对象的onResponse方法时也可以获取Response对象。
```
try {
    Response response = client.newCall(request).execute();
    Headers headers = response.headers();
    for (int i = 0; i < headers.size(); i++) {
        Log.d(TAG,headers.name(i) + ": " + headers.value(i));
    }
    Log.d(TAG, response.body().string());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
&emsp;&emsp;常用API
```
//判断请求是否成功
isSuccessful();

//得到响应头Headers对象
headers();
//获取响应头的名称
Headers.name(index);
//获取响应头的值。
Headers.value(index);
//获取某个响应头的值
Headers.get(headerName);

//得到响应体ResponseBody对象,调用其string()方法可以很方便地将响应体中的数据转换为字符串，该方法会将所有的数据放入到内存之中，所以如果数据超过1M，最好不要调用string()方法以避免占用过多内存，这种情况下可以考虑通过source()、byteStream()或charStream()进行流式处理。对于Json的数据结果，可采用Gson来进行Json与Java对象的转换：gson.fromJson(response.body().charStream(), XXX.class);
body();
```
### ResponseBody
&emsp;&emsp;通过Response的body()方法可以得到响应体ResponseBody，响应体必须最终要被关闭，否则会导致资源泄露、App运行变慢甚至崩溃。
&emsp;&emsp;ResponseBody和Response都实现了Closeable和AutoCloseable接口，它们都有close()方法，Response的close()方法内部直接调用了ResponseBody的close()方法，无论是同步调用execute()还是异步回调onResponse()，最终都需要关闭响应体
&emsp;&emsp;对于同步调用，确保响应体被关闭的最简单的方式是使用try代码块.
```
 Call call = client.newCall(request);
 try (Response response = call.execute()) {
   ... // Use the response.
 }
```
&emsp;&emsp;对于异步调用
```
call.enqueue(new Callback() {
     public void onResponse(Call call, Response response) throws IOException {
       try (ResponseBody responseBody = response.body()) {
         ... // Use the response.
       }
     }

     public void onFailure(Call call, IOException e) {
       ... // Handle the failure.
     }
   });
```
&emsp;&emsp;
***
## 使用实例
&emsp;&emsp;上面的例子上已有Get的使用例子，这里略。

### 用POST发送String
```
    private static OkHttpClient client;
    public final static MediaType MEDIA_TYPE_JSON = MediaType.parse("application/json; charset=utf-8");
	client = new OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .readTimeout(10, TimeUnit.SECONDS)
        .build();
    new Thread(new Runnable() {
        @Override
        public void run() {
            String  postBody = "{  \n" +
                    "    \"type\":\"flight_alert\",\n" +
                    "    \"date\" : 1478498633000\n" +
                    "    \"flightNo\": \"MU336\",\n" +
                    "    \"flightDate\": \"2016-11-23\",\n" +
                    "    \"departure\": \"北京\",\n" +
                    "    \"destination\": \"上海\",\n" +
                    "    \"title\": \"离开登机口\",\n" +
                    "    \"content\": \"MU336航班已经离开登机口开始滑行。正在排队等待起飞。\",\n" +
                    "}";
            Request request = new Request.Builder()
                    .url("https://www.baidu.com/")//顺便写的url
                    .post(RequestBody.create(MEDIA_TYPE_JSON, postBody))
                    .build();
            try {
                Response response = client.newCall(request).execute();
                if(!response.isSuccessful())
                    Log.e(TAG, "Unexpected code " + response);
                Log.d(TAG, response.body().string());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }).start();

```
&emsp;&emsp;几种常见数据的MIME类型值. 详细MIME类型请参考[Media Types](https://www.iana.org/assignments/media-types/media-types.xhtml) 和 [MIME 参考手册](http://www.w3school.com.cn/media/media_mimeref.asp)
```
json ：application/json
xml：application/xml
```
### 用POST发送Stream流

```
	private static OkHttpClient client;
    public final static MediaType MEDIA_TYPE_JSON = MediaType.parse("application/json; charset=utf-8");
    String  postBody = "{  \n" +
            "    \"type\":\"flight_alert\",\n" +
            "    \"date\" : 1478498633000\n" +
            "    \"flightNo\": \"MU336\",\n" +
            "    \"flightDate\": \"2016-11-23\",\n" +
            "    \"departure\": \"北京\",\n" +
            "    \"destination\": \"上海\",\n" +
            "    \"title\": \"离开登机口\",\n" +
            "    \"content\": \"MU336航班已经离开登机口开始滑行。正在排队等待起飞。\",\n" +
            "}";
	RequestBody requestBody = new RequestBody() {
        @Override
        public MediaType contentType() {
            //返回MediaType类型
            return MEDIA_TYPE_JSON;
        }

        @Override
        public void writeTo(BufferedSink bufferedSink) throws IOException {
            //进行写入数据
            bufferedSink.writeUtf8(postBody);
            //或得到OutputStream后，通过它向bufferedSink写入数据
                /*OutputStream outputStream = bufferedSink.outputStream();
                outputStream.write(Byte.valueOf(postBody));*/
        }
    };
    Request request = new Request.Builder()
            .url("https://www.baidu.com/")
            .post(requestBody)
            .build();
        try {
        Response response = client.newCall(request).execute();
        if(!response.isSuccessful())
            Log.e(TAG, "Unexpected code " + response);
        Log.d(TAG, response.body().string());
    } catch (IOException e) {
        e.printStackTrace();
    }


```

### 用POST发送File

```
    public void PostFile() {
        File file = new File("Hello.txt");
        Request request = new Request.Builder()
                .url("https://www.baidu.com/")
                .post(RequestBody.create(MEDIA_TYPE_TXT, file))
                .build();
        try {
            Response response = client.newCall(request).execute();
            if(!response.isSuccessful())
                Log.e(TAG, "Unexpected code " + response);
            Log.d(TAG, response.body().string());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
用POST发送Form表单中的键值对
```
        RequestBody formBody = new FormBody.Builder()
                .add("type", "flight_alert")
                .add("flightNo", "MU336")
                .build();
		//发送multipart数据
		/*
		RequestBody requestBody = new MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("title", "Square Logo")
        .addFormDataPart("image", "logo-square.png",
            RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
        .build();
		*/
        Request request = new Request.Builder()
                .url("https://www.baidu.com/")
                .post(formBody)
                .build();
        try {
            Response response = client.newCall(request).execute();
            if(!response.isSuccessful())
                Log.e(TAG, "Unexpected code " + response);
            Log.d(TAG, response.body().string());
        } catch (IOException e) {
            e.printStackTrace();
        }

```
***
## 缓存响应结果
&emsp;&emsp;对于一些数据并不需要实时的或者数据是有一个效期的，我们可以对结果进行缓存，下次请求时，直接取缓存结果， 另外需要限制该缓存目录的大小。
&emsp;&emsp;需要注意的是Okhttp的缓存目录应该是私有的，不能被其他应用访问。另外多个缓存实例访问同一个缓存目录是会出错的，因此大部分的应用一般只调用一次new OkHttpClient()，然后为其配置缓存目录。

```
    private static OkHttpClient client;
    public final static MediaType MEDIA_TYPE_JSON = MediaType.parse("application/json; charset=utf-8");
    public final static MediaType MEDIA_TYPE_TXT = MediaType.parse("text/plain; charset=utf-8");
    private static final int cacheSize = 1024 * 1024 * 8;//8MB
    private static String okhttpCachePath;
    String  postBody = "{  \n" +
            "    \"type\":\"flight_alert\",\n" +
            "    \"date\" : 1478498633000\n" +
            "    \"flightNo\": \"MU336\",\n" +
            "    \"flightDate\": \"2016-11-23\",\n" +
            "    \"departure\": \"北京\",\n" +
            "    \"destination\": \"上海\",\n" +
            "    \"title\": \"离开登机口\",\n" +
            "    \"content\": \"MU336航班已经离开登机口开始滑行。正在排队等待起飞。\",\n" +
            "}";
	okhttpCachePath = getCacheDir().getPath() + File.separator + "okhttp";
        File okHttpCache = new File(okhttpCachePath);
        if(!okHttpCache.exists()){
            okHttpCache.mkdirs();
        }
        Cache cache = new Cache(okHttpCache, cacheSize);
        client = new OkHttpClient.Builder()
                .cache(cache)
                .connectTimeout(10, TimeUnit.SECONDS)
                .readTimeout(10, TimeUnit.SECONDS)
                .build();
	Request request = new Request.Builder()
            .url("https://www.baidu.com/")
			.cacheControl(CacheControl.FORCE_NETWORK)//强制发送网络请求,不使用缓存
            .cacheControl(CacheControl.FORCE_CACHE)//强制使用缓存，即使缓存数据过期，但如果缓存不存在则会返回504 Unsatisfiable Request
            .post(RequestBody.create(MEDIA_TYPE_JSON, postBody))
            .build();
    try {
        Response response = client.newCall(request).execute();
        if(!response.isSuccessful())
            Log.e(TAG, "Unexpected code " + response);
        Log.d(TAG, response.body().string());
    } catch (IOException e) {
        e.printStackTrace();
    }

```
***
## 中断请求及设置超时
### 中断请求
&emsp;&emsp;通过调用Call的cancel方法立即中止请求，注意如果线程正在写入Request或读取Response，那么会抛出IOException异常。同步请求和异步请求都可以被取消。
```
final Call call = client.newCall(request);
call.cancel();
```
### 设置超时
&emsp;&emsp;写入超时（上传数据）、读取超时（下载数据）的超时时间都默认设置为10秒。需要更长超时时间，能通过设置。
```
        client = new OkHttpClient.Builder()
                .writeTimeout(15, TimeUnit.SECONDS)
                .connectTimeout(15, TimeUnit.SECONDS)
                .readTimeout(15, TimeUnit.SECONDS)
                .build();

```
## 处理身份验证
&emsp;&emsp;有些网络请求是需要用户名密码登录的，如果没提供登录需要的信息，那么会得到401 Not Authorized未授权的错误，这时候Okhttp会自动查找是否配置了Authenticator，如果配置过Authenticator，会用Authenticator中包含的登录相关的信息构建一个新的Request，尝试再次发送HTTP请求。

```
 private final OkHttpClient client;

  public Authenticate() {
    client = new OkHttpClient.Builder()
        .authenticator(new Authenticator() {
          @Override public Request authenticate(Route route, Response response) throws IOException {
            System.out.println("Authenticating for response: " + response);
            System.out.println("Challenges: " + response.challenges());
            String credential = Credentials.basic("jesse", "password1");
            return response.request().newBuilder()
                .header("Authorization", credential)
                .build();
          }
        })
        .build();
  }

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://publicobject.com/secrets/hellosecret.txt")
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }
```
&emsp;&emsp;如果用户名密码有问题，那么Okhttp会一直用这个错误的登录信息尝试登录，我们应该判断如果之前已经用该用户名密码登录失败了，就不应该再次登录，这种情况下需要让Authenticator对象的authenticate()方法返回null，这就避免了没必要的重复尝试，代码片段如下所示：
```
	if (credential.equals(response.request().header("Authorization"))) {
		return null; 
	}
```

> 本文大部分内容整理是来自https://blog.csdn.net/iispring/article/details/51661195 大神的博客。
