# 数据层事件

[TOC]

当您调用 Data Layer API 时，可以在调用完成时收到其状态。您还可以监听因您的应用在 Wear OS by Google 谷歌网络上任何位置做出的数据更改而引发的数据事件。

## 1 等待数据层调用的状态

对 Data Layer API 的调用（例如，使用 `DataClient` 类的 `putDataItem` 方法的调用）有时会返回 `Task` 对象。一旦创建了 `Task` 对象，操作便会在后台排队。如果您在此之后不再进行任何处理，则操作最终会以静默方式完成。不过，您通常需要在操作完成后对结果进行某种处理，因此 `Task` 对象允许您以同步或异步方式等待结果状态。

### 1.1 异步调用

如果您的代码在主界面线程上运行，请勿对 Data Layer API 进行阻塞调用。您可以通过向 `Task` 对象添加回调方法来异步运行调用，该方法在操作完成时触发：

```kotlin
    // Using Kotlin function references
    task.addOnSuccessListener(::handleDataItem)
    task.addOnFailureListener(::handleDataItemError)
    task.addOnCompleteListener(::handleTaskComplete)
    ...
    fun handleDataItem(dataItem: DataItem) { ... }
    fun handleDataItemError(exception: Exception) { ... }
    fun handleTaskComplete(task: Task<DataItem>) { ... }
```

### 1.2 同步调用

如果您的代码在后台服务中的单独处理程序线程上运行（在 `WearableListenerService` 中就是如此），则可以阻塞调用。在这种情况下，您可以对 `Task` 对象调用 [Tasks.await()](https://developers.google.com/android/reference/com/google/android/gms/tasks/Tasks.html#await(com.google.android.gms.tasks.Task,    long, java.util.concurrent.TimeUnit))，这样会使该调用阻塞，直到请求完成并返回 `Result` 对象：

```kotlin
    try {
        Tasks.await(dataItemTask).apply {
            Log.d(TAG, "Data item set: $uri")
        }
    }
    catch (e: ExecutionException) { ... }
    catch (e: InterruptedException) { ... }
```

## 2 监听数据层事件

由于数据层会在手持式设备和穿戴式设备之间同步和发送数据，因此通常有必要监听重要事件。举例来说，此类事件包括创建数据项和接收消息。

如需监听数据层事件，您有两个选择：

- 创建一项扩展 `WearableListenerService` 的服务。
- 创建实现 `DataClient.OnDataChangedListener` 接口的 Activity 或类

无论选择哪一个，您都需要替换想要处理的事件的数据事件回调方法。

*注意：关于电池用量，在应用的清单中注册了 `WearableListenerService`，如果应用尚未运行，该服务会将其启动。如果您只需要在应用已在运行时监听事件（交互式应用通常就是如此），那么请不要使用 `WearableListenerService`。*

### 2.1 使用  WearableListenerService

您通常会同时在穿戴式设备应用和手持式设备应用中创建此服务的实例。如果您对其中一个应用中的数据事件不感兴趣，则无需在该特定应用中实现此服务。 [`WearableListenerServiceAPI 参考文档`](https://developers.google.com/android/reference/com/google/android/gms/wearable/WearableListenerService)

使用 [`WearableListenerService`](https://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService) 监听的一些事件如下：

- [`onDataChanged()`](https://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService#onDataChanged(com.google.android.gms.wearable.DataEventBuffer))：每当创建、删除或更改数据项对象时，系统都会在所有连接的节点上触发此回调。
- [`onMessageReceived()`](https://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onMessageReceived(com.google.android.gms.wearable.MessageEvent))：从某个节点发送的消息会在目标节点上触发此回调。
- [`onCapabilityChanged()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/WearableListenerService.html#onCapabilityChanged(com.google.android.gms.wearable.CapabilityInfo))：当您的应用实例广播的某个功能发布到网络上时，该事件会触发此回调。如果您要寻找一个附近的节点，可以查询回调中提供的节点的 [`isNearby()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/Node.html#isNearby()) 方法。

除了此列表上的事件外，您还可以监听来自 `ChannelClient.ChannelCallback` 的事件，如 `onChannelOpened()`。

上述所有事件都在后台线程上执行，而不是在主线程上执行。该Service对象的生命周期由Wear OS控制；

#### 2.1.2 创建 WearableListenerService

1. 创建一个扩展 [`WearableListenerService`](https://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService) 的类。
2. 监听您感兴趣的事件，例如 [`onDataChanged()`](https://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService#onDataChanged(com.google.android.gms.wearable.DataEventBuffer))。
3. 在 Android 清单中声明一个 intent 过滤器，以向系统通知您的 [`WearableListenerService`](https://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService)。 此声明使系统能根据需要绑定您的服务。

```kotlin
public class DataLayerListenerService extends WearableListenerService {

    private static final String TAG = "DataLayerService";

    private static final String START_ACTIVITY_PATH = "/start-activity";
    private static final String DATA_ITEM_RECEIVED_PATH = "/data-item-received";
    public static final String COUNT_PATH = "/count";
    public static final String IMAGE_PATH = "/image";
    public static final String IMAGE_KEY = "photo";

    @Override
    public void onDataChanged(DataEventBuffer dataEvents) {
        Log.d(TAG, "onDataChanged: " + dataEvents);

        // Loop through the events and send a message back to the node that created the data item.
        for (DataEvent event : dataEvents) {
            Uri uri = event.getDataItem().getUri();
            String path = uri.getPath();

            if (COUNT_PATH.equals(path)) {
                // Get the node id of the node that created the data item from the host portion of
                // the uri.
                String nodeId = uri.getHost();
                // Set the data of the message to be the bytes of the Uri.
                byte[] payload = uri.toString().getBytes();

                // Send the rpc
                // Instantiates clients without member variables, as clients are inexpensive to
                // create. (They are cached and shared between GoogleApi instances.)
                Task<Integer> sendMessageTask =
                        Wearable.getMessageClient(this)
                                .sendMessage(nodeId, DATA_ITEM_RECEIVED_PATH, payload);

                sendMessageTask.addOnCompleteListener(
                        new OnCompleteListener<Integer>() {
                            @Override
                            public void onComplete(Task<Integer> task) {
                                if (task.isSuccessful()) {
                                    Log.d(TAG, "Message sent successfully");
                                } else {
                                    Log.d(TAG, "Message failed.");
                                }
                            }
                        });
            }
        }
    }

    @Override
    public void onMessageReceived(MessageEvent messageEvent) {
        Log.d(TAG, "onMessageReceived: " + messageEvent);

        // Check to see if the message is to start an activity
        if (messageEvent.getPath().equals(START_ACTIVITY_PATH)) {
            Intent startIntent = new Intent(this, MainActivity.class);
            startIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            startActivity(startIntent);
        }
    }
}
```

### 2.1.3 对 WearableListenerService 使用过滤器

添加过滤器，使得只有特定事件才会唤醒或启动您的应用。这一变化可提高系统效率，并减少耗电量以及与您的应用有关的其他开销。

Android 清单:

```xml
        <service android:name=".DataLayerListenerService" >
            <intent-filter>
                <!-- listeners receive events that match the action and data filters -->
                <action android:name="com.google.android.gms.wearable.DATA_CHANGED" />
                <data android:scheme="wear" android:host="*" android:pathPrefix="/count"/>
            </intent-filter>
            <intent-filter>
                <action android:name="com.google.android.gms.wearable.MESSAGE_RECEIVED" />
                <data android:scheme="wear" android:host="*" android:pathPrefix="/start-activity"/>
            </intent-filter>
            <!-- 
                <action android:name="com.google.android.gms.wearable.CAPABILITY_CHANGED" />
                <action android:name="com.google.android.gms.wearable.CHANNEL_EVENT" />
            -->
        </service>
```

需遵守标准 Android 过滤器匹配规则。您可以为每个清单指定多项服务，为每项服务指定多个 Intent 过滤器，为每个过滤器指定多项操作，并为每个过滤器指定多个数据 stanza。过滤器可以匹配通配符主机或特定主机。要匹配通配符主机，请使用 `host="*"`。要匹配特定主机，请指定 `host=`。

您还可以匹配字面量路径或路径前缀。如果要按路径或路径前缀进行匹配，您必须指定通配符或特定主机。 如果您不这样做，系统会忽略您指定的路径。

匹配 intent 过滤器时，需要牢记两条重要规则：

- 如果没有为 Intent 过滤器指定地址协议，系统会忽略其他所有 URI 属性。
- 如果没有为过滤器指定主机，系统会忽略所有路径属性。

#### 2.1.4 使用实时监听器

如果您的应用只关心用户与应用互动时的数据层事件，则处理每项数据更改可能不需要长时间运行的服务。在这种情况下，您可以通过实现以下一个或多个接口来监听 Activity 中的事件：

- `DataClient.OnDataChangedListener`
- `MessageClient.OnMessageReceivedListener`
- `CapabilityClient.OnCapabilityChangedListener`
- `ChannelClient.ChannelCallback`

如需创建用于监听数据事件的 Activity，请执行以下操作：

1. 实现所需接口。
2. 在 `onCreate()` 或 `onResume()` 方法中，调用 `Wearable.getDataClient(this).addListener()`、`MessageClient.addListener()`、`CapabilityClient.addListener()` 或 `ChannelClient.registerChannelCallback()`，以通知 Google Play 服务您的 Activity 想要监听数据层事件。
3. 在 `onStop()` 或 `onPause()` 中，使用 `DataClient.removeListener()`、`MessageClient.removeListener()`、`CapabilityClient.removeListener()` 或 `ChannelClient.unregisterChannelCallback()` 取消注册任何监听器。
4. 如果某个 Activity 仅对具有特定路径前缀的事件感兴趣，您可以添加具有合适前缀过滤器的监听器，以便仅接收与当前应用状态有关的数据。
5. 实现 `onDataChanged()`、`onMessageReceived()`、`onCapabilityChanged()` 或来自 `ChannelClient.ChannelCallback` 的方法，具体取决于您实现的接口。这些方法在主线程上被调用，或者您也可以使用 `WearableOptions` 指定自定义 `Looper`。