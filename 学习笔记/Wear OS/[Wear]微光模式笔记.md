# Wear OS的微光模式笔记

某些 Wear OS 应用对用户始终可见， 搭载 Android 5.1 或更高版本的 Wear OS 设备可以让应用保持在前台运行，同时节省电池电量。Wear OS 应用可以控制手表在低功耗（微光）模式下显示的内容。在微光模式和交互模式下均运行的 Wear 应用称为始终开启的应用。

> **重要提示**：27.1.0 版的 Android 支持库提供了一种新的方式来支持微光模式，这种方式使用 `AmbientModeSupport` 类 **，而不是** `WearableActivity` 类。您可以决定是使用这种[全新的首选方式](https://developer.android.com/training/wearables/apps/always-on?hl=zh-cn#ambient-mode-class)来支持微光模式，还是 [扩展 WearableActivity 类](https://developer.android.com/training/wearables/apps/always-on?hl=zh-cn#ambient-mode-using-the-wearableactivity-class)。
>
> **注意**：[`AmbientMode`](https://developer.android.com/reference/androidx/wear/ambient/AmbientMode?hl=zh-cn) 类已被弃用，取而代之的是 [`AmbientModeSupport`](https://developer.android.com/reference/androidx/wear/ambient/AmbientModeSupport?hl=zh-cn) 类。

如果用户在应用显示期间有一段时间未与应用进行交互，或者用户用手掌遮住屏幕，系统就会将 Activity 切换到微光模式。

## Gradle 配置

添加WAKE_LOCK权限：

```groovy
    <uses-permission android:name="android.permission.WAKE_LOCK" />
```

## 使用 AmbientModeSupport 类支持微光模式

如果使用 [`AmbientModeSupport`](https://developer.android.com/reference/androidx/wear/ambient/AmbientModeSupport?hl=zh-cn) 类支持微光模式，您可以从以下方面受益：

- Android 支持库中的 [`Activity`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn) 子类，例如 [`FragmentActivity`](https://developer.android.com/reference/androidx/fragment/app/FragmentActivity?hl=zh-cn)；可使用 Fragment 的功能
- [架构组件](https://developer.android.com/topic/libraries/architecture?hl=zh-cn)，此类组件属于[生命周期感知型](https://developer.android.com/topic/libraries/architecture/lifecycle?hl=zh-cn)组件
- 更好地支持 [Google 登录](https://developers.google.com/identity/sign-in/android/sign-in?hl=zh-cn)

如需使用 [`AmbientModeSupport`](https://developer.android.com/reference/androidx/wear/ambient/AmbientModeSupport?hl=zh-cn) 类，您可以扩展某个 [`FragmentActivity`](https://developer.android.com/reference/androidx/fragment/app/FragmentActivity?hl=zh-cn) 子类或 FragmentActivity 本身并实现一个提供程序接口，该接口又可用于监听微光模式更新。

- 创建继承于`FragmentActivity`类的 `Activity`

- 其`Activity`实现`AmbientCallbackProvider` 接口

  ```kotlin
      class MainActivity : AppCompatActivity(), AmbientModeSupport.AmbientCallbackProvider {
          …
          override fun getAmbientCallback(): AmbientModeSupport.AmbientCallback = MyAmbientCallback()
          …
      }
  ```

  而`MyAmbientCallback`则是实现`AmbientCallback`接口的内容类， 可对微光事件进行操作。

  ```kotlin
      private class MyAmbientCallback : AmbientModeSupport.AmbientCallback() {
  
          override fun onEnterAmbient(ambientDetails: Bundle?) {
            // Handle entering ambient mode
          }
  
          override fun onExitAmbient() {
            // Handle exiting ambient mode
          }
  
          override fun onUpdateAmbient() {
            // Update the content
          }
      }
  ```

- 注册`AmbientCallbackProvider` 接口的实例
  在`Activity`的`Oncreate()`方法， 通过通过调用 `AmbientModeSupport.attach(FragmentActivity)` 启用微光模式。此方法会返回 `AmbientModeSupport.AmbientController`。您可以通过该控制器检查回调之外的微光状态，您可能需要保留对 `AmbientModeSupport.AmbientController` 对象的引用：

  ```kotlin
      class MainActivity : AppCompatActivity(), AmbientModeSupport.AmbientCallbackProvider {
          ...
          /*
           * Declare an ambient mode controller, which will be used by
           * the activity to determine if the current mode is ambient.
           */
          private lateinit var ambientController: AmbientModeSupport.AmbientController
          ...
          override fun onCreate(savedInstanceState: Bundle?) {
              super.onCreate(savedInstanceState)
              setContentView(R.layout.activity_main)
              ...
              ambientController = AmbientModeSupport.attach(this)
          }
          ...
      }
  ```

## 使用 WearableActivity 类支持微光模式

虽然`AmbientMode`已弃用，但目前仍然可使用；

- 创建扩展 [`WearableActivity`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn) 的 Activity。
- 在您的 Activity 的 [`onCreate()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onCreate(android.os.Bundle)) 方法中，调用 [`setAmbientEnabled()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#setAmbientEnabled()) 方法。

```java
    public class MainActivity extends WearableActivity {
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setAmbientEnabled();
            ...
        }
        
        /** * 進入微光模式 */
        override fun onEnterAmbient(ambientDetails: Bundle?) {
            ...
        }

        /** * 退出微光模式 */
        override fun onExitAmbient() {
            ....
        }
        
        /** * 微光模式下，更新數據 */
        fun onUpdateAmbient() {
            //refresh Data
        }
    }
```

## 进阶使用

### 模式切换

在应用切换到微光模式后，将 Activity 界面更新为更基本的布局以减少耗电量。您应该使用黑色背景和最简单的白色图形和文本。为了方便用户从交互模式转换到微光模式，请尽量在屏幕上保持相似的项目布局。要详细了解如何在微光屏幕上呈现内容，请参阅 [Wear OS 表盘](https://developer.android.com/design/wear/watchfaces?hl=zh-cn#DisplayModes)设计指南。

> **注意**：在微光模式下，应停用屏幕上的所有交互元素，如按钮。如需详细了解如何为始终开启应用设计用户交互，请参阅 [Wear OS 应用结构](https://developer.android.com/design/wear/structure?hl=zh-cn#AlwaysOn)设计指南。

当 Activity 切换到微光模式时，系统会调用您的穿戴式设备 Activity 中的 [`onEnterAmbient()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#onEnterAmbient(android.os.Bundle)) 方法。以下代码段展示了如何在系统切换到微光模式后将**文本颜色更改为白色并停用抗锯齿**：

```kotlin
    override fun onEnterAmbient(ambientDetails: Bundle?) {
        super.onEnterAmbient(ambientDetails)

        stateTextView.setTextColor(Color.WHITE)
        stateTextView.paint.isAntiAlias = false
    }
    
```

当用户点按屏幕或抬起手腕时，Activity 会从微光模式切换到交互模式。系统会调用 [`onExitAmbient()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#onExitAmbient()) 方法。替换此方法以更新界面布局，让您的应用在全彩色交互状态下显示:

```kotlin
    override fun onExitAmbient() {
        super.onExitAmbient()

        stateTextView.setTextColor(Color.GREEN)
        stateTextView.paint.isAntiAlias = true
    }
```

### 微光模式下更新内容

微光模式允许您用提供给用户的新信息更新屏幕，但您必须小心地在显示更新与电池续航时间之间取得平衡。您应认真考虑仅替换 `onUpdateAmbient()` 方法，以在微光模式下实现每分钟更新一次屏幕。如果您的应用需要更频繁的更新，您需要考虑电池续航时间与更新频率之间的取舍。为了节省电池电量，更新频率不应超过每 10 秒一次。但实际上，您更新应用的频率应小于该值。

#### 每分钟更新一次

系统提供了一个回调方法 [`onUpdateAmbient()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#onUpdateAmbient())，允许您按照建议的这个频率(每分钟更新，个别系统可能不一样)更新屏幕；

```kotlin
    override fun onUpdateAmbient() {
        super.onUpdateAmbient()
        // Update the content
    }
```

#### 更频繁地更新

您可以按照高于每分钟一次的频率在微光模式下更新 Wear 应用，不过不建议这样做。对于需要更频繁地更新的应用，您可以使用 [`AlarmManager`](https://developer.android.com/reference/android/app/AlarmManager?hl=zh-cn) 对象唤醒处理器并更频繁地更新屏幕。

> **注意**：闹钟管理器可能会在触发时创建 Activity 的新实例。为防止出现这种情况，请确保在清单中使用 `android:launchMode="singleInstance"` 参数声明您的 Activity。

如需实现一个闹钟以在微光模式下更频繁地更新内容，请按以下步骤操作：

- 准备闹钟管理器
  闹钟管理器会启动一个待定 intent，用来更新屏幕并安排下次闹钟时间; 

  ```kotlin
      // Action for updating the display in ambient mode, per our custom refresh cycle.
      private const val AMBIENT_UPDATE_ACTION = "com.your.package.action.AMBIENT_UPDATE"
      ...
      private lateinit var ambientUpdateAlarmManager: AlarmManager
      private lateinit var ambientUpdatePendingIntent: PendingIntent
      private lateinit var ambientUpdateBroadcastReceiver: BroadcastReceiver
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
  
          setAmbientEnabled()
  
          ambientUpdateAlarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
  
          ambientUpdatePendingIntent = Intent(AMBIENT_UPDATE_ACTION).let { ambientUpdateIntent ->
              PendingIntent.getBroadcast(this, 0, ambientUpdateIntent, PendingIntent.FLAG_UPDATE_CURRENT)
          }
  
          ambientUpdateBroadcastReceiver = object : BroadcastReceiver() {
              override fun onReceive(context: Context, intent: Intent) {
                  refreshDisplayAndSetNextUpdate()
              }
          }
          ...
      }
  ```

  分别在 `onResume()` 和 `onPause()` 中注册和取消注册广播接收器：

  ```kotlin
      override fun onResume() {
          super.onResume()
          IntentFilter(AMBIENT_UPDATE_ACTION).also { filter ->
              registerReceiver(ambientUpdateBroadcastReceiver, filter)
          }
      }
  
      override fun onPause() {
          super.onPause()
          unregisterReceiver(ambientUpdateBroadcastReceiver)
          ambientUpdateAlarmManager.cancel(ambientUpdatePendingIntent)
      }
  ```

  

- 设置更新的频率:
  闹钟管理器在微光模式下每 20 秒触发一次。当计时器开始计时的时候，闹钟就会触发 intent 来更新屏幕，然后设置下次更新的延迟。

  ```kotlin
      // Milliseconds between waking processor/screen for updates
      private val AMBIENT_INTERVAL_MS: Long = TimeUnit.SECONDS.toMillis(20)
      ...
      private fun refreshDisplayAndSetNextUpdate() {
          if (isAmbient) {
              // Implement data retrieval and update the screen for ambient mode
          } else {
              // Implement data retrieval and update the screen for interactive mode
          }
          val timeMs: Long = System.currentTimeMillis()
          // Schedule a new alarm
          if (isAmbient) {
              // Calculate the next trigger time
              val delayMs: Long = AMBIENT_INTERVAL_MS - timeMs % AMBIENT_INTERVAL_MS
              val triggerTimeMs: Long = timeMs + delayMs
              ambientUpdateAlarmManager.setExact(
                      AlarmManager.RTC_WAKEUP,
                      triggerTimeMs,
                      ambientUpdatePendingIntent)
          } else {
              // Calculate the next trigger time for interactive mode
          }
      }
  ```

- 当 Activity 切换到微光模式或当前处于微光模式时安排下一次更新; 
  通过替换 [`onEnterAmbient()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#onEnterAmbient(android.os.Bundle)) 方法和 [`onUpdateAmbient()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#onUpdateAmbient()) 方法，安排闹钟在 Activity 进入微光模式或 Activity 已处于微光模式时更新屏幕：

  ```kotlin
      override fun onEnterAmbient(ambientDetails: Bundle?) {
          super.onEnterAmbient(ambientDetails)
  
          refreshDisplayAndSetNextUpdate()
      }
  
      override fun onUpdateAmbient() {
          super.onUpdateAmbient()
          refreshDisplayAndSetNextUpdate()
      }
      
  ```

  

- 当 Activity 切换到交互模式或停止时取消闹钟。
  当设备切换到交互模式时，在 [`onExitAmbient()`](https://developer.android.com/reference/android/support/wearable/activity/WearableActivity?hl=zh-cn#onExitAmbient()) 方法中取消闹钟：

  ```kotlin
      override fun onExitAmbient() {
          super.onExitAmbient()
  
          ambientUpdateAlarmManager.cancel(ambientUpdatePendingIntent)
      }
      
  ```

  当用户退出或停止 Activity 时，在 Activity 的 [`onDestroy()`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onDestroy()) 方法中取消闹钟：

  ```kotlin
      override fun onDestroy() {
          ambientUpdateAlarmManager.cancel(ambientUpdatePendingIntent)
          super.onDestroy()
      }
  ```

### 保持向后兼容性

在搭载低于 5.1（API 级别 22）版的 Android 系统的 Wear 设备上，支持微光模式的 Activity 会自动回退为普通 Activity。不需要特殊的应用代码来支持搭载这些 Android 版本的设备。当设备切换到微光模式时，设备会返回主屏幕并退出您的 Activity。

如果您的应用不应在搭载版本低于 5.1 的 Android 设备上安装或更新，请使用以下代码更新您的清单：

```xml
    <uses-library android:name="com.google.android.wearable" android:required="true" />    
```



https://developer.android.com/training/wearables/apps/always-on?hl=zh-cn#kotlin