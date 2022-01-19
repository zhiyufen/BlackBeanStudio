# Wear OS能力调研

[TOC]

![](https://developer.android.com/wear/images/principles_2.png)

## 1.0 应用

### 1.1  通知

https://developer.android.com/training/wearables/notifications/creating

Wear API Level 30可能支持OnGoing Notification.

#### 1.1 .1 创建通知

- 通常，所有通知均从手机桥接至手表，但也可设置让其不桥接通知；
- 可构建展开式通知，并添加通知操作(Button); 
  ![](https://developer.android.com/wear/images/expanded_diagram.png)
- 通知内可添加内嵌操作，在 Wear 设备上，内嵌操作以附加按钮的形式显示在通知底部。
  ![](https://developer.android.com/wear/images/inline.png)
- 可指定仅限穿戴式设备的操作； 
  - 语音操作是穿戴式设备体验的重要组成部分。要创建支持语音输入的操作；
  - 除允许语音输入之外，您还可以提供多达五条文本回复，以供用户快速回复。
  - 可接收字符串形式的语音输入；
- 区别：从独立手表应用创建通知与创建桥接通知没有什么不同。来自独立 Wear 应用的通知与桥接通知看起来相似，但它们的行为略有差异。如果未设置 [`contentIntent`](https://developer.android.com/reference/androidx/core/app/NotificationCompat.Builder#setContentIntent(android.app.PendingIntent)) 或者通知从配对手机桥接而来，则点按通知会打开[展开式通知](https://developer.android.com/training/wearables/notifications/creating#expanded)。 然而，如果通知来自独立手表应用，点按通知会触发 `contentIntent` 以打开您的 Wear 应用。如需了解如何从独立应用创建通知并模拟展开式通知的行为

#### 1.1 .2 通知样式

最常见的通知样式包括：

- BIG_TEXT_STYLE
- BIG_PICTURE_STYLE
- INBOX_STYLE
- MESSAGING_STYLE

从目前的样式支持来看，Wear似乎并不支持自定义的布局

#### 1.1 .3 通知桥接模式

默认情况下，通知会从配套手机上的应用[桥接（共享）](https://developer.android.com/training/wearables/notifications)到配对手表。如果您构建了独立手表应用，并且还有配套手机应用，这些应用可能会创建重复的通知。Wear OS by Google 谷歌包含处理这一重复通知问题的功能。

开发者可以通过以下一种或多种方式改变通知的行为：

- 在清单文件中指定桥接配置
- 在运行时指定桥接配置
- 设置关闭 ID 以便在各设备间同步通知关闭行为



### 1.2  独立应用

一般情况下，独立应用和 Wear 2.0 的最低目标 API 级别为 25。仅当您为 Wear 1.0 和 2.0 使用同一 APK（因而具有嵌入的 Wear 1.0 APK）时，最低 SDK 级别才可以为 23。

- 手表应用可以归入以下类别之一：
  - 完全独立于手机应用
  - 半独立（手机应用不是必需的，只提供可选功能）
  - 依赖于手机应用
- 共享代码和数据存储
- 在另一台设备上检测您的应用 
  - 指定用于检测应用的功能名;
  - 从手表进行应用检测并打开网址(应用商店) 
  - 检测配对手机类型的详细信息
- 与 iPhone 配对的手表的位置数据
- 应仅获取必要的数据；

### 1.3 创建自定义布局

#### 1.3.1 自定义通知

一般而言，您应在手机上创建通知，让它们自动同步到穿戴式设备。

您可以使用自定义布局显示 Activity。您只能在手表上创建和发出自定义通知，系统不会将这些通知同步到手机。

#### 1.3.2 创建布局

如需了解详情，请参阅[为 Wear 设备创建自定义界面](https://developer.android.com/training/wearables/ui)

#### 1.3.3 Wear 界面库 API 参考文档

参考文档详细介绍了如何使用每个界面微件。请浏览 [Wear API 参考文档](https://developer.android.com/reference/androidx/wear/widget/package-summary)，了解上述类。

### 1.4 让应用保持可见状态

在微光模式和交互模式下均运行的 Wear 应用称为始终开启的应用。

搭载 Android 5.1 或更高版本的 Wear OS 设备可以让应用保持在前台运行，同时节省电池电量。Wear OS 应用可以控制手表在低功耗（微光）模式下显示的内容。

让应用始终可见会影响电池续航时间，因此在为应用添加这项功能时，您应该仔细考虑这个影响。

### 1.5 Wear 中的身份验证

随着独立手表的出现，Wear OS 应用现在可以完全独立地在手表上运行，而无需配套应用的协助。这种新的能力也意味着，当 Wear OS 独立应用需要访问云端数据时，它们需要自行管理身份验证。Wear OS 支持多种身份验证方法，以便独立 Wear 应用能够获得用户身份验证凭据。

目前，Wear OS 支持以下身份验证方法：

- [Google 登录](https://developer.android.com/training/wearables/apps/auth-wear#Google-Sign-in)
- [OAuth 2.0 支持](https://developer.android.com/training/wearables/apps/auth-wear#OAuth)
- [通过数据层传递令牌](https://developer.android.com/training/wearables/apps/auth-wear#pass-tokens)
- [自定义代码身份验证](https://developer.android.com/training/wearables/apps/auth-wear#custom-authentication)

### 1.6 打包和分发Wear 应用

所有运行 Wear 2.0 的设备都使用 Android 7.1.1（API 级别 25）。如果您的应用仅支持运行 Wear 2.0 或更高版本的设备，其最低和目标 API 级别应为 25。如果您的应用同时支持 Wear 1.x 和 2.0，其最低和目标 API 级别可以为 23。所有 Wear 应用的目标 API 级别都必须为 23 或更高级别，因此需要获取[运行时权限](https://developer.android.com/training/permissions/requesting)。

如果 Wear 2.0 应用具有配套应用，您必须使用同一密钥为这两个应用[签名](https://developer.android.com/studio/publish/app-signing)。 此要求也适用于 [Wear 1.x 应用](https://developer.android.com/training/wearables/apps/packaging#wear-1x)（这类应用一律具有配套应用）。

#### 1.6.1 分发到 Wear 1.x 和 2.0 手表

过去，Wear 1.x 的标准分发模型是在手机应用内嵌入手表应用。现在，Wear OS 允许您以相同的方式为 Wear 1.0 和 2.0 分发 Wear 应用。当用户安装您的手机应用时，如果您在 Play 商店中提供了兼容的 Wear 应用，该应用将自动安装到 Wear 1.0 手表上。此功能允许您停止在手机应用的 APK 中嵌入 Wear 应用。您可以在同时面向 Wear 1.0 和 Wear 2.0 手表的 Play 商店中提供手表 APK 的独立版本。

### 1.7 打造中国版 Wear OS 应用

- 您应使用 [FusedLocationProvider](https://developers.android.google.cn/training/articles/wear-location-detection.html) 来检测用户在中国的位置
- 

### 1.8 请求权限 

https://developer.android.com/training/articles/wear-permissions

### 1.9 发送和同步数据

Data Layer API 由系统可以发送和同步的一组数据对象以及用于向应用通知某些事件的监听器组成：

- **数据项**：
  [`DataItem`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem.html) 提供在手持式设备和穿戴式设备之间自动同步的数据存储。
- **资源**：
  [`Asset`](https://developers.google.com/android/reference/com/google/android/gms/wearable/Asset.html) 对象用于发送二进制 blob，如图片。您将资源附加到数据项后，系统会自动为您处理资源传输，通过缓存大型资源来避免重复传输，从而节省蓝牙带宽。
- **消息**：
  `MessageClient` 可以发送消息，适用于远程过程调用 (RPC)，如从穿戴式设备控制手持式设备的媒体播放器，或者从手持式设备启动穿戴式设备上的 intent。消息也适用于单向请求或请求/响应通信模型。如果手持式设备和穿戴式设备已连接，系统会将消息加入队列以进行传递，并返回成功的结果代码。如果这些设备未连接，会返回错误。成功的结果代码并不能说明消息已成功传递，因为设备可能会在收到结果代码后断开连接。
- **信道**：您可以使用 `ChannelClient` 将大型实体（如音乐和电影文件）从手持式设备传输到穿戴式设备。使用 `ChannelClient` 进行数据传输有以下好处：
  - 当使用附加到 `DataItem` 对象的 `Asset` 对象时，在未提供自动同步的情况下，在两台或更多连接的设备之间传输大型数据文件。` ChannelClient` 比 `DataClient` 更节省磁盘空间，后者在与连接的设备同步之前，会在本地设备上创建资源的副本。
  - 可靠地发送因过大而无法使用 `MessageClient` 发送的文件。
  - 传输流式数据，如从网络服务器提取的音乐或来自麦克风的语音数据。
- **WearableListenerService**（适用于服务）：
  通过扩展 [`WearableListenerService`](https://developers.google.com/android/reference/com/google/android/gms/wearable/WearableListenerService)，您可以在服务中监听重要的数据层事件。系统管理 [`WearableListenerService`](https://developers.google.com/android/reference/com/google/android/gms/wearable/WearableListenerService) 的生命周期，在需要发送数据项或消息时绑定到该服务，而在不需要执行任何操作时取消绑定该服务。
- **OnDataChangedListener**（适用于前台 Activity）：
  通过在某个 Activity 中实现 [`OnDataChangedListener`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataClient.OnDataChangedListener)，您可以在此 Activity 处于前台时侦听重要的数据层事件。使用此监听器代替 [`WearableListenerService`](https://developers.google.com/android/reference/com/google/android/gms/wearable/WearableListenerService) 后，您可以仅在用户正在使用您的应用时监听相关更改。

#### 1.9.1  网络访问和同步

Wear OS 应用可以发出网络请求。当手表通过蓝牙连接到手机时，手表的网络流量通常由手机代理。但是，当手机不可用时，会使用 WLAN 和移动数据网络，具体取决于硬件。Wear 平台可处理网络之间的转换。

您可以使用 HTTP、TCP 和 UDP 等协议。不过，不能使用 [android.webkit](https://developer.android.com/reference/android/webkit/package-summary) API（包括 [CookieManager](https://developer.android.com/reference/android/webkit/CookieManager) 类）。您可以通过读取和写入请求和响应的标头来使用 Cookie。

此外，我们还建议您使用以下 API：

- [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler) API，适用于异步作业，包括定期轮询（如下所述）
- 多网络 API，在需要连接到特定网络类型时使用，请参阅[多个网络连接](https://developer.android.com/about/versions/android-5.0#Wireless)

#### 1.9.2 访问 Wearable Data Layer

Wear 应用可以使用 Data Layer API 与手机应用通信，但不建议使用此 API [连接到网络](https://developer.android.com/training/wearables/data-layer/network-access)。

如果您在 Activity 范围内使用该 API，请使用 `Wearable` 类的 `getDataClient(activity)` 方法让某些互动（例如，要求用户更新他们的 Google Play 服务版本）显示为对话框而非通知。

默认情况下，监听器的回调在应用的主界面线程上进行。 如需在其他线程上进行回调，请使用 `WearableOptions` 对象指定自定义 `Looper`（请参阅 `WearableOptions.Builder`）：

#### 1.9.3 同步数据项

[`DataItem`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem.html) 会定义系统在手持式设备与穿戴式设备之间同步数据时使用的接口：

- **负载** - 一个字节数组，大小上限为 100KB；
- **路径** - 必须以正斜杠开头的唯一字符串（例如，`"/path/to/data"`）

可设置优先级；

#### 1.9.4 传输资源

如需通过蓝牙传输发送图片等庞大的二进制数据 blob，您可以将 [`Asset`](https://developer.android.com/reference/com/google/android/gms/wearable/Asset) 附加到数据项，然后将数据项放入重复数据存储。

资源自动处理数据的缓存，以防止重新传输并节省蓝牙带宽。 一种常见模式是，手持式设备应用下载图片，将其缩小到适合在穿戴式设备上显示的尺寸，然后将其作为资源传输到穿戴式设备应用。

#### 1.9.5 发送和接收消息

您可以使用 `MessageClient` API 发送消息并将以下内容附加到消息上：

- 任意负载（可选）
- 唯一标识消息操作的路径

与数据项的情况不同，手持式设备应用和穿戴式设备应用之间不会发生同步。 消息是一种单向通信机制，适用于远程过程调用 (RPC)，如将消息发送到穿戴式设备以启动 Activity。

**注意**：成功的结果代码不能保证消息的传递。如果应用需要数据可靠性，请考虑使用 `DataItem` 对象或 `ChannelClient` 类在设备间发送数据。

#### 1.9.6 处理数据层事件

当您调用 Data Layer API 时，可以在调用完成时收到其状态。您还可以监听因您的应用在 Wear OS by Google 谷歌网络上任何位置做出的数据更改而引发的数据事件。

- 可同步/异步调用；
- 可监听数据层事件：
  创建一项扩展 `WearableListenerService` 的服务。
  创建实现 `DataClient.OnDataChangedListener` 接口的 Activity 或类

## 2.0 使用您的数据丰富表盘主题

数据提供程序应用会通过提供包含文本、字符串、图片和数字的字段，向表盘[复杂功能](https://developer.android.com/training/wearables/watch-faces/complications)提供信息。

数据提供程序服务扩展了 [`ComplicationProviderService`](https://developer.android.com/reference/android/support/wearable/complications/ComplicationProviderService)，可直接将有用的信息提供给表盘。

## 3.0 创建表盘主题

您可以使用动态数字画布创建表盘。Wear OS by Google 谷歌提供颜色、动画和上下文信息选项。

### 3.1 设计表盘

自定义表盘利用可包含颜色、动画及上下文信息的动态数字画布。

微光模式背景经常为全黑或者没有图片的灰色。

屏幕密度为 hdpi 的 Wear OS 设备的背景图片大小应为 320 x 320 像素，才能同时适合方形设备和圆形设备。背景图片的角在圆形设备上不可见。在您的代码中，您可以检测设备屏幕的尺寸，并在设备的分辨率低于图片的分辨率时缩小背景图片。为了提升性能，您应只缩放背景图片一次并存储生成的位图。

您应只在需要时运行应用代码来检索上下文数据，并存储结果，以便在每次绘制表盘主题时再利用这些数据。例如，您不需要每分钟都提取天气更新信息。

为了延长电池续航时间，用于在微光模式下绘制表盘主题的应用代码应相对简单。在此模式下，您通常使用一组有限的颜色来绘制形状的轮廓。在交互模式下，您可以使用全彩色、复杂的形状、渐变和动画来绘制您的表盘主题。

### 3.2 构建表盘服务

表盘是封装在 [Wear OS 应用](https://developer.android.com/training/wearables/apps)中的一种[服务](https://developer.android.com/guide/components/services)。当用户选择可用的表盘时，系统会显示该表盘并调用服务回调方法。

如果用户安装了包含表盘的 Wear OS 应用，可以通过手表上的表盘选择器选择要使用的表盘，或者也可以通过配对手机上的配套应用选择表盘。

#### 3.2.1 实现服务和回调方法

Wear OS 中的表盘以[服务](https://developer.android.com/guide/components/services)的形式实现。如果表盘处于活动状态，那么当时间发生变化或发生重要事件（如切换到微光模式或接收新通知）时，系统就会调用表盘服务中的方法。服务实现随后会使用更新后的时间和任何其他相关数据在屏幕上绘制表盘。

如需实现表盘，您需要扩展 [`CanvasWatchFaceService`](https://developer.android.com/reference/android/support/wearable/watchface/CanvasWatchFaceService) 和 [`CanvasWatchFaceService.Engine`](https://developer.android.com/reference/android/support/wearable/watchface/CanvasWatchFaceService.Engine) 类，然后替换 [`CanvasWatchFaceService.Engine`](https://developer.android.com/reference/android/support/wearable/watchface/CanvasWatchFaceService.Engine) 类中的回调方法。这些类包含在[穿戴式设备支持库](https://developer.android.com/reference/android/support/wearable/watchface/package-summary)中。

#### 3.2.2 注册表盘服务

实现表盘服务后，您需要在穿戴式设备应用的清单文件中注册实现。当用户安装此应用时，系统会使用该服务的相关信息将表盘显示在 [Wear OS 配套应用](https://play.google.com/store/apps/details?id=com.google.android.wearable.app&hl=zh-CN)和穿戴式设备上的表盘选择器中。

在向用户显示设备上安装的所有表盘时，[Wear OS 配套应用](https://play.google.com/store/apps/details?id=com.google.android.wearable.app&hl=zh-CN)和穿戴式设备上的表盘选择器会使用 `com.google.android.wearable.watchface.preview` 元数据条目定义的预览图片。

预览图片的大小一般为 320x320 像素。

在圆形设备上看起来截然不同的表盘可同时提供圆形和方形预览图片。要指定圆形预览图片，请使用 `com.google.android.wearable.watchface.preview_circular` 元数据条目。如果表盘同时包含两种预览图片，配套应用和穿戴式设备上的表盘选择器会根据手表的形状显示合适的图片。如果不包含圆形预览图片，则对方形和圆形设备都使用方形预览图片。对于圆形设备，则会将方形预览图片剪裁为圆形。

您的穿戴式设备应用可以包含多个表盘。您必须在穿戴式设备应用的清单文件中为每个表盘实现添加一个服务条目。

### 3.3 绘制表盘

配置项目并添加用来实现表盘服务的类后，您便可以开始编写代码以初始化并绘制自定义表盘。

当系统加载您的服务时，您应分配并初始化表盘所需的大多数资源，其中包括加载位图资源、创建定时器对象以运行自定义动画、配置绘图样式和执行其他计算。您通常可以仅执行这些操作一次并重复利用它们的结果。这种做法可提升表盘的性能，并使代码维护起来更容易。

### 3.4 表盘复杂功能

复杂功能是指表盘主题中显示的除时间以外的其他任何功能。例如，电量指示器就是一项复杂功能。Complications API 既适用于表盘主题，也适用于数据提供器应用。

![](https://developer.android.com/wear/images/complications-data-flow.png)

### 3.5 向复杂功能提供数据

表盘[复杂功能](https://developer.android.com/training/wearables/watch-faces/complications)可显示数据提供程序提供的数据。数据提供程序会向表盘提供包含文本、字符串、图片和数字的原始字段。

数据提供程序服务扩展了 [`ComplicationProviderService`](https://developer.android.com/reference/android/support/wearable/complications/ComplicationProviderService)，可直接在表盘上为用户提供实用信息。

### 3.6 向表盘添加复杂功能

表盘[复杂功能](https://developer.android.com/training/wearables/watch-faces/complications)可显示来自数据提供程序的数据。利用 Complications API，表盘可以选择要用于获取基础数据的数据提供程序。这样一来，表盘不但可以显示时刻，还可以显示其他信息，而且无需通过代码来获取数据。

利用 Complications API，用户也可以自行选择数据提供程序。此外，Wear OS by Google 谷歌提供了一个[用于选择数据源的界面](https://developer.android.com/training/wearables/watch-faces/adding-complications#allowing_users_to_choose_data_providers)。

表盘可以指定在用户选择提供程序之前使用的默认提供程序；

如果一个表盘与某个提供程序具有合作关系，并希望将其用作默认提供程序，则该表盘可以请求该提供程序将其列为安全的表盘。

### 3.7 创建互动式表盘

用户可以通过多种方式与您的表盘互动。例如，用户可以点按表盘，了解当前正在播放的歌曲或查看当天的日程。Wear OS by Google 谷歌允许表盘在表盘上的给定位置接受单次点按手势，只要没有其他界面元素也响应该手势即可。

### 3.8 提供配置 Activity

一些表盘支持配置参数，可让用户自定义表盘的外观和行为。例如，有些表盘允许用户选择自定义的背景颜色，而显示两个不同时区的时间的表盘，则可让用户选择自己感兴趣的时区。

支持配置参数的表盘主题可让用户使用穿戴式设备应用中的 Activity 和/或手持式设备应用中的 Activity 自定义表盘主题。用户可以在穿戴式设备上启动穿戴式设备配置 Activity，也可以从手持式设备应用（如果已安装）中启动配套设备配置 Activity。

## 4.0 Tile

类似Android平台的Widget， Tile可让用户只需要简单切换，就可以快速查阅相关信息；

用户从已提供里的Tile中可以添加/删除Tile；开发者可自定义Tile, 但需要Target API 为 26及以上；

![](https://developer.android.com/images/training/articles/example_tiles.png)

https://developer.android.com/training/articles/wear-tiles

## 5.0 Ongoing Activity API



![](https://developer.android.com/codelabs/ongoing-activity/img/cb2a383284f10acd.gif)

在表盘中，您会看到一个动画图标（针对计时器），如果用户点按该图标，系统会启动为计时器提供支持的应用。

注： 仅在 Wear OS的API级别 >= 30才支持

https://developer.android.com/training/wearables/ongoing-activity