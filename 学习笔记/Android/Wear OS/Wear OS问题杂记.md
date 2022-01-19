# Wear OS问题杂记

[TOC]

## 1. 手机尝试向手表发送DataItem时，无法发送成功；

- Log 显示：

  ```kotlin
  Task failed: java.util.concurrent.ExecutionException: com.google.android.gms.common.api.ApiException: 17: API: Wearable.API is not available on this device.
  ```

  问题原因：在AndroidManifest.xml里meta-data配置的gms版本不对, 我这边项目之前不用gms, 有人直接自定义一个google_play_services_version值，导致没用到最新配置gms对应的版本；

  ```xml
          <meta-data
              android:name="com.google.android.gms.version"
              android:value="@integer/google_play_services_version" />
  ```

  ```groovy
      // Wear App
      implementation 'com.google.android.gms:play-services-wearable:17.0.0'
  ```

  解决方案：

  - 删除本地自定义的google_play_services_version值，直接用gms包里定义的；

  其它原因：

  - 手表里的包名和签名不一样； 
  - putDataItem时， putDataMapRequest.dataMap里没相应的数据；
  - putDataItem时，  putDataMapRequest.dataMap里的数据没变， 重复发送，Wear OS不会处理；

## 2. 连接时，手表端WearableListenerService无法接收Capability变化的回调（onCapabilityChanged）

在开发功能，修改现有 wear.xml 里的能力字符串时， 由于WearOs系统里有缓存， 导致该能力并不会公布出去，系统里还是保存旧的能力字符串，因此，如有需要修改(新增应该可以的)， 修改完后，需要重启一下手机；



