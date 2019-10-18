## Android 混淆规则


### 1. 混淆简介
Android代码混淆是一种应用源代码保护技术，用来防止别人对apk进行逆向分析；从Android2.3开始，Google就在SDK中加入了ProGuard的工具，使用它来进行代码的混淆。

ProGuard是一个压缩、优化和混淆Java字节码文件的免费工具， 其作用有以下几点：
-删除代码中的注释；
-删除代码中没有用到的类、字段、方法和属性；
-会把代码中的包名、类名、方法名，变量名等修改为abcd...这种没有意义的名字，使得反编译出来的代 码难以阅读，从而达到防止apk被破解和逆向分析的目的；
-经过ProGuard混淆后，apk安装包会变小；

在实际项目中，有些Java类不能进行混淆，需要配置混淆规则；打包apk时一般都需要进行混淆处理；
另外混淆后会生成mapping.txt文件，该文件需要存档以便用来还原和查看混淆后的出错日志；

### 2. 混淆规则常用的命令：

```
    buildTypes {
        debug {
            //是否进行混淆
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        release {
            minifyEnabled true          //开启混淆只需要设置为true即可
            //添加混淆规则的位置
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

```

### 3. 混淆规则常用的指令

常用指令：
```
optimizationpasses
dontoptimize
dontusemixedcaseclassnames
dontskipnonpubliclibraryclasses
dontpreverify
dontwarn
verbose
optimizations
keep
keepnames
keepclassmembers
keepclassmembernames
keepclasseswithmembers
keepclasseswithmembernames
//更多参考官网： https://www.guardsquare.com/en/products/proguard
```

keep相关指令说明：

| 命令                      | 作用                                     |
| --------------------------- | ------------------------------------------ |
| -keep                       | 防止类和成员被移除或者被重命名 |
| -keepnames                  | 防止类和成员被重命名             |
| -keepclassmembers           | 防止成员被移除或者被重命名    |
| -keepnames                  | 防止成员被重命名                   |
| -keepclasseswithmembers     | 防止拥有该成员的类和成员被移除或者被重命名 |
| -keepclasseswithmembernames | 防止拥有该成员的类和成员被重命名 |

keep指令格式：
```
  [保持命令] [类] {
    [成员] 
}
```
“类”代表类相关的限定条件，它将最终定位到某些符合该限定条件的类。它的内容可以使用：
- 具体的类
- 访问修饰符（public、protected、private）
- 通配符*，匹配任意长度字符，但不含包名分隔符(.)
- 通配符**，匹配任意长度字符，并且包含包名分隔符(.)
- extends，即可以指定类的基类
- implement，匹配实现了某接口的类
- $，内部类

“成员”代表类成员相关的限定条件，它将最终定位到某些符合该限定条件的类成员。它的内容可以使用：
- <init> 匹配所有构造器
- <fields> 匹配所有域
- <methods> 匹配所有方法
- 通配符*，匹配任意长度字符，但不含包名分隔符(.)
- 通配符**，匹配任意长度字符，并且包含包名分隔符(.)
- 通配符***，匹配任意参数类型
- …，匹配任意长度的任意类型参数。比如void test(…)就能匹配任意 void test(String a) 或者是 void test(int a, String b) 这些方法。
- 访问修饰符（public、protected、private）

基本例子：
```
//保留该包下的类名不会被混淆
-keep class pakageName.*

//保留该包及其子包的类名不会被混淆
-keep class pakageName.**

//保留类名及其该类的内容不会被混淆（包括变量名，方法名等）
-keep class pageName.* {*;}

//不保留类名, 但保留该类的方法名、变量名等不会被混淆
-keepclassmembers class pageName.*{*;}

//保留所有继承某类的子类不会被混淆（implement同理）
-keep public class * extend android.app.Activity

//保留该内部类中所有的public方法名、变量名不会被混淆
-keepclassmembers class pageName$内部类名 { ////"$"的含义是保留某类的内部类不会被混淆
    public *;
}

-keep class pageName {                                                                                                                                                                                 
    public <init>;                                  //保留所有的public的构造方法不会被混淆
}

-keep class pageName {
    public <init>（java.lang.String）;              //保留所有的public的构造方法并且参数是String对象，不会被混淆
}

-keep class pageName {                              //保留所有的private的方法名不会被混淆
    private <methods>;
}

```

### 3. 通用的一些混淆规则

- 四大组件、Fragment、自定义控件不需要添加混淆规则, 但这些默认是不会被混淆的， 所以不必要添加

- 注解
```
-keepattributes *Annotation*
```

- R文件下面的资源

```
-keep class **.R$* {*;}
```

- 本地的native方法（JNI）
```
-keepclasseswithmembernames class * {
    native <methods>;
}
```

- 反射(该pageName是被反射类的包名)
```
-keep class pageName{*;}
```

- JavaBean中的泛型
```
-keepattributes Signature
```

- JavaBean（如果使用了Gson进行解析Json字符串，就需要添加JavaBean的混淆规则，因为Gson使用了反射的原理）
```
-keep class pageName**
-keep class pageName**{*;}
```

- 枚举
```
# 保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```

- Parcelable序列化和Creator静态成员变量
```
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}
```

- Serializable序列化
```
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
```

- WebView中JavaScript调用的方法
```
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, jav.lang.String);
}
```

- 与JS交互
```
#Android4.2以上需要添加以下的两个混淆配置
-keepattributes *Annotation*
-keepattributes *JavascriptInterface*
-keepclassmembers class pageName$内部类名 {
    public *;
}
```

- 第三方的Jar包、依赖、SDK等
这个就需要查看项目中添加了那些第三方的Jar包、依赖、SDK了，然后在其官网上或者Github找相应的混淆规则，一般都会有的，如果没有可以自己写混淆规则。根据上面介绍的一些常用的命令含义，一般都能满足自己写混淆规则了。


##### 通用的混淆规则例子：

```
# 代码混淆压缩比，在0~7之间
-optimizationpasses 5
# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames
# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses
# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
-dontpreverify
-verbose
# 避免混淆泛型
-keepattributes Signature

# 保留Annotation不混淆
-keepattributes *Annotation*,InnerClasses
#google推荐算法
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
# 避免混淆Annotation、内部类、泛型、匿名类
-keepattributes *Annotation*,InnerClasses,Signature,EnclosingMethod
# 重命名抛出异常时的文件名称
-renamesourcefileattribute SourceFile
# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable
# 处理support包
-dontnote android.support.**
-dontwarn android.support.**
# 保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

# 保留R下面的资源
-keep class **.R$* {*;}
# 保留四大组件，自定义的Application等这些类不被混淆
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.preference.Preference
-keep public class com.android.vending.licensing.ILicensingService

# 保留在Activity中的方法参数是view的方法，
# 这样以来我们在layout中写的onClick就不会被影响
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}
# 对于带有回调函数的onXXEvent、**On*Listener的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
    void *(**On*Listener);
}
# 保留本地native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留Parcelable序列化类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

-keepclassmembers class * implements java.io.Serializable {
   static final long serialVersionUID;
   private static final java.io.ObjectStreamField[]   serialPersistentFields;
   private void writeObject(java.io.ObjectOutputStream);
   private void readObject(java.io.ObjectInputStream);
   java.lang.Object writeReplace();
   java.lang.Object readResolve();
}
#assume no side effects:删除android.util.Log输出的日志
-assumenosideeffects class android.util.Log {
    public static *** v(...);
    public static *** d(...);
    public static *** i(...);
    public static *** w(...);
    public static *** e(...);
}
#保留Keep注解的类名和方法
-keep,allowobfuscation @interface android.support.annotation.Keep
-keep @android.support.annotation.Keep class *
-keepclassmembers class * {
    @android.support.annotation.Keep *;
}
```

### 4.自定义资源保持规则

用shrinkResources true开启资源压缩后，所有未被使用的资源默认被移除。假如你需要定义哪些资源必须被保留，在 res/raw/ 路径下创建一个 xml 文件，例如keep.xml 。

通过一些属性的设置可以实现定义资源保持的需求，可配置的属性有：

- keep 定义哪些资源需要被保留（资源之间用“,”隔开）
- discard 定义哪些资源需要被移除（资源之间用“,”隔开）
- shrinkMode 开启严格模式
- 当代码中通过Resources.getIdentifier() 用动态的字符串来获取并使用资源时，普通的资源引用检查就可能会有问题。例如，如下代码会导致所有以“img_”开头的资源都被标记为已使用。
- 当代码中通过 Resources.getIdentifier() 用动态的字符串来获取并使用资源时，普通的资源引用检查就可能会有问题。例如，如下代码会导致所有以“img_”开头的资源都被标记为已使用。
```
String name = String.format("img_%1d", angle + 1);
res = getResources().getIdentifier(name, "drawable", getPackageName());
```
我们可以设置 tools:shrinkMode 为strict来开启严格模式，使只有确实被使用的资源被保留。

举个例子：
```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
    tools:discard="@layout/unused2"
    tools:shrinkMode="strict"/>
```


### 5. 检查混淆结果及解出混淆栈

##### 检查混淆结果
需要从代码层面检查。使用上文的配置进行混淆打包后在<module-name>/build/outputs/mapping/release/目录下会输出以下文件：
```
dump.txt    // 描述APK文件中所有类的内部结构
mapping.txt // 提供混淆前后类、方法、类成员等的对照表
seeds.txt   // 列出没有被混淆的类和成员
usage.txt   // 列出被移除的代码
```
我们可以根据 seeds.txt 文件检查未被混淆的类和成员中是否已包含所有期望保留的，再根据 usage.txt文件查看是否有被误移除的代码。

##### 解出混淆栈
混淆后的类、方法名等等难以阅读，这固然会增加逆向工程的难度，但对追踪线上 crash 也造成了阻碍。我们拿到 crash 的堆栈信息后会发现很难定位，这时需要将混淆反解。

在 <sdk-root>/tools/proguard/路径下有附带的的反解工具（Window 系统为proguardgui.bat，Mac 或 Linux 系统为proguardgui.sh）。

这里以 Window 平台为例。双击运行 proguardgui.bat 后，可以看到左侧的一行菜单。点击 ReTrace，选择该混淆包对应的 mapping 文件（混淆后在 <module-name>/build/outputs/mapping/release/ 路径下会生成 mapping.txt 文件，它的作用是提供混淆前后类、方法、类成员等的对照表），再将 crash 的 stack trace 黏贴进输入框中，点击右下角的 ReTrace ，混淆后的堆栈信息就显示出来了。
以上使用 GUI 程序进行操作，另一种方式是利用该路径下的 retrace 工具通过命令行进行反解，命令是
```
retrace.bat|retrace.sh [-verbose] mapping.txt [<stacktrace_file>]
```
例如：
```
retrace.bat -verbose mapping.txt obfuscated_trace.txt
```
注意事项：

所有在 AndroidManifest.xml 涉及到的类已经自动被保持，因此不用特意去添加这块混淆规则。（很多老的混淆文件里会加，现在已经没必要）; 

proguard-android.txt已经存在一些默认混淆规则，没必要在 proguard-rules.pro 重复添加

本文学习转直接复制相关内容到此笔记的文章:
> [Android 代码混淆 混淆方案](https://www.jianshu.com/p/e9d3c57ab92f?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=qq)

> [Android添加混淆规则](https://www.jianshu.com/p/8a2beaf5432f?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
