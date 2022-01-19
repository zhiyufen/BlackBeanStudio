# 同步数据项(DataItem)

[TOC]

## 1.0 DataItem概念

[`DataItem`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem.html) 会定义系统在手持式设备与穿戴式设备之间同步数据时使用的接口。`DataItem` 通常由下面几部分组成：

- **负载** - 一个字节数组，可供您随意使用任何数据进行设置，从而以您自己的方式执行对象序列化和反序列化。负载的大小上限为 **100KB**。
- **路径** - 必须以正斜杠开头的唯一字符串（例如，`"/path/to/data"`），当你接收到数据时，可用其作为唯一Key；

## 2.0 创建并发送DataItem

并不会直接创建DataItem进行发送，而是创建一个PutDataRequest的对象，放置DATaItem数据;

创建步骤：

1. 创建一个PutDataMapRequest对象，设置数据项的路径。

   **注意*：路径字符串是数据项的唯一标识符，让您可从连接的任一端访问该数据项。路径必须以正斜杠开头。如果您在应用中使用层级式数据，应创建与数据结构匹配的路径架构。

2. 调用 [`PutDataMapRequest.getDataMap()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/PutDataMapRequest.html#getDataMap()) 获取一个可用来设置值的数据映射。

3. 使用 `put...()` 方法（如 [`putString()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataMap.html#putString(java.lang.String, java.lang.String))）为数据映射设置任何需要的值。

4. 如果延迟同步会对用户体验产生负面影响，请调用 [`setUrgent()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/PutDataRequest#setUrgent())。

5. 调用 [`PutDataMapRequest.asPutDataRequest()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/PutDataMapRequest.html#asPutDataRequest()) 获取一个 [`PutDataRequest`](https://developers.google.com/android/reference/com/google/android/gms/wearable/PutDataRequest.html) 对象。

6. 使用 [`DataClient`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataClient) 类的 `putDataItem` 方法请求系统创建数据项。

从官方文档说：如果手持式设备和穿戴式设备断开连接，数据会进行缓存并在重新建立连接后进行同步。

但目前测试似乎不是这样的？

```kotlin
// Create a data map and put data in it
val putDataReq: PutDataRequest = PutDataMapRequest.create("/count").run {
       dataMap.putInt("count", count++)
       asPutDataRequest()
    }
putDataReq.setUrgent()
val dataItemTask: Task<DataItem> = Wearable.getDataClient(getApplicationContext()).putDataItem(putDataReq)

try {
   // Block on a task and get the result synchronously (because this is on a background
   // thread).
    //阻塞当前线程，直到数据项发送成功；
    DataItem dataItem = Tasks.await(dataItemTask);
    LOGD(TAG, "DataItem saved: " + dataItem);
} catch (ExecutionException exception) {
    Log.e(TAG, "Task failed: " + exception);
} catch (InterruptedException exception) {
    Log.e(TAG, "Interrupt occurred: " + exception);
}
```

### 2.1 DataItem的优先级

DataClient API 让您可以紧急请求同步 DataItems，因为如果延迟同步 [`DataItems`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem.html) 会对用户体验产生负面影响，您可以将其标记为紧急;

通过调用 [`setUrgent()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/PutDataRequest#setUrgent()) 让系统立即同步您的 [`DataItems`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem)。

注： 如果您不调用 [`setUrgent()`](https://developers.google.com/android/reference/com/google/android/gms/wearable/PutDataRequest#setUrgent())，系统可能延迟长达 30 分钟后再同步非紧急 [`DataItems`](https://developers.google.com/android/reference/com/google/android/gms/wearable/DataItem)， 以便延长设备的电池续航时间；但通常来说，即便有延迟，也只有几分钟。现在默认紧急程度为非紧急； 

### 2.2 Tasks

```kotlin
try {
   // Block on a task and get the result synchronously (because this is on a background
   // thread).
    //阻塞当前线程，直到数据项发送成功；
    DataItem dataItem = Tasks.await(dataItemTask);
    LOGD(TAG, "DataItem saved: " + dataItem);
} catch (ExecutionException exception) {
    Log.e(TAG, "Task failed: " + exception);
} catch (InterruptedException exception) {
    Log.e(TAG, "Interrupt occurred: " + exception);
}
```

这里Tasks.await(dataItemTask)虽然已经返回结果了，但不意味一定会同步到Wear设备上；有可能只是进行缓存，等待下次连接时进行发送(此处待确认)；

Tasks似乎调用call进行异步监听其结果(Todo)

## 3.0 监听数据项事件

如果数据层连接的一端更改了数据项，您可能希望在连接的另一端出现任何更改时收到通知。您可以通过实现数据项事件监听器来实现这一目的。

实现步骤：

1. Activity实现DataClient.OnDataChangedListener的接口，数据项发生变化时会发送到该接口；
2. 在OnResume里注册数据项的监听，同时需要在onPause时移除掉监听接口；

```kotlin
class MainActivity : Activity(), DataClient.OnDataChangedListener {

    private var count = 0

    override fun onResume() {
        super.onResume()
        Wearable.getDataClient(this).addListener(this)
    }

    override fun onPause() {
        super.onPause()
        Wearable.getDataClient(this).removeListener(this)
    }

    override fun onDataChanged(dataEvents: DataEventBuffer) {
        dataEvents.forEach { event ->
            // DataItem changed
            if (event.type == DataEvent.TYPE_CHANGED) {
                event.dataItem.also { item ->
                    if (item.uri.path.compareTo("/count") == 0) {
                        DataMapItem.fromDataItem(item).dataMap.apply {
                            updateCount(getInt("count"))
                        }
                    }
                }
            } else if (event.type == DataEvent.TYPE_DELETED) {
                // DataItem deleted
            }
        }
    }

    // Our method to update the count
    private fun updateCount(int: Int) { ... }

    ...
}
```

收到数据时，我们需要通过DataMapItem来获取发送过来的真实数据；

### DataEvent的类型

- DataEvent.TYPE_CHANGED： 表示数据项有变化(新)；
- DataEvent.TYPE_DELETED： 表示数据项进行删除；