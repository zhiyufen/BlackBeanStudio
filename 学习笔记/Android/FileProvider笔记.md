## FileProvider笔记

Android 7.0强制启用了被称作 [StrictMode](https://link.jianshu.com?t=https://developer.android.com/reference/android/os/StrictMode.html)的策略，带来的影响就是你的App对外无法暴露`file://`类型的**URI**了。

如果你使用**Intent**携带这样的**URI**去打开外部App(比如：打开系统相机拍照)，那么会抛出**FileUriExposedException**异常。

### 使用FileProvider

**FileProvider**使用大概分为以下几个步骤：

1. manifest中申明FileProvider
2. res/xml中定义对外暴露的文件夹路径
3. 生成content://类型的Uri
4. 给Uri授予临时权限
5. 使用Intent传递Uri

#### manifest中申明FileProvider

```xml
<manifest>
  ...
  <application>
    ...
    <provider
        android:name="androidx.core.content.FileProvider"
        android:authorities="com.yufen.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"/>
    </provider>
    ...
  </application>
</manifest>
```

- android:name：`provider`你可以使用`v4`包提供的FileProvider，或者自定义您自己的，只需要在`name`申明就好了，一般使用系统的就足够了。
- android:authorities：类似`schema`，命名空间之类，后面会用到。
- android:exported：`false`表示我们的`provider`不需要对外开放。
- android:grantUriPermissions：申明为`true`，你才能获取临时共享权限。

#### res/xml中定义对外暴露的文件夹路径

新建`file_paths.xml`，文件名随便起，后面会引用到。

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
  <files-path name="my_images" path="images"/>
</paths>
```

**name：**一个引用字符串。
 **path：**文件夹“相对路径”，完整路径取决于当前的标签类型。

> path可以为空，表示指定目录下的所有文件、文件夹都可以被共享。

``这个元素内可以包含以下**一个或多个**，具体如下：

```xml
<files-path name="name" path="path" />
```

物理路径相当于[Context.getFilesDir()](https://link.jianshu.com?t=https://developer.android.com/reference/android/content/Context.html#getFilesDir()) + /path/。

```xml
<cache-path name="name" path="path" />
```

物理路径相当于[Context.getCacheDir()](https://link.jianshu.com?t=https://developer.android.com/reference/android/content/Context.html#getFilesDir()) + /path/。

```dart
<external-path name="name" path="path" />
```

物理路径相当于[Environment.getExternalStorageDirectory()](https://link.jianshu.com?t=https://developer.android.com/reference/android/os/Environment.html#getExternalStorageDirectory()) + /path/。

```dart
<external-files-path name="name" path="path" />
```

物理路径相当于**Context.getExternalFilesDir(String) **+ /path/。

```dart
<external-cache-path name="name" path="path" />
```

物理路径相当于[Context.getExternalCacheDir()](https://link.jianshu.com?t=https://developer.android.com/reference/android/content/Context.html#getExternalCacheDir()) + /path/。

> 注意：external-cache-path在support-v4:24.0.0这个版本并未支持，直到support-v4:25.0.0才支持，最近适配才发现这个坑!!!

**番外：**以上是官方提供的几种path类型，不过如果你想使用外置**SD**卡，可以用这个：

```xml
<root-path name="name" path="path" />
```

#### 生成content://类型的Uri

我们需要生成`content://xxx`类型的**Uri**，方法就是通过`Context.getUriForFile`来实现：

```dart
File imagePath = new File(Context.getFilesDir(), "images");
File newFile = new File(imagePath, "default_image.jpg");
Uri contentUri = getUriForFile(getContext(), 
                 "com.mydomain.fileprovider", newFile);
```

**imagePath：**使用的路径需要和你在`file_paths.xml`申明的其中一个符合(**或者子文件夹："images/work"**)。当然，你可以申明**N**个你需要共享的路径：

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">    
  <files-path name="my_images" path="images"/>    
  <files-path name="my_docs" path="docs"/>
  <external-files-path name="my_video" path="video" />
  //...
</paths>
```

**getUriForFile：**第一个参数是**Context**；第二个参数，就是我们之前在`manifest#provider`中定义的`android:authorities`属性的值；第三个参数是**File**。。

#### 给Uri授予临时权限

```java
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION
               | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);  
```

**FLAG_GRANT_READ_URI_PERMISSION：**表示读取权限；
 **FLAG_GRANT_WRITE_URI_PERMISSION：**表示写入权限。

你可以同时或单独使用这两个权限，视你的需求而定。

#### 使用Intent传递Uri

以拍照代码作为示例，需要这样改写：

```dart
// 重新构造Uri：content://
File imagePath = new File(Context.getFilesDir(), "images");
if (!imagePath.exists()){imagePath.mkdirs();}
File newFile = new File(imagePath, "default_image.jpg");
Uri contentUri = getUriForFile(getContext(), 
                 "com.mydomain.fileprovider", newFile);
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
intent.putExtra(MediaStore.EXTRA_OUTPUT, contentUri);
// 授予目录临时共享权限
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION
               | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
startActivityForResult(intent, 100);
```





本文笔记于：

> https://www.jianshu.com/p/55eae30d133c