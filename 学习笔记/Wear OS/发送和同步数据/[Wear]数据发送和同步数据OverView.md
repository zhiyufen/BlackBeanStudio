# 在Wear上发送和同步数据OverView

[TOC]

在Wear和手机之间的数据同步由 Wearable Data Layer API 完成， 为应用提供了一个可选的信道。尽管 Wear 应用可以使用 Wearable Data Layer API 与手机应用通信，但不建议使用此 API [连接到网络](https://developer.android.com/training/wearables/data-layer/network-access)。



## 1.0 支持数据类型及对应API

Data Layer API 由系统可以发送和同步的一组数据对象以及用于向应用通知某些事件的监听器组成：

- **数据项**：
  [`DataItem`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem.html) 提供在手持式设备和穿戴式设备之间自动同步的数据存储。负载的大小上限为 **100KB**
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



## 2.0 Wearable Data Layer

如需调用 Data Layer API，请使用 `Wearable` 类获取各种客户端类的实例;

> When a Wear app has an accompanying mobile app, you must use the same key to [sign](https://developer.android.com/studio/publish/app-signing) each of the two apps for them to communicate using the [Data Layer APIs](https://developer.android.com/training/wearables/data-layer).

客户端类实例： 

- DataClient： 一般用于发送DataItem；
- MessageClient:  用于发送 Message数据
- CapabilityClient: 用于检测哪些设备上有能够胜任的节点，确保其有处理某消息的能力；
- NodeClient： 用于当前设备的节点和连接设备的节点信息；
- ChannelClient： 用于操作Channel来发送大型数据；

### 2.1 获取Client:

```kotlin
val dataClient: DataClient = Wearable.getDataClient(context)
val messageClient: MessageClient = Wearable.getMessageClient(this)
```

Wearable API 客户端（如 `DataClient` 和 `MessageClient`）的创建成本很低，并且不需要仅创建一次并一直保留, 需要使用再从Wearable里取相关的客户端也不迟；

默认情况下，监听器的回调在应用的主界面线程上进行。 如需在其他线程上进行回调，请使用 `WearableOptions` 对象指定自定义 `Looper`

```kotlin
Wearable.WearableOptions.Builder().setLooper(myLooper).build().let { options ->
                Wearable.getDataClient(context, options)
            }
```

## 3.0 多设备同步

Wear OS 支持多部穿戴式设备连接到一部手持式设备。例如，如果用户在手持式设备上保存了一条记事，它会自动显示在用户的所有 Wear 设备上。为了在设备之间同步数据，Google 服务器在设备网络中托管了一个云节点。系统会将数据同步到直接连接的设备、云节点，以及通过 WLAN 连接到云节点的穿戴式设备。

![](https://developer.android.com/wear/images/wear_cloud_node.png)