# 发送和接收消息

[TOC]

## 1 消息的概念

与数据项的情况不同，手持式设备应用和穿戴式设备应用之间不会发生同步。

 消息是一种单向通信机制，适用于远程过程调用 (RPC)，如将消息发送到穿戴式设备以启动 Activity。

可将多部穿戴式设备连接到用户的手持式设备。网络中连接的每部设备被视为一个节点。如果连接了多部设备，您必须考虑哪些节点接收消息。例如，在穿戴式设备上接收语音数据的语音转录应用中，您应将消息发送到具有处理请求的处理能力和电池容量的节点，如手持式设备。

*注： 对于低于 7.3.0 的 Google Play 服务版本，一次只能将一部穿戴式设备连接到手持式设备。您可能需要更新现有代码，以将多个连接的节点功能考虑在内。如果您不实现这些变更，您的消息可能无法传送到目标设备。*

## 2 发送消息

Wear设备发送消息需要确保发送到有处理该消息能力的设备；

### 2.1 公布能力

要从穿戴式设备启动手持式设备上的 Activity，可使用 `MessageClient` 类发送请求。由于可将多部穿戴式设备连接到手持式设备，因此穿戴式设备应用需要确定连接的节点是否能够启动该 Activity。在手持式设备应用中，公布运行它的节点可提供特定功能。

如需公布手持式设备应用的功能，请执行以下操作：

1. 在项目的 `res/values/` 目录中创建一个 XML 配置文件，并将其命名为 `wear.xml`。
2. 将一个名为 `android_wear_capabilities` 的资源添加到 `wear.xml`。
3. 定义设备提供的功能。

***注意**：功能是您定义的自定义字符串，且在您的应用中必须是唯一的。*

```xml
<resources>
    <string-array name="android_wear_capabilities" translatable="false">
        <!-- declaring the provided capabilities -->
        <item>capability_1</item>
        <item>capability_2</item>
    </string-array>
</resources>
```

### 2.2 检索具体所需功能的节点

您可以通过调用 `CapabilityClient` 类的 `getCapability` 方法检测能够胜任的节点。

```kotlin
// capabilityInfo has the reachable nodes with the transcription capability
CapabilityInfo capabilityInfo = Tasks.await(Wearable.getCapabilityClient(context).getCapability(
      "capability_1", CapabilityClient.FILTER_REACHABLE));
```

注: 调用时，需要手持设备和Wear 设备需要使用同一签名，否则返回空的节点；

如需在节点连接到穿戴式设备时检测能够胜任的节点，请注册监听器的实例，具体来说就是 `CapabilityClient` 对象的 `OnCapabilityChangedListener`

```kotlin
// This example uses a Java 8 Lambda. Named or anonymous classes can also be used.
        CapabilityClient.OnCapabilityChangedListener capabilityListener =
            capabilityInfo -> { updateCapability1(capabilityInfo); };
        Wearable.getCapabilityClient(context).addListener(
            capabilityListener,
            "capability_1");
```

检测到能够胜任的节点后，确定要将消息发送到何处。您应选取一个靠近穿戴式设备的节点，以最大限度地减少消息经过多个节点的情况。附近的节点是指直接连接到设备的节点。要确定节点是否位于附近，可调用 [`Node.isNearby()`](https://developer.android.com/reference/com/google/android/gms/wearable/Node#isNearby()) 方法。

```kotlin
    private var transcriptionNodeId: String? = null

    private fun updateTranscriptionCapability(capabilityInfo: CapabilityInfo) {
        transcriptionNodeId = pickBestNodeId(capabilityInfo.nodes)
    }

    private fun pickBestNodeId(nodes: Set<Node>): String? {
        // Find a nearby node or pick one arbitrarily
        return nodes.firstOrNull { it.isNearby }?.id ?: nodes.firstOrNull()?.id
    }
```

### 2.3 传递消息

确定要使用的最佳节点后，请使用 `MessageClient` 类发送消息。在尝试发送消息之前，请验证该节点是否可用。此调用是同步的并且会阻止处理，直到系统将消息排队以进行传递。

*注意：成功的结果代码不能保证消息的传递。如果应用需要数据可靠性，请考虑使用 `DataItem` 对象或 `ChannelClient` 类在设备间发送数据。*

```kotlin
private fun requestTranscription(voiceData: ByteArray) {
        transcriptionNodeId?.also { nodeId ->
            val sendTask: Task<*> = Wearable.getMessageClient(context).sendMessage(
                    nodeId,
                    "capability_1",
                    voiceData
            ).apply {
                addOnSuccessListener { ... }
                addOnFailureListener { ... }
            }
        }
    }
```

您还可以向所有连接的节点广播消息。要检索您可以向其发送消息的所有连接的节点，

```kotlin
    private fun getNodes(): Collection<String> {
        return Tasks.await(Wearable.getNodeClient(context).connectedNodes).map { it.id }
    }
```

## 3 接收信息

要收到已接收消息的通知，实现 `MessageClient.OnMessageReceivedListener` 接口以提供消息事件的监听器。然后，使用 `addListener` 方法注册该监听器。

```kotlin
fun onMessageReceived(messageEvent: MessageEvent) {
        if (messageEvent.path == VOICE_TRANSCRIPTION_MESSAGE_PATH) {
            val startIntent = Intent(this, MainActivity::class.java).apply {
                addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
                putExtra("VOICE_DATA", messageEvent.data)
            }
            startActivity(this, startIntent)
        }
    }
```

