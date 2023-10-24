## TargetSdk升级到33笔记

[TOC]

## 1. 权限

### 1.1 通知运行时权限

从Target SDK 升级到33+时， 发送通知或者使用OngoingActivity来发送持续性通知时，需要动态申请权限：`POST_NOTIFICATIONS`权限；

```xml
<manifest ...>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    <application ...>
        ...
    </application>
</manifest>
```

对于新安装的app(TargetSDK 33+)， 该权限默认是关，就是需要用户同意该权限才能使用； 但对于升级上来的app， 系统会自动允许该通知权限，不需要再向用户申请该权限；

### 1.2 后台身体传感器权限变更

Android 13 以及 Wear OS 4 开发者预览版引入了一种方式，可让应用从后台访问身体传感器，如心率、体温和血氧百分比。这种新的访问模式类似于为 [Android 10（API 级别 29）中的后台位置信息访问权限](https://developer.android.com/training/location/permissions?hl=zh-cn#request-background-location)引入的访问模式，它要求应用请求 [`BODY_SENSORS_BACKGROUND`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BODY_SENSORS_BACKGROUND) 权限。 `BODY_SENSORS_BACKGROUND` 是受限权限，除非安装程序将该权限列入许可名单，或用户允许您的应用拥有该权限，否则应用无法拥有该权限。

```xml
<uses-permission android:name="android.permission.BODY_SENSORS">
<uses-permission android:name="android.permission.BODY_SENSORS_BACKGROUND">
```

然后，您的应用必须先[请求](https://developer.android.com/training/permissions/requesting?hl=zh-cn)身体传感器访问权限，再请求后台传感器访问权限。

### 1.3 用户可只允许访问大概位置

从Andorid 12(API 31+)开始， 即使app申请精确位置权限(`ACCESS_FINE_LOCATION`)， 但用户可只允许访问大概位置；

> 注： 申请精确位置权限，不能只申请`ACCESS_FINE_LOCATION`，还需要同时申请 `ACCESS_COARSE_LOCATION`， 不然有些版本会直接无视该权限请求；

更详细，请参考：https://developer.android.com/training/location/permissions#approximate-request

### 1.4 Android 上的软件包可见性过滤

如果应用以 Android 11（API 级别 30）或更高版本为目标平台，并查询与设备上已安装的其他应用相关的信息，则系统在默认情况下会过滤此信息。此过滤行为意味着您的应用无法检测设备上安装的所有应用，这有助于最大限度地减少您的应用可以访问但在执行其用例时不需要的潜在敏感信息。

有限的应用可见性会影响提供其他应用相关信息的方法的返回结果，例如 [`queryIntentActivities()`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#queryIntentActivities(android.content.Intent, int))、[`getPackageInfo()`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#getPackageInfo(java.lang.String, int)) 和 [`getInstalledApplications()`](https://developer.android.com/reference/android/content/pm/PackageManager?hl=zh-cn#getInstalledApplications(int))。有限的可见性还会影响与其他应用的显式交互，例如启动另一个应用的服务。

部分软件包是[自动可见](https://developer.android.com/training/package-visibility/automatic?hl=zh-cn)的。您的应用始终可以在查询其他已安装的应用时检测这些软件包。如需查看其他软件包，请使用 [``](https://developer.android.com/guide/topics/manifest/queries-element?hl=zh-cn) 元素[声明您的应用需要提高软件包可见性](https://developer.android.com/training/package-visibility/declaring?hl=zh-cn)。[用例](https://developer.android.com/training/package-visibility/use-cases?hl=zh-cn)页面提供了有关如何选择性地扩展软件包可见性的示例。其中介绍的工作流可让您在保护用户隐私的同时执行常见的应用交互场景。

在极少数情况下，如果遇到 `` 元素无法提供适当的软件包可见性，您还可以使用 `QUERY_ALL_PACKAGES` 权限。如果您在 Google Play 上发布应用，那么应用使用此权限[需要获得批准](https://support.google.com/googleplay/android-developer/answer/10158779?hl=zh-cn)。

### 1.4 精确闹钟权限(Exact alarm permission)

从TargetSDK是Android 12+，需要精确闹钟的，需要添加下面权限声明(暂不需要动态申请)

```xml
<manifest ...>
    <uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"/>
    <application ...>
        ...
    </application>
</manifest>
```

更详细闹钟相关信息，请参考https://developer.android.com/training/scheduling/alarms

## 2. App组件和导航

### 2.1 App 组件 export 声明

每个组件必须显式声明其`android:exported`为`true`或`false`; 如:

```xml
<service android:name="com.example.app.backgroundService"
         android:exported="false">
    <intent-filter>
        <action android:name="com.example.app.START_BACKGROUND" />
    </intent-filter>
</service>
```

### 2.2 Intent filters会阻塞不匹配的Intent

当我们发一个Intent给 某个App(其TargetSDK已是Android13+)时， 如果该Intent不匹配目标组件的 ` <intent-filter> ` 时，会直接抛出一个 ` ActivityNotFoundException`; 

### 2.3 前台服务启动限制

当App的Target是Android 12+(API 31+)时，除了特殊情况外，在后台不允许启动前台服务；非要起来的话，会报'ForegroundServiceStartNotAllowedException'异常；

例外详细请参考：https://developer.android.com/guide/components/foreground-services#background-start-restriction-exemptions

### 2.4 返回键行为更改

在Android 12+后， 用户按返回键后，系统不会再像之前那边直接去结束该Activity， 而是将该Activity移到后台Activity栈中；

更多请参考：https://developer.android.com/about/versions/12/behavior-changes-all#back-press

### 2.5 App links verification

https://developer.android.com/training/app-links/verify-android-applinks



## 3. Power and data management

### 3.1 [Backup & Restore](https://developer.android.com/about/versions/12/behavior-changes-12#backup-restore)

### [3.2 "Restricted" App Standby Bucket](https://developer.android.com/about/versions/12/behavior-changes-all#restrictive-app-standby-bucket)

###  3.3[App hibernation](https://developer.android.com/topic/performance/app-hibernation)