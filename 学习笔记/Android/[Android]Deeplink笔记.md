## [Android]Deeplink笔记

### DeepLink的使用

#### 配置Activity的Androidmanifest文件

```xml
        <activity android:name=".phone.cardlist.deeplink.DeepLinkActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="steven" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:host="steven"
                    android:scheme="app" />
            </intent-filter>
        </activity>
```

1. 声明action为action.VIEW确保能够应用能够接受到deeplink请求。
2. deeplink样式和URL样式相同，如示例代码中的deeplink就是 steven:// 或者 app://steven。
3. 如果包含有多个deeplink，则可以声明多个intent-filter，变更启动的data配置即可。

一般编写deeplink的格式如下：

```
[scheme]://[host]/[path]?[query]
```

比如点击H5的按钮进行跳转app:

```html
<a href='app://steven/myDemo?id=yufen&extra_title_string=Test&uri=www.baidu.com'>DeppLink Test</a>
```



### Deeplink adb 测试

为了能够更加快速方便的测试deeplink是否起到了对应的效果，我们可以采用adb指令的方式来快速访问deeplink。

```shell
adb shell am start -W -a android.intent.action.VIEW -d "具体的deeplink地址" packageName
```

