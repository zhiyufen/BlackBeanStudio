## WLAN直连笔记

[TOC]

### 概述

[使用 WLAN 直连](https://www.wi-fi.org/discover-wi-fi/wi-fi-direct) (P2P) 技术，可以让具备相应硬件的 Android 4.0（API 级别 14）或更高版本设备在没有中间接入点的情况下，通过 WLAN 进行直接互联。使用这些 API，您可以实现支持 WLAN P2P 的设备间相互发现和连接，从而获得比蓝牙连接更远距离的高速连接通信效果。对于多人游戏或照片共享等需要在用户之间共享数据的应用而言，这一技术非常有用。

通过 [Wi-Fi 直连](https://www.wi-fi.org/discover-wi-fi/wi-fi-direct)（也称为对等连接或点对点连接），您的应用可以在超出蓝牙功能的范围内快速查找附近的设备并与之互动。

通过 Wi-Fi 对等连接（点对点连接）API，应用无需连接到网络或热点就可以连接到附近的设备。如果您的应用旨在成为安全、近距离网络的一部分，则 Wi-Fi 直连选项比传统 Wi-Fi 临时网络更合适，原因如下：

- Wi-Fi 直连支持 WPA2 加密。（一些临时网络仅支持 WEP 加密。）
- 设备可以广播其提供的服务，这有助于其他设备更容易地发现合适的对等设备。
- 在确定哪个设备应该是网络的群组所有者时，Wi-Fi 直连会检查各设备的电源管理、界面和服务功能，并使用该信息选择可最有效处理服务器职责的设备。
- Android 不支持 Wi-Fi 临时模式。

WLAN P2P API ：

- 支持您发现、请求，以及连接到对等设备的方法（在 `WifiP2pManager` 类中定义）；
- 支持您获知 `WifiP2pManager` 方法调用成功与否的侦听器。调用 `WifiP2pManager` 方法时，每个方法均可收到作为参数传入的特定侦听器；
- 通知您 WLAN P2P 框架检测到的特定事件（例如连接断开或新发现对等设备）的 Intent。

### 相关API

[`WifiP2pManager`](https://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager?hl=zh-cn) 类提供的方法使您可以在设备上与 WLAN 硬件交互，以执行发现和连接对等设备等操作。可执行的操作如下：

**表 1.** WLAN P2P 方法

| 方法                           | 说明                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| `initialize()`                 | 通过 WLAN 框架注册应用。必须先调用此方法，然后再调用任何其他 WLAN P2P 方法。 |
| `connect()`                    | 启动与具有指定配置的设备的对等连接。                         |
| `cancelConnect()`              | 取消任何正在进行的对等群组协商。                             |
| `requestConnectInfo()`         | 请求设备连接信息。                                           |
| `createGroup()`                | 以群组所有者的身份，使用当前设备创建对等群组。               |
| `removeGroup()`                | 移除当前对等群组。                                           |
| `requestGroupInfo()`           | 请求对等群组信息。                                           |
| `discoverPeers()`              | 启动对等设备发现                                             |
| `requestPeers()`               | 请求已发现对等设备的当前列表。                               |
| `startListening()`             | API33: 强制P2p进入`Listen`状态， 该设备将进入 其SocialCahnnel, 并响应其他Wi-Fi Direct对等方的探测请求。 |
| `addExternalApprover()`        | API33： 有连接请求时， 对连接请求进行过滤： 过滤好是自己需要的对等设备时，再反馈结果来请求连接：`setConnectionRequestResult()` |
| `setConnectionRequestResult()` | API33： 发送其由`addExternalApprover`过滤后，进行反馈其结果；结果有： `CONNECTION_REQUEST_ACCEPT(审批通过，可连接), CONNECTION_REQUEST_REJECT, CONNECTION_REQUEST_DEFER_TO_SERVICE, or CONNECTION_REQUEST_DEFER_SHOW_PIN_TO_SERVICE` |

`WifiP2pManager` 方法使您可以在侦听器中进行传递，以便 WLAN P2P 框架可以向您的 Activity 通知通话状态。下表介绍可用的侦听器接口和使用侦听器的相应 `WifiP2pManager` 方法调用：

**表 2.** WLAN P2P 侦听器

| 侦听器接口                              | 相关操作                                                     |
| :-------------------------------------- | :----------------------------------------------------------- |
| `WifiP2pManager.ActionListener`         | `connect()`、`cancelConnect()`、`createGroup()`、`removeGroup()` 和 `discoverPeers()` |
| `WifiP2pManager.ChannelListener`        | `initialize()`                                               |
| `WifiP2pManager.ConnectionInfoListener` | `requestConnectInfo()`                                       |
| `WifiP2pManager.GroupInfoListener`      | `requestGroupInfo()`                                         |
| `WifiP2pManager.PeerListListener`       | `requestPeers()`                                             |

WLAN P2P API 定义当发生特定 WLAN P2P 事件时会广播的 Intent，例如发现新的对等设备时，或设备的 WLAN 状态更改时。您可以通过创建处理这些 Intent 的[广播接收器](https://developer.android.com/guide/topics/connectivity/wifip2p?hl=zh-cn#creating-br)，在应用中注册接收这些 Intent：

**表 3.** WLAN P2P 广播Intent

| Intent                                | 说明                                                         |
| :------------------------------------ | :----------------------------------------------------------- |
| `WIFI_P2P_CONNECTION_CHANGED_ACTION`  | 当设备的 WLAN 连接状态更改时广播：指示 Wi-Fi 点对点连接的状态已更改； |
| `WIFI_P2P_PEERS_CHANGED_ACTION`       | 指示可用的对等设备列表已更改。当您调用 `discoverPeers()` 时广播。如果您在应用中处理此 Intent，则通常需要调用 `requestPeers()` 以获取对等设备的更新列表。 |
| `WIFI_P2P_STATE_CHANGED_ACTION`       | 当 WLAN P2P 在设备上启用或停用时广播。从 Android 10 开始，这不是固定的。如果您的应用依赖于在注册时接收这些广播（因为其之前一直是固定的），请在初始化时使用适当的 `get` 方法获取信息。 |
| `WIFI_P2P_THIS_DEVICE_CHANGED_ACTION` | 当设备的详细信息（例如设备名称）更改时广播。从 Android 10 开始，这不是固定的。如果您的应用依赖于在注册时接收这些广播（因为其之前一直是固定的），请在初始化时使用适当的 `get` 方法获取信息。 |

### WLAN P2P Intent 的广播接收器

广播接收器允许您通过 Android 系统接收 Intent 广播，以便您的应用对您感兴趣的事件作出响应。创建广播接收器以处理 WLAN P2P Intent 的基本步骤如下：

1. 创建扩展 `BroadcastReceiver` 类的类。对于类的构造函数；
2. 在广播接收器中，查看您感兴趣的 Intent `onReceive()`。根据接收到的 Intent，执行任何必要操作。

广播接收器以 `WifiP2pManager` 对象和 Activity 作为参数，并在接收到 Intent 时，使用这两个类恰当地执行所需操作：

```kotlin
/**
 * A BroadcastReceiver that notifies of important Wi-Fi p2p events.
 */
class WiFiDirectBroadcastReceiver(
        private val manager: WifiP2pManager,
        private val channel: WifiP2pManager.Channel,
        private val activity: MyWifiActivity
) : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val action: String = intent.action
        when (action) {
            WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION -> {
                // Check to see if Wi-Fi is enabled and notify appropriate activity
                // Determine if Wifi P2P mode is enabled or not, alert
                // the Activity.
                val state = intent.getIntExtra(WifiP2pManager.EXTRA_WIFI_STATE, -1)
                activity.isWifiP2pEnabled = state == WifiP2pManager.WIFI_P2P_STATE_ENABLED
            }
            WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION -> {
                // The peer list has changed! We should probably do something about that.
                // Call WifiP2pManager.requestPeers() to get a list of current peers
            }
            WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION -> {
                 // Connection state changed! We should probably do something about that.
                // Respond to new connection or disconnections
            }
            WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION -> {
                // Respond to this device's wifi state changing
            }
        }
    }
}
```

通过 Android Q，以下广播 Intent 已从粘性变为非粘性：

- [WIFI_P2P_CONNECTION_CHANGED_ACTION](https://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager?hl=zh-cn#WIFI_P2P_CONNECTION_CHANGED_ACTION)

  应用可使用 `requestConnectionInfo()`、`requestNetworkInfo()` 或 `requestGroupInfo()` 来检索当前连接信息。

- [WIFI_P2P_THIS_DEVICE_CHANGED_ACTION](https://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager?hl=zh-cn#WIFI_P2P_THIS_DEVICE_CHANGED_ACTION)

  应用可使用 `requestDeviceInfo()` 来检索当前连接信息。

### 基础开发

在使用 WLAN P2P API 之前，您必须确保您的应用可以访问硬件，并且设备支持 WLAN P2P API 协议。如果设备支持 WLAN P2P，您可以获得 `WifiP2pManager` 的实例，创建并注册广播接收器，然后开始使用 WLAN P2P API。

#### 声明权限

```xml
<uses-sdk android:minSdkVersion="14" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

除上述权限以外，您还需要启用位置信息模式才能使用下列 API：

- discoverPeers
- discoverServices
- requestPeers

#### WLAN P2P是否开启并支持

检查 WLAN P2P 是否开启并受支持。您可以在广播接收器收到 [`WIFI_P2P_STATE_CHANGED_ACTION`](https://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager?hl=zh-cn#WIFI_P2P_STATE_CHANGED_ACTION) Intent 时，在接收器中检查此项。向您的 Activity 通知 WLAN P2P 的状态，并作出相应回应(提示用户或者开启WiFi)；

```kotlin
override fun onReceive(context: Context, intent: Intent) {
...
val action: String = intent.action
when (action) {
    WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION -> {
        val state = intent.getIntExtra(WifiP2pManager.EXTRA_WIFI_STATE, -1)
        when (state) {
            WifiP2pManager.WIFI_P2P_STATE_ENABLED -> {
                // Wifi P2P is enabled
            }
            else -> {
                // Wi-Fi P2P is not enabled
            }
        }
    }
}
...
}
```

#### 注册您的应用连接到 WLAN P2P 框架

在 Activity 的 onCreate() 方法中，获取 WifiP2pManager 的实例，并通过调用 initialize()，在 WLAN P2P 框架中注册您的应用。此方法会返回 WifiP2pManager.Channel，用于将您的应用连接到 WLAN P2P 框架。此外，您还应该通过 WifiP2pManager 和 WifiP2pManager.Channel 对象以及对 Activity 的引用，创建广播接收器实例。这样广播接收器便可通知 Activity 感兴趣的事件并进行相应更新。此外，您还可以操纵设备的 WLAN 状态（如有必要）：

```kotlin
val manager: WifiP2pManager? by lazy(LazyThreadSafetyMode.NONE) {
    getSystemService(Context.WIFI_P2P_SERVICE) as WifiP2pManager?
}

var mChannel: WifiP2pManager.Channel? = null
var receiver: BroadcastReceiver? = null

override fun onCreate(savedInstanceState: Bundle?) {
    ...
	// 注册应用并返回Channel对象；
    mChannel = manager?.initialize(this, mainLooper, null)
    mChannel?.also { channel ->
        //创建广播接收器
        receiver = WiFiDirectBroadcastReceiver(manager, channel, this)
    }

}
```

#### 注册广播接收器及其过滤器

创建 Intent 过滤器，然后添加与广播接收器检查内容相同的 Intent：

```kotlin
val intentFilter = IntentFilter().apply {
    addAction(WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION)
    addAction(WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION)
    addAction(WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION)
    addAction(WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION)
}
```

在 Activity 的 `onResume()` 方法中注册广播接收器，然后在 Activity 的`onPause()` 方法中取消注册该接收器：

```kotlin
/* register the broadcast receiver with the intent values to be matched */
override fun onResume() {
    super.onResume()
    mReceiver?.also { receiver ->
        registerReceiver(receiver, intentFilter)
    }
}

/* unregister the broadcast receiver */
override fun onPause() {
    super.onPause()
    mReceiver?.also { receiver ->
        unregisterReceiver(receiver)
    }
}
```

获取 `WifiP2pManager.Channel` 并设置广播接收器后，应用便可调用 WLAN P2P 方法并收到 WLAN P2P Intent。

### 执行常见操作

#### 发现对等设备

如要发现可连接的对等设备，请调用 `discoverPeers()`，以检测范围内的可用对等设备。对此功能的调用为异步操作，如果您已创建 `WifiP2pManager.ActionListener`，则系统会通过 `onSuccess()` 和 `onFailure()` 告知应用成功与否。`onSuccess()` 方法仅会通知您发现进程已成功，但不会提供有关其发现的实际对等设备（如有）的任何信息：

```kotlin
manager?.discoverPeers(channel, object : WifiP2pManager.ActionListener {

    override fun onSuccess() {
        ...
    }

    override fun onFailure(reasonCode: Int) {
        ...
    }
})
```

所以上面一般只是用于打log或失败时，进行相关处理（比如是否GPS没打开之类的）；

**注：** 这仅启动对等设备发现。`discoverPeers()` 方法启动发现过程，然后立即返回。系统通过在提供的操作监听器中调用方法通知您是否成功启动对等设备发现过程。此外，在启动某个连接或形成点对点连接群组之前，发现一直处于活跃状态。

如果发现进程成功并检测到对等设备，则系统会广播 `WIFI_P2P_PEERS_CHANGED_ACTION` Intent，您可以在广播接收器中侦听该 Intent，以获取对等设备列表。当应用接收到 `WIFI_P2P_PEERS_CHANGED_ACTION` Intent 时，您可以通过 `requestPeers()` 请求已发现对等设备的列表。以下代码展示如何完成此项设置：

```kotlin
override fun onReceive(context: Context, intent: Intent) {
    val action: String = intent.action
    when (action) {
        ...
        WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION -> {
            // request available peers from the wifi p2p manager. This is an
            // asynchronous call and the calling activity is notified with a
            // callback on PeerListListener.onPeersAvailable()
            manager?.requestPeers(channel) { peers: WifiP2pDeviceList? ->
                // Handle peers list
            }
        }
        ...
    }
}
```

`requestPeers()` 方法也为异步操作，并可在对等设备列表可用时通过 `onPeersAvailable()`（定义见 `WifiP2pManager.PeerListListener` 接口）通知您的 Activity。`onPeersAvailable()` 方法为您提供 `WifiP2pDeviceList`，提供有关 Wi-Fi 点对点连接检测到的对等设备的信息。通过这些信息，您的应用还可以确定对等设备何时加入或离开网络。

#### 连接到对等设备

获取可能对等设备的列表，且已确定您要连接的设备(知道其)后，调用`connect(device.deviceAddress)` 方法即可连接到相应设备。调用此方法需要使用 `WifiP2pConfig` 对象，其中包含要连接的设备的信息。您可以通过 `WifiP2pManager.ActionListener` 获知连接是否成功。以下代码展示如何创建与所需设备的连接：

```kotlin
val device: WifiP2pDevice = ...
val config = WifiP2pConfig()
config.deviceAddress = device.deviceAddress
mChannel?.also { channel ->
    manager?.connect(channel, config, object : WifiP2pManager.ActionListener {

        override fun onSuccess() {
            //success logic
        }

        override fun onFailure(reason: Int) {
            //failure logic
        }
	}
})
```

如果群组中的每个设备都支持 Wi-Fi 直连，则在连接时无需明确提示输入群组的密码。但如需允许不支持 Wi-Fi 直连的设备加入群组，则需要通过调用 `requestGroupInfo()` 检索此密码，如以下代码段所示：

```kotlin
manager.requestGroupInfo(channel) { group ->
        val groupPassword = group.passphrase
    }
```

**注：** 在 `connect()` 方法中实现的 `WifiP2pManager.ActionListener` 仅在启动成功或失败时通知您。如需监听连接状态的更改，请实现 `WifiP2pManager.ConnectionInfoListener` 接口。其 `onConnectionInfoAvailable()` 回调在连接状态更改时通知您。如果多个设备要连接到单个设备（就像有三个或更多玩家的游戏，或聊天应用），应将一个设备指定为“群组所有者”。您可以按照[创建群组](https://developer.android.com/training/connect-devices-wirelessly/wifi-direct?hl=zh-cn#create-group)部分中的步骤，将特定设备指定为网络的群组所有者。

```kotlin
    private val connectionListener = WifiP2pManager.ConnectionInfoListener { info ->

        // InetAddress from WifiP2pInfo struct.
        val groupOwnerAddress: String = info.groupOwnerAddress.hostAddress

        // After the group negotiation, we can determine the group owner
        // (server).
        if (info.groupFormed && info.isGroupOwner) {
            // Do whatever tasks are specific to the group owner.
            // One common case is creating a group owner thread and accepting
            // incoming connections.
        } else if (info.groupFormed) {
            // The other device acts as the peer (client). In this case,
            // you'll want to create a peer thread that connects
            // to the group owner.
        }
    }
```

现在返回广播接收器的 `onReceive()`方法，并修改监听 `WIFI_P2P_CONNECTION_CHANGED_ACTION` intent 的部分。收到此 intent 时，调用 `requestConnectionInfo()`。这是一个异步调用，因此，由您作为参数提供的连接信息监听器接收结果。

```kotlin
    when (intent.action) {
        ...
        WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION -> {

            // Connection state changed! We should probably do something about
            // that.

            mManager?.let { manager ->

                val networkInfo: NetworkInfo? = intent
                        .getParcelableExtra(WifiP2pManager.EXTRA_NETWORK_INFO) as NetworkInfo

                if (networkInfo?.isConnected == true) {

                    // We are connected with the other device, request connection
                    // info to find group owner IP

                    manager.requestConnectionInfo(channel, connectionListener)
                }
            }
        }
        ...
    }
```

#### 创建群组

如果您希望运行应用的设备充当包括传统设备（即不支持 Wi-Fi 直连的设备）的网络的群组所有者，则请遵循[连接到对等设备](https://developer.android.com/training/connect-devices-wirelessly/wifi-direct?hl=zh-cn#connect)部分中的相同步骤序列，除非您使用 `createGroup()` 而不是 `connect()` 创建新的 `WifiP2pManager.ActionListener`。`WifiP2pManager.ActionListener` 中的回调处理方式相同，如以下代码段所示：

```kotlin
    manager.createGroup(channel, object : WifiP2pManager.ActionListener {
        override fun onSuccess() {
            // Device is ready to accept incoming connections from peers.
        }

        override fun onFailure(reason: Int) {
            Toast.makeText(
                    this@WiFiDirectActivity,
                    "P2P group creation failed. Retry.",
                    Toast.LENGTH_SHORT
            ).show()
        }
    })
```

> **注**：如果网络中的所有设备都支持 Wi-Fi 直连，则可在每个设备上使用 `connect()` 方法，因为该方法随后将自动创建群组并选择群组所有者。

创建群组后，可以调用 `requestGroupInfo()` 检索有关网络上对等设备的详细信息，包括设备名称和连接状态。

#### 传输数据

建立连接后，您可以通过套接字在设备之间传输数据。数据传输的基本步骤如下：

1. 创建 [`ServerSocket`](https://developer.android.com/reference/java/net/ServerSocket?hl=zh-cn)。此套接字会在指定端口等待来自客户端的连接，然后加以屏蔽直到连接发生，因此请在后台线程中执行此操作。
2. 创建客户端 [`Socket`](https://developer.android.com/reference/java/net/Socket?hl=zh-cn)。客户端使用 IP 地址和服务器套接字端口连接到服务器设备。
3. 将数据从客户端发送到服务器。客户端套接字成功连接到服务器套接字后，您可以通过字节流将数据从客户端发送到服务器。
4. 服务器套接字等待客户端连接（通过 [`accept()`](https://developer.android.com/reference/java/net/ServerSocket?hl=zh-cn#accept()) 方法）。在客户端连接前，此调用会屏蔽连接，所以这是另一个线程。发生连接时，服务器设备可接收到客户端数据。对这些数据执行任何操作，例如将其保存到文件中，或向用户显示这些数据。

以下示例（修改自 [WLAN P2P 演示](https://android.googlesource.com/platform/development/+/master/samples/WiFiDirectDemo/)示例）展示如何创建此客户端-服务器套接字通信，以及如何通过服务将 JPEG 图像从客户端传输到服务器。

服务端代码：

```kotlin
class FileServerAsyncTask(
        private val context: Context,
        private var statusText: TextView
) : AsyncTask<Void, Void, String?>() {

    override fun doInBackground(vararg params: Void): String? {
        /**
         * Create a server socket.
         */
        val serverSocket = ServerSocket(8888)
        return serverSocket.use {
            /**
             * Wait for client connections. This call blocks until a
             * connection is accepted from a client.
             */
            val client = serverSocket.accept()
            /**
             * If this code is reached, a client has connected and transferred data
             * Save the input stream from the client as a JPEG file
             */
            val f = File(Environment.getExternalStorageDirectory().absolutePath +
                    "/${context.packageName}/wifip2pshared-${System.currentTimeMillis()}.jpg")
            val dirs = File(f.parent)

            dirs.takeIf { it.doesNotExist() }?.apply {
                mkdirs()
            }
            f.createNewFile()
            val inputstream = client.getInputStream()
            copyFile(inputstream, FileOutputStream(f))
            serverSocket.close()
            f.absolutePath
        }
    }

    private fun File.doesNotExist(): Boolean = !exists()

    /**
     * Start activity that can handle the JPEG image
     */
    override fun onPostExecute(result: String?) {
        result?.run {
            statusText.text = "File copied - $result"
            val intent = Intent(android.content.Intent.ACTION_VIEW).apply {
                setDataAndType(Uri.parse("file://$result"), "image/*")
            }
            context.startActivity(intent)
        }
    }
}
```

在客户端上，通过客户端套接字连接到服务器套接字，然后传输数据。本示例传输的是客户端设备文件系统中的 JPEG 文件。

```kotlin
val context = applicationContext
val host: String
val port: Int
val len: Int
val socket = Socket()
val buf = ByteArray(1024)
...
try {
    /**
     * Create a client socket with the host,
     * port, and timeout information.
     */
    socket.bind(null)
    socket.connect((InetSocketAddress(host, port)), 500)

    /**
     * Create a byte stream from a JPEG file and pipe it to the output stream
     * of the socket. This data is retrieved by the server device.
     */
    val outputStream = socket.getOutputStream()
    val cr = context.contentResolver
    val inputStream: InputStream = cr.openInputStream(Uri.parse("path/to/picture.jpg"))
    while (inputStream.read(buf).also { len = it } != -1) {
        outputStream.write(buf, 0, len)
    }
    outputStream.close()
    inputStream.close()
} catch (e: FileNotFoundException) {
    //catch logic
} catch (e: IOException) {
    //catch logic
} finally {
    /**
     * Clean up any open sockets when done
     * transferring or if an exception occurred.
     */
    socket.takeIf { it.isConnected }?.apply {
        close()
    }
}
```

