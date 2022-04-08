## OnGoingActivity笔记

[TOC]

### 简介

![image](https://developer.android.com/codelabs/ongoing-activity/img/cb2a383284f10acd.gif?hl=zh_cn)

在表盘中，您会看到一个动画图标（针对计时器），如果用户点按该图标，系统会启动为计时器提供支持的应用。

**持续性活动**是 Wear OS 中的一项新功能，可以让**持续性通知**显示于 Wear OS 界面内的其他界面中，从而提高用户与长时间进行的活动之间的互动度。

> 持续性通知通常用于表示该通知存在用户正与之积极互动的后台任务（例如，播放音乐），或者存在正等待进行某种处理并因此占用设备的后台任务（例如，文件下载、同步操作或活跃的网络连接）。

利用 **Ongoing Activity** API，应用就可以将信息显示于 Wear OS 中的多个新界面中，方便用户保持互动。

使用持续性活动时，应用信息会显示于新界面中，但原始通知将不再显示于通知栏中。

> 持续性活动仅适用于 Android API 级别 30 及更高版本。在其他版本的平台上，该功能将被忽略并显示为正常的持续性通知。

### 使用

#### 依赖

```groovy
// TODO: Review dependencies for Ongoing Activity.
implementation "androidx.wear:wear-ongoing:1.0.0-beta01"
// Includes LocusIdCompat and new Notification categories for Ongoing Activity.
implementation "androidx.core:core-ktx:1.6.0"
```

第一个依赖项是使用 Wear OS Ongoing Activity API 所必需的。

第二个依赖项是为了获取通知 API 的最新功能，这些功能支持与持续性活动结合使用的各项功能。以下两个功能适用于持续性通知，因此也适用于持续性活动：

- [**类别**](https://developer.android.com/training/notify-user/build-notification?hl=zh_cn#system-category) - Android 使用一些预定义的系统范围类别来确定当用户启用了勿扰模式时，是否发出给定通知干扰客户。类别决定了持续性活动在表盘上的优先级。最近，新增了一些类别来支持 Wear。
- [**LocusId**](https://developer.android.com/reference/android/content/LocusId?hl=zh_cn) - Locus 是 Android 10（API 级别 29）中引入的新概念，可以让 Android 系统在内容捕获、**快捷方式**和**通知**等不同子系统之间关联状态。如果您有多个启动器，可以使用 Locus ID 将持续性活动绑定到特定的动态 [`Shortcut`](https://developer.android.com/guide/topics/ui/shortcuts?hl=zh_cn)，使其可正确显示于应用启动器的“Recents”部分。

#### 创建持续性通知

```kotlin
// TODO: Review Notification builder code.
        val notificationBuilder = notificationCompatBuilder
            .setStyle(bigTextStyle)
            .setContentTitle(titleText)
            .setContentText(mainText)
            .setSmallIcon(R.mipmap.ic_launcher)
            .setDefaults(NotificationCompat.DEFAULT_ALL)
            // Makes Notification an Ongoing Notification (a Notification with a background task).
            .setOngoing(true)
            // For an Ongoing Activity, used to decide priority on the watch face.
            .setCategory(NotificationCompat.CATEGORY_WORKOUT)
            .setVisibility(NotificationCompat.VISIBILITY_PUBLIC)
            .addAction(
                R.drawable.ic_walk, getString(R.string.launch_activity),
                activityPendingIntent
            )
            .addAction(
                R.drawable.ic_cancel,
                getString(R.string.stop_walking_workout_notification_text),
                servicePendingIntent
            )

        return notificationBuilder.build()
```

在没离开该应用时，使用：

```kotlin
notificationManager.notify(NOTIFICATION_ID, notification)
```

离开界面后,转为前台Service：

```kotlin
// startForeground takes care of notificationManager.notify(...).
startForeground(NOTIFICATION_ID, notification)
```

关注通知构建器中的 `setOngoing()` 调用和 `setCategory()`。

- `setCategory()`类别可以帮助 Wear OS 确定通知在表盘上的优先级。
- `setOngoing`() 调用将通知设为持续性通知，也就是说，该通知存在用户正与之积极互动的后台任务，例如跟踪步行锻炼。

当用户在进行步行锻炼时离开 `MainActivity`，我们就会创建此通知。

尝试滑动退出应用。如果您转到表盘下方的导航栏，应该能找到仍在继续跟踪步行分数的持续性通知。

点按后显示的内容如下所示：

![990814a9f59a510a.png](https://developer.android.com/codelabs/ongoing-activity/img/990814a9f59a510a.png?hl=zh_cn)

![ac59318b7243c403.png](https://developer.android.com/codelabs/ongoing-activity/img/ac59318b7243c403.png?hl=zh_cn)

注： 目前手上的Watch，没办法显示上面的行为；

#### 创建持续性活动

**持续性活动**必须与**持续性通知**相关联。我们已经有了持续性通知，所以现在要创建一个持续性活动。

```kotlin
        // TODO: Create an Ongoing Activity.
        val ongoingActivityStatus = Status.Builder()
            // Sets the text used across various surfaces.
            .addTemplate(mainText)
            .build()

        val ongoingActivity =
            OngoingActivity.Builder(applicationContext, NOTIFICATION_ID, notificationBuilder)
                // Sets icon that will appear on the watch face in active mode. If it isn't set,
                // the watch face will use the static icon in active mode.
                // Supported animated icon types: AVD and AnimationDrawable.
                .setAnimatedIcon(R.drawable.animated_walk)
                // Sets the icon that will appear on the watch face in ambient mode.
                // Falls back to Notification's smallIcon if not set. If neither is set,
                // an Exception is thrown.
                .setStaticIcon(R.drawable.ic_walk)
                // Sets the tap/touch event, so users can re-enter your app from the
                // other surfaces.
                // Falls back to Notification's contentIntent if not set. If neither is set,
                // an Exception is thrown.
                .setTouchIntent(activityPendingIntent)
                // In our case, sets the text used for the Ongoing Activity (more options are
                // available for timers and stop watches).
                .setStatus(ongoingActivityStatus)
                .build()

        // Applies any Ongoing Activity updates to the notification builder.
        // This method should always be called right before you build your notification,
        // since an Ongoing Activity doesn't hold references to the context.
        ongoingActivity.apply(applicationContext)
```

- Status: 用来包含将显示于各 Wear OS 界面上的文本;
- setAnimatedIcon: 设置一个动画图标（针对处于活动模式的表盘（[VectorDrawable](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable)）；
- setStaticIcon: 一个静态图标（针对处于微光模式的表盘）；
- setTouchIntent: 设备触摸事件，让用户可以通过点按从表盘或全局应用启动器的“Recents”部分返回到应用中；

> 注意：请务必为持续性活动设置静态图标和触摸 intent，[显式](https://developer.android.com/reference/androidx/wear/ongoing/OngoingActivity.Builder?hl=zh_cn#setStaticIcon(android.graphics.drawable.Icon))设置或设置为使用[通知](https://developer.android.com/reference/androidx/core/app/NotificationCompat.Builder?hl=zh_cn#setSmallIcon(int))的回调均可。如果不进行此设置，您会收到 IllegalArgumentException。

