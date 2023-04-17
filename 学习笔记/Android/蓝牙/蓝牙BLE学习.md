## 蓝牙BLE学习

[TOC]

`Google`在`android 4.3（API Level 18）`的`android`版本中引入了低功耗蓝牙BLE核心API。低功耗蓝牙`BLE`也就是我们经常说的蓝牙4.0， 该技术拥有极低的运行和待机功耗，使用一粒纽扣电池甚至可连续工作数年之久。可方便应用发现设备、查询服务和传输信息。

### 相关概念

以下是对 BLE 关键术语和概念的总结：

- **通用属性配置文件 (GATT)**

  GATT 配置文件是一种通用规范，内容针对在 BLE 链路上发送和接收称为“属性”的简短数据片段。目前所有低功耗应用配置文件均以 GATT 为基础。

  - 蓝牙特别兴趣小组 (Bluetooth SIG) 为低功耗设备定义诸多[配置文件](https://www.bluetooth.org/en-us/specification/adopted-specifications)。配置文件是描述设备如何在特定应用中工作的规范。请注意，一台设备可以实现多个配置文件。例如，一台设备可能包含心率监测仪和电池电量检测器。

- **属性协议 (ATT)** — 属性协议 (ATT) 是 GATT 的构建基础，二者的关系也被称为 GATT/ATT。ATT 经过优化，可在 BLE 设备上运行。为此，该协议尽可能少地使用字节。每个属性均由通用唯一标识符 (UUID) 进行唯一标识，后者是用于对信息进行唯一标识的字符串 ID 的 128 位标准化格式。由 ATT 传输的*属性*采用*特征*和*服务*格式。

- **特征** — 特征包含一个值和 0 至多个描述特征值的描述符。您可将特征理解为类型，后者与类类似。 

- **描述符** — 描述符是描述特征值的已定义属性。例如，描述符可指定人类可读的描述、特征值的可接受范围或特定于特征值的度量单位。

- **Service** — 服务是一系列特征。例如，您可能拥有名为“心率监测器”的服务，其中包括“心率测量”等特征。您可以在 [bluetooth.org](https://www.bluetooth.org/en-us/specification/adopted-specifications) 上找到基于 GATT 的现有配置文件和服务的列表。

以下是 Android 设备与 BLE 设备交互时应用的角色和职责：

- 中央与外围。这适用于 BLE 连接本身。担任中央角色的设备进行扫描、寻找广播；外围设备发出广播。
- GATT 服务器与 GATT 客户端。这确定两个设备建立连接后如何相互通信。

要了解两者的区别，请想象您有一部 Android 手机和一个 Activity 追踪器，该 Activity 追踪器是一个 BLE 设备。手机支持中央角色；Activity 追踪器支持外围角色。手机与 Activity 追踪器建立连接后，它们便开始相互传送 GATT 数据。根据它们传送数据的种类，其中一个会充当 GATT 服务器。

### 扫描

#### 声明蓝牙权限和定位权限

```xml
    <!-- Android 12以下才需要定位权限， Android 9以下官方建议申请ACCESS_COARSE_LOCATION -->
    <uses-permission
        android:name="android.permission.ACCESS_COARSE_LOCATION"
        android:maxSdkVersion="30" />
    <uses-permission
        android:name="android.permission.ACCESS_FINE_LOCATION"
        android:maxSdkVersion="30" />
    <uses-permission
        android:name="android.permission.BLUETOOTH"
        android:maxSdkVersion="30" />
    <uses-permission
        android:name="android.permission.BLUETOOTH_ADMIN"
        android:maxSdkVersion="30" />
    <!-- Android 12在不申请定位权限时，必须加上android:usesPermissionFlags="neverForLocation"，否则搜不到设备 -->
    <uses-permission
        android:name="android.permission.BLUETOOTH_SCAN"
        android:usesPermissionFlags="neverForLocation"
        tools:targetApi="s" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <!--ble模块 设置为true表示只有支持ble的手机才能安装-->
    <uses-feature
        android:name="android.hardware.bluetooth_le"
        android:required="true" />
```

由于蓝牙扫描需要用到模糊定位权限( `Android10` 后需要精准定位权限 )，所以`android6.0`之后,除了在 `AndroidManifest.xml`中 申明权限之外，还需要动态申请定位权限，才可进行蓝牙扫描，否则不会扫描到任何Ble设备。

> Android 12及以上机型，如果想不申请定位权限就能搜索到设备，必须在BLUETOOTH_SCAN权限上加上android:usesPermissionFlags="neverForLocation"，否则仍需申请定位权限。
>
> 可依据 `PackageManager.hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)` 获知该手机是否支持BLE

#### 设置BLE

如果不支持 BLE，则应妥善停用任何 BLE 功能。如果设备支持 BLE 但已停用此功能，则您可以请求用户在不离开应用的同时启用蓝牙。借助 `BluetoothAdapter`;

```java
//初始化ble设配器
BluetoothManager manager = (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
BluetoothAdapter mBluetoothAdapter = manager.getAdapter();
//判断蓝牙是否开启，如果关闭则请求打开蓝牙
if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
    //方式一：请求打开蓝牙
    Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(intent, 1);
    //方式二：半静默打开蓝牙
    //低版本android会静默打开蓝牙，高版本android会请求打开蓝牙
    //mBluetoothAdapter.enable();
}
```

同时可以在`activity`层通过广播监听蓝牙的关闭与开启，进行自己的逻辑处理：

```java
new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        //获取蓝牙广播  本地蓝牙适配器的状态改变时触发
        String action = intent.getAction();
        if (action.equals(BluetoothAdapter.ACTION_STATE_CHANGED)) {
            //获取蓝牙广播中的蓝牙新状态
            int blueNewState = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, 0);
            //获取蓝牙广播中的蓝牙旧状态
            int blueOldState = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, 0);
            switch (blueNewState) {
                //正在打开蓝牙
                case BluetoothAdapter.STATE_TURNING_ON:
                    break;
                    //蓝牙已打开
                case BluetoothAdapter.STATE_ON:
                    break;
                    //正在关闭蓝牙
                case BluetoothAdapter.STATE_TURNING_OFF:
                    break;
                    //蓝牙已关闭
                case BluetoothAdapter.STATE_OFF:
                    break;
            }
        }
    }
};
```

#### 开始扫描

在`android 4.3` 和 `android 4.4`进行蓝牙扫描：

```
//开始扫描：
mBluetoothAdapter.startLeScan(mLeScanCallback);
//停止扫描
mBluetoothAdapter.stopLeScan(mLeScanCallback);
```

在 `android 5.0`之后的版本（包括 5.0）建议使用新的Api进行蓝牙扫描：

```java
//获取 5.0 的扫描类实例
mBLEScanner = mBluetoothAdapter.getBluetoothLeScanner();
//开始扫描
//可设置过滤条件，在第一个参数传入，但一般不设置过滤。
mBLEScanner.startScan(null,mScanSettings,mScanCallback);
//停止扫描
mBLEScanner.stopScan(mScanCallback);
```

`Android 8`更新了一个扫描API，系统层为你提供后台持续扫描的能力。**(即便APP已被杀死，扫描仍会继续。**但如果用户重启或关闭蓝牙后，该扫描停止)： [BluetoothLeScanner#startScan](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fbluetooth%2Fle%2FBluetoothLeScanner%23startScan(java.util.List%3Candroid.bluetooth.le.ScanFilter%3E%2C%20android.bluetooth.le.ScanSettings%2C%20android.app.PendingIntent))

```java
public int startScan (List<ScanFilter> filters, 
                ScanSettings settings, 
                PendingIntent callbackIntent)
```

示例代码：

```java
//如果没打开蓝牙，不进行扫描操作，或请求打开蓝牙。
if(!mBluetoothAdapter.isEnabled()) {
    return;
}
 //处于未扫描的状态  
if (!mScanning){
    //android 5.0后
    if(android.os.Build.VERSION.SDK_INT >= 21) {
        //标记当前的为扫描状态
        mScanning = true;
        //获取5.0新添的扫描类
        if (mBLEScanner == null){
            //mBLEScanner是5.0新添加的扫描类，通过BluetoothAdapter实例获取。
            mBLEScanner = mBluetoothAdapter.getBluetoothLeScanner();
        }
        //开始扫描 
        //mScanSettings是ScanSettings实例，mScanCallback是ScanCallback实例，后面进行讲解。
        mBLEScanner.startScan(null,mScanSettings,mScanCallback);
    } else {
        //标记当前的为扫描状态
        mScanning = true;
        //5.0以下  开始扫描
        //mLeScanCallback是BluetoothAdapter.LeScanCallback实例
        mBluetoothAdapter.startLeScan(mLeScanCallback);
    }
    //设置结束扫描
    mHandler.postDelayed(new Runnable() {
        @Override
        public void run() {
            //停止扫描设备
            if(android.os.Build.VERSION.SDK_INT >= 21) {
                //标记当前的为未扫描状态
                mScanning = false;
                mBLEScanner.stopScan(mScanCallback);
            } else {
                //标记当前的为未扫描状态
                mScanning = false;
                //5.0以下  停止扫描
                mBluetoothAdapter.stopLeScan(mLeScanCallback);
            }
        }
    },SCAN_TIME);
}
```

> 1. 在开始扫描时，可先停止扫描再重新开始扫描；
> 2. `android 6.0` 以上需要获取到定位权限; 
> 3. `android 7.0` 后不能在30秒内扫描+停止超过5次

#### 扫描时的设置

##### `ScanSettings`

`ScanSettings`实例对象是通过`ScanSettings.Builder`构建的。通过`Builder`对象为`ScanSettings`实例设置扫描模式、回调类型、匹配模式等参数，用于配置`android 5.0` 的扫描参数。

```java
//创建ScanSettings的build对象用于设置参数
ScanSettings.Builder builder = new ScanSettings.Builder()
    //设置高功耗模式
    .setScanMode(SCAN_MODE_LOW_LATENCY);
    //android 6.0添加设置回调类型、匹配模式等
    if(android.os.Build.VERSION.SDK_INT >= 23) {
        //定义回调类型
        builder.setCallbackType(ScanSettings.CALLBACK_TYPE_ALL_MATCHES)
        //设置蓝牙LE扫描滤波器硬件匹配的匹配模式
        builder.setMatchMode(ScanSettings.MATCH_MODE_STICKY);
    }
//芯片组支持批处理芯片上的扫描
if (bluetoothadapter.isOffloadedScanBatchingSupported()) {
    //设置蓝牙LE扫描的报告延迟的时间（以毫秒为单位）
    //设置为0以立即通知结果
    builder.setReportDelay(0L);
}
builder.build();
```

配置描述：

- `setScanMode()`设置扫描模式。可选择模式主要三种( 从上到下，会越来越耗电,但扫描间隔越来越短，即扫描速度会越来越快。)：

  - `ScanSettings.SCAN_MODE_LOW_POWER` 低功耗模式(默认扫描模式,如果扫描应用程序不在前台，则强制使用此模式。)
  - `ScanSettings.SCAN_MODE_BALANCED` 平衡模式
  - `ScanSettings.SCAN_MODE_LOW_LATENCY` 高功耗模式(建议仅在应用程序在前台运行时才使用此模式。)

- `setCallbackType()`设置回调类型。可选择模式主要三种：

  - `ScanSettings.CALLBACK_TYPE_ALL_MATCHES` 数值: 1。

    寻找符合过滤条件的蓝牙广播，如果没有设置过滤条件，则返回全部广播包

  - `ScanSettings.CALLBACK_TYPE_FIRST_MATCH` 数值: 2

    仅针对与筛选条件匹配的第一个广播包触发结果回调。

  - `ScanSettings.CALLBACK_TYPE_MATCH_LOST` 数值: 4

    回调类型一般设置`ScanSettings.CALLBACK_TYPE_ALL_MATCHES`，有过滤条件时过滤，返回符合过滤条件的蓝牙广播。无过滤条件时，返回全部蓝牙广播。

- `setMatchMode()`设置蓝牙LE扫描滤波器硬件匹配的匹配模式

  - `ScanSettings.MATCH_MODE_STICKY` 粘性模式，在通过硬件报告之前，需要更高的信号强度和目击阈值
  - `MATCH_MODE_AGGRESSIVE`   激进模式，即使信号强度微弱且持续时间内瞄准/匹配的次数很少，hw也会更快地确定匹配。

- `Bluetoothadapter.isOffloadedScanBatchingSupported()`判断当前手机蓝牙芯片是否支持批处理扫描。

  - 如果支持扫描则使用批处理扫描，可通过`ScanSettings.Builder`对象调用`setReportDelay(Long)`方法来设置蓝牙LE扫描的报告延迟的时间（以毫秒为单位）来启动批处理扫描模式。

- `ScanSettings.Builder.setReportDelay(Long);`

  - 当设备蓝牙芯片支持批处理扫描时，用来设置蓝牙LE扫描的报告延迟的时间（以毫秒为单位）。
  - 该参数默认为 0，如果不修改它的值，则默认只会在`onScanResult(int,ScanResult)`中返回扫描到的蓝牙设备，不会触发`onBatchScanResults(List)`方法。(`onScanResult(int,ScanResult)` 和 `onBatchScanResults(List)` 是互斥的。 )
  - 设置为0以立即通知结果,不开启批处理扫描模式。即`ScanCallback`蓝牙回调中，不会触发`onBatchScanResults(List)`方法，但会触发`onScanResult(int,ScanResult)`方法，返回扫描到的蓝牙设备。
  - 当设置的时间大于0L时，则会开启批处理扫描模式。即触发`onBatchScanResults(List)`方法，返回扫描到的蓝牙设备列表。但不会触发`onScanResult(int,ScanResult)`方法。

#### 扫描后回调

##### `android 4.3` 的扫描回调接口`BluetoothAdapter.LeScanCallback`：

```java
//5.0以下
mLeScanCallback = new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(BluetoothDevice device, int rssi, byte[] scanRecord) {
        //对扫描到的设备进行操作。如：获取设备信息。
    }
};
```

参数：

- `BluetoothDevice` 扫描到的设备实例，可从实例中获取到相应的信息。如：名称，mac地址
- `rssi` 可理解成设备的信号值。该数值是一个负数，越大则信号越强。
- `scanRecord` 远程设备提供的广播数据的内容。

获取`BluetoothDevice`中的信息：

- `getAddress()`：返回此BluetoothDevice的硬件地址；mac可用于再创建`BluetoothDevice`对象进行`gatt`连接；
- `getBondState()`：获取远程设备的绑定状态；
  - `BOND_NONE`：数值 10
     表示远程设备未绑定，没有共享链接密钥，因此通信（如果允许的话）将是未经身份验证和未加密的。（扫描到未绑定的小米手环）
  - `BOND_BONDING`：数值 11 表示正在与远程设备进行绑定;
  - `BOND_BONDED`：数值 12 表示远程设备已绑定，远程设备本地存储共享连接的密钥，因此可以对通信进行身份验证和加密。（扫描到已绑定的小米手环）
- `getName()`：获取远程设备的设置蓝牙名称；
- `getType()`：获取远程设备的设置蓝牙设备类型；一般是2，表示LE设备；

##### android 5.0以上 扫描回调：`ScanCallback`

```java
mScanCallback = new ScanCallback() {
    //当一个蓝牙ble广播被发现时回调
    @Override
    public void onScanResult(int callbackType, ScanResult result) {
        super.onScanResult(callbackType, result);
        //扫描类型有开始扫描时传入的ScanSettings相关
        //对扫描到的设备进行操作。如：获取设备信息。
    }

    //批量返回扫描结果
    //@param results 以前扫描到的扫描结果列表。
    @Override
    public void onBatchScanResults(List<ScanResult> results) {
        super.onBatchScanResults(results);
    }

    //当扫描不能开启时回调
    @Override
    public void onScanFailed(int errorCode) {
        super.onScanFailed(errorCode);
        //扫描太频繁会返回ScanCallback.SCAN_FAILED_APPLICATION_REGISTRATION_FAILED，表示app无法注册，无法开始扫描。
    }
};
```

> - 回调函数中尽量不要做耗时操作！
> - 一般蓝牙设备对象都是通过`onScanResult(int,ScanResult)`返回，而不会在`onBatchScanResults(List)`方法中返回，除非手机支持批量扫描模式并且开启了批量扫描模式。批处理的开启请查看`ScanSettings`。

### 通信

#### 蓝牙基础协议

蓝牙两个最基本的协议：**GAP 和 GATT**

##### GAP（Generic Access Profile）简介

GAP是通用访问配置文件的首字母缩写，主要控制蓝牙连接和广播。GAP使蓝牙设备对外界可见，并决定设备是否可以或者怎样与其他设备进行交互。

GAP定义了多种角色，但主要的两个是：中心设备 和 外围设备。

- **中心设备**：可以扫描并连接多个外围设备,从外设中获取信息。
- **外围设备**：小型，低功耗，资源有限的设备。可以连接到功能更强大的中心设备，并为其提供数据。

##### GAP 广播数据

GAP 中外围设备通过两种方式向外广播数据：广播数据 和 扫描回复( 每种数据最长可以包含 31 byte)。

广播数据是必需的，因为外设必需不停的向外广播，让中心设备知道它的存在。而扫描回复是可选的，中心设备可以向外设请求扫描回复，这里包含一些设备额外的信息。

![img](https://upload-images.jianshu.io/upload_images/6974508-8c5617ac280868cb?imageMogr2/auto-orient/strip|imageView2/2/w/773/format/webp)

外围设备会设定一个广播间隔。每个广播间隔中，它会重新发送自己的广播数据。广播间隔越长，越省电，同时也不太容易扫描到。

外设通过广播自己让中心设备发现自己，并建立 GATT 连接，从而进行更多的数据交换。但有些情况是不需要连接的，只要外设广播自己的数据即可。目的是让外围设备，把自己的信息发送给多个中心设备。因为基于 GATT 连接的方式的，只能是一个外设连接一个中心设备。

##### GATT（Generic Attribute Profile）简介

GATT配置文件是一个通用规范，用于在BLE链路上发送和接收被称为“属性”的数据块。目前所有的BLE应用都基于GATT。

BLE设备通过叫做 **Service** 和 **Characteristic** 的东西进行通信

GATT使用了 ATT（Attribute Protocol）协议，ATT 协议把 Service, `Characteristic`对应的数据保存在一个查询表中，次查找表使用 16 bit ID 作为每一项的索引。

 GATT 连接是**独占**的。也就是**一个 BLE 外设同时只能被一个中心设备连接。**一旦外设被连接，它就会马上停止广播，这样它就对其他设备不可见了。当外设与中心设备断开，外设又开始广播，让其他中心设备感知该外设的存在。而中心设备可**同时**与多个外设进行连接。

![img](https:////upload-images.jianshu.io/upload_images/6974508-ed44a780d84fbe0a?imageMogr2/auto-orient/strip|imageView2/2/w/762/format/webp)

##### GATT  通信

中心设备和外设需要**双向通信**的话，唯一的方式就是建立 GATT 连接。

GATT 通信的双方是 C/S 关系。外设作为 GATT 服务端（Server），它维持了 ATT 的查找表以及 service 和 `characteristic` 的定义。中心设备是 GATT 客户端（Client），它向 外设（Server） 发起请求来获取数据。

#### GATT 结构

![img](https://upload-images.jianshu.io/upload_images/6974508-e5cca6da09deb021?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- **Profile**：并不是实际存在于 BLE 外设上的，它只是一个被 Bluetooth SIG 或者外设设计者预先定义的 `Service` 的集合。例如心率`Profile`（`Heart Rate Profile`）就是结合了 `Heart Rate Service` 和 `Device Information Service`。
- **Service**：包含一个或者多个 `Characteristic`。每个 `Service` 有一个 `UUID` 唯一标识。
- **Characteristic**： 是最小的逻辑数据单元。一个`Characteristic`包括一个单一value变量和0-n个用来描述`characteristic`变量的`Descriptor`。与 `Service` 类似，每个 `Characteristic` 用 16 bit 或者 128 bit 的 `UUID` 唯一标识。

**实际开发中，和 BLE 外设打交道，主要是通过 `Characteristic`。可以从 `Characteristic` 读取数据，也可以往 `Characteristic` 写数据，从而实现双向的通信。**

```undefined
UUID 有 16 bit 、32bit 和 128 bit 的。16 bit 的 UUID 是官方通过认证的，需要花钱购买。
```

Bluetooth_Base_UUID定义为 `00000000-0000-1000-8000-00805F9B34FB`

- 若16 bit UUID为xxxx，转换为128 bit UUID为`0000xxxx-0000-1000-8000-00805F9B34FB`
- 若32 bit UUID为xxxxxxxx，转换为128 bit UUID为`xxxxxxxx-0000-1000-8000-00805F9B34FB`

#### 中心设备与外设通讯

简单介绍BLE开发当中各种主要类和其作用：

- **BluetoothDeivce**：蓝牙设备，代表一个具体的蓝牙外设。
- **BluetoothGatt**：通用属性协议，定义了BLE通讯的基本规则和操作
- **BluetoothGattCallback**：GATT通信回调类，用于回调的各种状态和结果。
- **BluetoothGattService**：服务，由零或多个特征组构成。
- **BluetoothGattCharacteristic**：特征，里面包含了一组或多组数据，是GATT通信中的最小数据单元。
   **BluetoothGattDescriptor**：特征描述符，对特征的额外描述，包括但不仅限于特征的单位，属性等。

##### 获取蓝牙设备对象

对扫描到的蓝牙可以用集合形式进行缓存，也可只保存其mac地址，存储到字符集合中，用于后续的连接。

根据mac地址获取到BluetoothDeivce用于连接：

```java
BluetoothManager bluetoothmanager = (BluetoothManager)context.getSystemService(Context.BLUETOOTH_SERVICE);
 mBluetoothAdapter = bluetoothmanager.getAdapter();
 //获取蓝牙设备对象进行连接
mBluetoothDevice = mBluetoothAdapter.getRemoteDevice(macAddressStr)
```

##### 连接设备

作为中心设备，连接其它外设设备时；调用`BluetoothDevice#connectGatt()`进行ble连接，第二个参数默认选择false,不自动连接。并定义`BluetoothGatt`变量，存储`BluetoothDevice#connectGatt()`返回的对象。

![img](https://upload-images.jianshu.io/upload_images/6974508-8417b061398928e6?imageMogr2/auto-orient/strip|imageView2/2/w/859/format/webp)

```java
//定义Gatt实现类
private BluetoothGatt mBluetoothGatt; 

//创建Gatt回调
private BluetoothGattCallback mGattCallback = new daqiBluetoothGattCallback();
//连接设备
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    mBluetoothGatt = mBluetoothDevice.connectGatt(mContext,
            false, mGattCallback, BluetoothDevice.TRANSPORT_LE);
} else {
    mBluetoothGatt = mBluetoothDevice.connectGatt(mContext, false, mGattCallback);
}
```

而mGattCallback对象则实现`BluetoothGattCallBack`类，监听蓝牙连接过程中各种回调的监听。**蓝牙Gatt回调方法中都不应该进行耗时操作，需要将其方法内进行的操作丢进另一个线程，尽快返回。**

```java
//定义子线程handle，用于在BluetoothGattCallback中回调方法中的操作抛到该线程工作。
private Handler mHandler;
//定义handler工作的子线程
private HandlerThread mHandlerThread;

初始化handler
mHandlerThread = new HandlerThread("daqi");
mHandlerThread.start();
//将handler绑定到子线程中
mHandler = new Handler(mHandlerThread.getLooper());
//定义重连次数
private int reConnectionNum = 0;
//最多重连次数
private int maxConnectionNum = 3;

//定义蓝牙Gatt回调类
public class daqiBluetoothGattCallback extends BluetoothGattCallback{
        //连接状态回调
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
            super.onConnectionStateChange(gatt, status, newState);
            // status 用于返回操作是否成功,会返回异常码。
            // newState 返回连接状态，如BluetoothProfile#STATE_DISCONNECTED、BluetoothProfile#STATE_CONNECTED
            
            //操作成功的情况下
            if (status == BluetoothGatt.GATT_SUCCESS){
                //判断是否连接码
                if (newState == BluetoothProfile.STATE_CONNECTED) {
                
                } else if(newState == BluetoothProfile.STATE_DISCONNECTED){
                    //判断是否断开连接码
                    
                }
            }else{
                //异常码
                //重连次数不大于最大重连次数
            if(reConnectionNum < maxConnectionNum){
                //重连次数自增
                reConnectionNum++
                //连接设备
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    mBluetoothGatt = mBluetoothDevice.connectGatt(mContext,
                            false, mGattCallback, BluetoothDevice.TRANSPORT_LE);
                } else {
                    mBluetoothGatt = mBluetoothDevice.connectGatt(mContext, false, mGattCallback);
                }
            }else{
                //断开连接，返回连接失败回调
                
            }
                
            }
        }
        
        //服务发现回调
        @Override
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            super.onServicesDiscovered(gatt, status);
        }

        //特征写入回调
        @Override
        public void onCharacteristicWrite(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
            super.onCharacteristicWrite(gatt, characteristic, status);
        }
        
        //外设特征值改变回调
        @Override
        public void onCharacteristicChanged(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {
            super.onCharacteristicChanged(gatt, characteristic);
        }
        
        //描述写入回调
        @Override
        public void onDescriptorWrite(BluetoothGatt gatt, BluetoothGattDescriptor descriptor, int status) {
            super.onDescriptorWrite(gatt, descriptor, status);
        }
    }
```

蓝牙连接成功(连接上和断开动作)和失败时，都会触发`BluetoothGattCallback#onConnectionStateChange()`方法；

错误代码:

- 133 ：连接超时或未找到设备；
- 8 ： 设备超出范围；
- 22 ：表示本地设备终止了连接

当发现服务成功后，会触发`BluetoothGattCallback#onServicesDiscovered()`回调：

```java
/定义需要进行通信的ServiceUUID
private UUID mServiceUUID = UUID.fromString("0000xxxx-0000-1000-8000-00805f9b34fb");

//定义蓝牙Gatt回调类
public class daqiBluetoothGattCallback extends BluetoothGattCallback{

    //服务发现回调
    @Override
    public void onServicesDiscovered(BluetoothGatt gatt, int status) {
        super.onServicesDiscovered(gatt, status);
        if (status == BluetoothGatt.GATT_SUCCESS) {
            mHandler.post(() ->
                //获取指定uuid的service
                BluetoothGattService gattService = mBluetoothGatt.getService(mServiceUUID);
                //获取到特定的服务不为空
                if(gattService != null){
                    
                }else{
                    //获取特定服务失败
                }
            );
        }
    }
｝
```

另外发现服务时，会存在发现不了特定服务的情况。或者说，整个`BluetoothGatt`对象中的服务列表为空。`BluetoothGatt`类中存在一个隐藏的方法`refresh（）`，用于刷新Gatt的服务列表。当发现不了服务时，可以通过反射去调用该方法，再发现一遍服务。

##### 读取和修改特征值

```java
//定义需要进行通信的ServiceUUID
private UUID mServiceUUID = UUID.fromString("0000xxxx-0000-1000-8000-00805f9b34fb");
//定义需要进行通信的CharacteristicUUID
private UUID mCharacteristicUUID = UUID.fromString("0000yyyy-0000-1000-8000-00805f9b34fb");


//定义蓝牙Gatt回调类
public class daqiBluetoothGattCallback extends BluetoothGattCallback{

    //服务发现回调
    @Override
    public void onServicesDiscovered(BluetoothGatt gatt, int status) {
        super.onServicesDiscovered(gatt, status);
        if (status == BluetoothGatt.GATT_SUCCESS) {
            mHandler.post(() ->
                //获取指定uuid的service
                BluetoothGattService gattService = mBluetoothGatt.getService(mServiceUUID);
                //获取到特定的服务不为空
                if(gattService != null){
                    //获取指定uuid的Characteristic
                    BluetoothGattCharacteristic gattCharacteristic = gattService.getCharacteristic(mCharacteristicUUID);
                    //获取特定特征成功
                    if（gattCharacteristic != null）{
                        //写入你需要传递给外设的特征值（即传递给外设的信息）
                        gattCharacteristic.setValue(bytes);
                        //通过GATt实体类将，特征值写入到外设中。
                        mBluetoothGatt.writeCharacteristic(gattCharacteristic);
                        
                        //如果只是需要读取外设的特征值：
                        //通过Gatt对象读取特定特征（Characteristic）的特征值
                        mBluetoothGatt.readCharacteristic(gattCharacteristic);
                    }
                }else{
                    //获取特定服务失败
                    
                }
            );
        }
    }
｝
```

当成功读取特征值时，会触发`BluetoothGattCallback#onCharacteristicRead()`回调：

```java
@Override
public void onCharacteristicRead(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
    super.onCharacteristicRead(gatt, characteristic, status);
    if (status == BluetoothGatt.GATT_SUCCESS) {
        //获取读取到的特征值
        characteristic.getValue()
    ｝
}
```

当成功写入特征值到外设时，会触发`BluetoothGattCallback#onCharacteristicWrite()`回调：

```java
@Override
public void onCharacteristicWrite(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
    super.onCharacteristicWrite(gatt, characteristic, status);
    if (status == BluetoothGatt.GATT_SUCCESS) {
        //获取写入到外设的特征值
        characteristic.getValue()
    ｝
}
```

##### 监听外设特征值改变

无论是对外设写入新值，还是读取外设特定`Characteristic`的值，其实都只是单方通信。如果需要双向通信，可以在`BluetoothGattCallback#onServicesDiscovered`中对某个特征值设置监听（**前提是该`Characteristic`具有NOTIFY属性**）：

```java
//设置订阅notificationGattCharacteristic值改变的通知
mBluetoothGatt.setCharacteristicNotification(notificationGattCharacteristic, true);
//获取其对应的通知Descriptor
BluetoothGattDescriptor descriptor = notificationGattCharacteristic.getDescriptor(UUID.fromString("00002902-0000-1000-8000-00805f9b34fb"));
if (descriptor != null){ 
    //设置通知值
    descriptor.setValue(BluetoothGattDescriptor.ENABLE_INDICATION_VALUE);
    boolean descriptorResult = mBluetoothGatt.writeDescriptor(descriptor);
}
```

当写入完特征值后，外设修改自己的特征值进行回复时，手机端会触发`BluetoothGattCallback#onCharacteristicChanged()`方法，获取到外设回复的值，从而实现双向通信。

```java
@Override
public void onCharacteristicChanged(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {
     if (status == BluetoothGatt.GATT_SUCCESS) {
        //获取外设修改的特征值
        String value = characteristic.getValue()
        //对特征值进行解析
    ｝
｝
```

##### 断开连接

断开连接的操作分为两步：

- mBluetoothGatt.disconnect();
- mBluetoothGatt.close();

调用`disconnect()`后，会触发手机会触发`BluetoothGattCallback#onConnectionStateChange()`的回调，回调断开连接信息，`newState = BluetoothProfile.STATE_DISCONNECTED`。但调用完`disconnect()`紧接着马上调用`close()`，会终止`BluetoothGattCallback#onConnectionStateChange()`的回调。可以看情况将两个进行拆分调用，来实现断开连接，但必须两个方法都调用。一般会在onConnectionStateChange()状态为连接不成功或断开时调用 其close()方法

> 当和外设进行ble通信时，如出现任何意外情况，马上调用断开连接操作。

### 广播

在蓝牙开发中，有些情况是不需要连接的，只要外围设备广播自己的数据即可，例如苹果的`ibeacon`。自`Android 5.0`更新蓝牙API后，手机可以作为外设广播数据。

广播包有两种：

- 广播包（Advertising Data）
- 响应包（Scan Response）

其中**广播包是每个外设都必须广播的，而响应包是可选的**。每个广播包的长度必须是**31个字节**，如果不到**31个字节** ，则剩下的全用**0**填充 补全，这部分的数据是无效的

![img](https:////upload-images.jianshu.io/upload_images/6974508-d01d3ea218fb54a9?imageMogr2/auto-orient/strip|imageView2/2/w/773/format/webp)

#### 广播数据单元

广播包中包含若干个广播数据单元，广播数据单元也称为 `AD Structure`。

**广播数据单元 = 长度值Length + AD type + AD Data。**

长度值`Length`只占**一个字节**，并且位于广播数据单元的**第一个字节**。

概念的东西有些抽象，先看看下面的广播报文：



![img](https:////upload-images.jianshu.io/upload_images/6974508-11be468e55918845?imageMogr2/auto-orient/strip|imageView2/2/w/445/format/webp)

​       0x代表这串字符串是十六进制的字符串。**两位十六进制数代表一个字节**。因为两个字符组成的十六进制字符串最大为`FF`，即255，而Java中byte类型的取值范围是-128到127，刚好可以表示一个255的大小。所以两个十六进制的字符串表示一个字节。

​       继续查看报文内容，开始读取第一个广播数据单元。读取**第一个**字节:`0x07`,转换为十进制就是7，即表示后面的7个字节是这个广播数据单元的数据内容。超过这7个字节的数据内容后，表示是一个新的广播数据单元。

​       而第二个广播数据单元，第一个字节的值是`0x16`,转换为十进制就是22，表示后面22个字节为第二个广播数据单元。

​       在广播数据单元的**数据部分**中，**第一个字节**代表**数据类型**（AD type），决定数据部分表示的是什么数据。（即广播数据单元第二个字节为AD type）

![img](https:////upload-images.jianshu.io/upload_images/6974508-63821d97bf504563?imageMogr2/auto-orient/strip|imageView2/2/w/662/format/webp)

**AD Type**的类型如下：

- Flags：TYPE = 0x01

  。用来标识设备LE物理连接。

  - bit 0: LE 有限发现模式
  - bit 1: LE 普通发现模式
  - bit 2: 不支持 BR/EDR
  - bit 3: 对 Same Device Capable(Controller) 同时支持 BLE 和 BR/EDR
  - bit 4: 对 Same Device Capable(Host) 同时支持 BLE 和 BR/EDR
  - bit 5..7: 预留

​       这bit 1~7分别代表着发送该广播的蓝牙芯片的物理连接状态。当bit的值为1时，表示支持该功能。
 例：

![img](https:////upload-images.jianshu.io/upload_images/6974508-b0c30d7be474f0c6?imageMogr2/auto-orient/strip|imageView2/2/w/486/format/webp)

- Service `UUID`。广播数据中可以将设备支持的GATT Service的`UUID`广播出来，来告知中心设备其支持的Service。对于不同bit的`UUID`,其对应的类型也有不同：
  - 非完整的16bit `UUID`: TYPE = **0x02**;
  - 完整的16bit `UUID` 列表: TYPE = **0x03**;
  - 非完整的32bit `UUID` 列表: TYPE = **0x04**;
  - 完整的32bit `UUID` 列表: TYPE = **0x05**;
  - 非完整的128bit `UUID` 列表: TYPE = **0x06**;
  - 完整的128bit `UUID`: TYPE = **0x07**;
- TX Power Level: TYPE = **0x0A**，表示设备发送广播包的信号强度。 数值范围：±127 dBm。
- 设备名字，DATA 是名字的字符串，可以是设备的全名，也可以是设备名字的缩写。
  - 缩写的设备名称： TYPE = **0x08**
  - 完整的设备名称： TYPE = **0x09**
- Service Data: Service 对应的数据。
  - 16 bit `UUID` Service: TYPE = **0x16**, 前 2 字节是 `UUID`，后面是 Service 的数据；
  - 32 bit `UUID` Service: TYPE = **0x20**, 前 4 字节是 `UUID`，后面是 Service 的数据；
  - 128 bit `UUID` Service: TYPE = **0x21**, 前 16 字节是 `UUID`，后面是 Service 的数据；
- 厂商自定义数据: TYPE = **0xFF**。厂商数据中，前两个字节表示厂商ID,剩下的是厂商自定义的数据。

#### BLE广播

##### 自定义`UUID`

```tsx
//`UUID`
public static `UUID` UUID_SERVICE = `UUID`.fromString("0000fff7-0000-1000-8000-00805f9b34fb");
```

> 开启广播一般需要3~4对象：广播设置（AdvertiseSettings）、广播包（AdvertiseData）、扫描包（可选）、广播回调（AdvertiseCallback）。

##### 广播设置

定义：

```java
//初始化广播设置
mAdvertiseSettings = new AdvertiseSettings.Builder()
        //设置广播模式，以控制广播的功率和延迟。
        .setAdvertiseMode(AdvertiseSettings.ADVERTISE_MODE_LOW_POWER)
        //发射功率级别
        .setTxPowerLevel(AdvertiseSettings.ADVERTISE_TX_POWER_HIGH)
        //不得超过180000毫秒。值为0将禁用时间限制。
        .setTimeout(3000)
        //设置是否可以连接
        .setConnectable(false)
        .build();
```

- `setAdvertiseMode()`设置广播模式:
  - 在均衡电源模式下执行蓝牙LE广播:`AdvertiseSettings#ADVERTISE_MODE_BALANCED`
  - 在低延迟，高功率模式下执行蓝牙LE广播: `AdvertiseSettings#ADVERTISE_MODE_LOW_LATENCY`
  - 在低功耗模式下执行蓝牙LE广播:`AdvertiseSettings#ADVERTISE_MODE_LOW_POWER`
- `setAdvertiseMode()`设置广播发射功率:
  - 使用高TX功率级别进行广播：`AdvertiseSettings#ADVERTISE_TX_POWER_HIGH`
  - 使用低TX功率级别进行广播：`AdvertiseSettings#ADVERTISE_TX_POWER_LOW`
  - 使用中等TX功率级别进行广播：`AdvertiseSettings#ADVERTISE_TX_POWER_MEDIUM`
  - 使用最低传输（TX）功率级别进行广播：`AdvertiseSettings#ADVERTISE_TX_POWER_ULTRA_LOW`
- `setTimeout()`设置持续广播的时间，单位为毫秒。最多180000毫秒。当值为0则无时间限制，持续广播，除非调用`BluetoothLeAdvertiser#stopAdvertising()`停止广播。
- `setConnectable()`设置该广播是否可以连接的。

##### 广播包

外设必须广播广播包，扫描包是可选。但添加扫描包也意味着广播更多得数据，即可广播62个字节。

```java
//初始化广播包
mAdvertiseData = new AdvertiseData.Builder()
        //设置广播设备名称
        .setIncludeDeviceName(true)
        //设置发射功率级别
        .setIncludeTxPowerLevel(true)
        .build();
        
//初始化扫描响应包
mScanResponseData = new AdvertiseData.Builder()
        //隐藏广播设备名称
        .setIncludeDeviceName(false)
        //隐藏发射功率级别
        .setIncludeTxPowerLevel(false)
        //设置广播的服务`UUID`
        .addService`UUID`(new Parcel`UUID`(UUID_SERVICE))
        //设置厂商数据
        .addManufacturerData(0x11,hexStrToByte(mData))
        .build();
```

无论是广播包还是扫描包，其广播的内容都是用`AdvertiseData`类封装的。

- `setIncludeDeviceName()`方法，可以设置广播包中是否包含蓝牙的名称;
- `setIncludeTxPowerLevel()`方法，可以设置广播包中是否包含蓝牙的发射功率;
- `addService`UUID`(Parcel`UUID`)`方法，可以设置特定的`UUID`在广播包中。
- `addServiceData(Parcel`UUID`，byte[])`方法，可以设置特定的`UUID`和其数据在广播包中;
- `#addManufacturerData(int，byte[])`方法，可以设置特定厂商Id和其数据在广播包中。

从`AdvertiseData.Builder`的设置中可以看出，如果一个外设需要在不连接的情况下对外广播数据，其数据可以存储在`UUID`对应的数据中，也可以存储在厂商数据中。但由于厂商ID是需要由Bluetooth SIG进行分配的，厂商间一般都将数据设置在厂商数据。

##### 广播名称

可以通过`BluetoothAdapter#setName()`设置广播的名称

```cpp
//获取蓝牙设配器
BluetoothManager bluetoothManager = (BluetoothManager)
        getSystemService(Context.BLUETOOTH_SERVICE);
mBluetoothAdapter = bluetoothManager.getAdapter();
//设置设备蓝牙名称
mBluetoothAdapter.setName("daqi");
```

##### 发送BLE广播

最后继承`AdvertiseCallback`自定义广播回调。

```java
private class daqiAdvertiseCallback extends AdvertiseCallback {
    //开启广播成功回调
    @Override
    public void onStartSuccess(AdvertiseSettings settingsInEffect){
        super.onStartSuccess(settingsInEffect);
        Log.d("daqi","开启服务成功");
    }

    //无法启动广播回调。
    @Override
    public void onStartFailure(int errorCode) {
        super.onStartFailure(errorCode);
        Log.d("daqi","开启服务失败，失败码 = " + errorCode);
    }
}
```

初始化完毕上面的对象后，就可以进行广播：

```csharp
//获取BLE广播的操作对象。
//如果蓝牙关闭或此设备不支持蓝牙LE广播，则返回null。
mBluetoothLeAdvertiser = mBluetoothAdapter.getBluetoothLeAdvertiser();
//mBluetoothLeAdvertiser不为空，且蓝牙已开打
if(mBluetoothAdapter.isEnabled()){
    if (mBluetoothLeAdvertiser != null){
         //开启广播
        mBluetoothLeAdvertiser.startAdvertising(mAdvertiseSettings,
            mAdvertiseData, mScanResponseData, mAdvertiseCallback);
    }else {
        Log.d("daqi","该手机不支持ble广播");
    }
}else{
    Log.d("daqi","手机蓝牙未开启");
}
```

​       广播主要是通过`BluetoothLeAdvertiser#startAdvertising()`方法实现，但在之前需要先获取`BluetoothLeAdvertiser`对象。

`BluetoothLeAdvertiser`对象存在两个情况获取为Null:

- 手机蓝牙模块不支持BLE广播
- 蓝牙未开启

所以在调用`BluetoothAdapter#getBluetoothLeAdvertiser()`前，需要先调用判断蓝牙已开启，并判断在`BluetoothAdapter`中获取的`BluetoothLeAdvertiser`是否为空。

​       与广播成对出现就是`BluetoothLeAdvertiser.stopAdvertising()`停止广播了，传入开启广播时传递的广播回调对象，即可关闭广播：

```css
mBluetoothLeAdvertiser.stopAdvertising(mAdvertiseCallback)
```

##### 启动GATT Service 和 Characteristic

虽然通过广播告知外边自身拥有这些Service,但手机自身并没有初始化Gattd的Service。导致外部的中心设备连接手机后，并不能找到对应的`GATT Service` 和 获取对应的数据。

Service类型有两个级别：

- `BluetoothGattService#SERVICE_TYPE_PRIMARY` 主服务
- `BluetoothGattService#SERVICE_TYPE_SECONDARY`次要服务（存在于主服务中的服务）

###### 创建`BluetoothGattService`

创建`BluetoothGattService`时，传入两个参数：`UUID`和Service类型：

```java
BluetoothGattService service = new BluetoothGattService(UUID_SERVICE,
                                BluetoothGattService.SERVICE_TYPE_PRIMARY);
```

###### 创建Gatt Characteristic

`Service`的下一级是`Characteristic`，`Characteristic`是最小的通信单元，通过对`Characteristic`进行读写操作来进行通信。

```cpp
//初始化特征值
mGattCharacteristic = new BluetoothGattCharacteristic(UUID_CHARACTERISTIC,
        BluetoothGattCharacteristic.PROPERTY_WRITE|
                BluetoothGattCharacteristic.PROPERTY_NOTIFY|
                BluetoothGattCharacteristic.PROPERTY_READ,
        BluetoothGattCharacteristic.PERMISSION_WRITE|
                BluetoothGattCharacteristic.PERMISSION_READ);
```

> 创建`BluetoothGattCharacteristic`时，传入三个参数：`UUID`、特征属性 和 权限属性。

​       特征属性表示该`BluetoothGattCharacteristic`拥有什么功能，即能对`BluetoothGattCharacteristic`进行什么操作。其中主要有3种：

- `BluetoothGattCharacteristic#PROPERTY_WRITE` 表示特征支持写
- `BluetoothGattCharacteristic#PROPERTY_READ`  表示特征支持读
- `BluetoothGattCharacteristic#PROPERTY_NOTIFY`  表示特征支持通知

权限属性用于配置该特征值所具有的功能。主要两种：

- `BluetoothGattCharacteristic#PERMISSION_WRITE`  特征写权限
- `BluetoothGattCharacteristic#PERMISSION_READ`  特征读权限

注意事项

- 当特征值只有读权限时，调用`BluetoothGatt#writeCharacteristic()`对特征值进行修改时，将返回false，无法写入。并不会触发`BluetoothGattCallback#onCharacteristicWrite()`回调。
- 当特征值只有写权限时，调用`BluetoothGatt#readCharacteristic()`对特征值进行读取时，将返回false，无法写入。并不会触发`BluetoothGattCallback#onCharacteristicRead()`回调。

###### 创建Gatt Descriptor

`Characteristic`下还有`Descriptor`，初始化`BluetoothGattDescriptor`时传入：`Descriptor UUID` 和 权限属性; 

```cpp
//初始化描述
mGattDescriptor = new BluetoothGattDescriptor(UUID_DESCRIPTOR,BluetoothGattDescriptor.PERMISSION_WRITE);
```

###### 添加 Characteristic 和 Descriptor

为`Service`添加`Characteristic`，为`Characteristic`添加`Descriptor`：

```cpp
//Service添加特征值
mGattService.addCharacteristic(mGattCharacteristic);
mGattService.addCharacteristic(mGattReadCharacteristic);
//特征值添加描述
mGattCharacteristic.addDescriptor(mGattDescriptor);
```

​       通过蓝牙管理器`mBluetoothManager`获取`Gatt Server`，用来添加`Gatt Service`。添加完`Gatt Service`后，外部中心设备连接手机时，将能获取到对应的`GATT Service` 和 获取对应的数据

```dart
//初始化GattServer回调
mBluetoothGattServerCallback = new daqiBluetoothGattServerCallback();

if (mBluetoothManager != null)
    mBluetoothGattServer = mBluetoothManager.openGattServer(this, mBluetoothGattServerCallback);
boolean result = mBluetoothGattServer.addService(mGattService);
if (result){
    Toast.makeText(daqiActivity.this,"添加服务成功",Toast.LENGTH_SHORT).show();
}else {
    Toast.makeText(daqiActivity.this,"添加服务失败",Toast.LENGTH_SHORT).show();
}
```

​       定义`Gatt Server`回调。当中心设备连接该手机外设、修改特征值、读取特征值等情况时，会得到相应情况的回调。

```java
private class daqiBluetoothGattServerCallback extends BluetoothGattServerCallback{

    //设备连接/断开连接回调
    @Override
    public void onConnectionStateChange(BluetoothDevice device, int status, int newState) {
        super.onConnectionStateChange(device, status, newState);
    }

    //添加本地服务回调
    @Override
    public void onServiceAdded(int status, BluetoothGattService service) {
        super.onServiceAdded(status, service);
    }
    
    //特征值读取回调
    @Override
    public void onCharacteristicReadRequest(BluetoothDevice device, int requestId, int offset, BluetoothGattCharacteristic characteristic) {
        super.onCharacteristicReadRequest(device, requestId, offset, characteristic);
    }
    
    //特征值写入回调
    @Override
    public void onCharacteristicWriteRequest(BluetoothDevice device, int requestId, BluetoothGattCharacteristic characteristic, boolean preparedWrite, boolean responseNeeded, int offset, byte[] value) {
        super.onCharacteristicWriteRequest(device, requestId, characteristic, preparedWrite, responseNeeded, offset, value);
    }
    
    //描述读取回调
    @Override
    public void onDescriptorReadRequest(BluetoothDevice device, int requestId, int offset, BluetoothGattDescriptor descriptor) {
        super.onDescriptorReadRequest(device, requestId, offset, descriptor);
    }
    
    //描述写入回调
    @Override
    public void onDescriptorWriteRequest(BluetoothDevice device, int requestId, BluetoothGattDescriptor descriptor, boolean preparedWrite, boolean responseNeeded, int offset, byte[] value) {
        super.onDescriptorWriteRequest(device, requestId, descriptor, preparedWrite, responseNeeded, offset, value);
    }
}
```

### MTU

#### 概念

MTU是指在一个协议数据单元中（`Protocol Data Unit, PDU`) 有效的最大传输`Byte`。

MTU默认是`23byte`,但是供我们使用的只有`20byte`。所以有时候不能满足我们的需求，需要我们手动设置MTU的大小。

> **为什么为限制成20个字节？**
>
> core spec里面定义了ATT的默认MTU为23个bytes， 除去ATT的opcode一个字节以及ATT的handle 2个字节之后，剩下的20个字节便是留给GATT的了。
>
> 考虑到有些Bluetooth smart设备功能弱小，不敢太奢侈的使用内存空间，**因此****core spec****规定每一个设备都必须支持MTU****为23****。**
>
> 在两个设备连接初期，大家都像新交的朋友一样，不知对方底细，因此严格的按照套路来走，即最多一次发20个字节，是最保险的。
>
> **由于ATT的最大长度为512byte**
>
> 因此一般认为MTU的最大长度为512个byte就够了，再大也没什么意义，你不可能发一个超过512的ATT的数据。
>
> 所以ATT的MTU的最大长度可视为512个bytes。
>
> http://eleaction01.spaces.eepw.com.cn/articles/article/item/196562

在Android中修改MTU很简单，在client端中， 只需要调用BluetoothGatt.requestMtu（int MTU）方法即可。requestMtu（intMTU）必须在发现蓝牙服务并建立蓝牙服务连接之后才能调用，否则MTU会默认为20Byte。如果调用成功会自定回调BluetoothGattCallback类中的onMtuChanged(BluetoothGatt gatt, int mtu, int status)方法。

```java
boolean isSuccess = gatt.requestMtu(int);
```



本文来自于：https://www.jianshu.com/p/2dba7f067372及其系列文章， 