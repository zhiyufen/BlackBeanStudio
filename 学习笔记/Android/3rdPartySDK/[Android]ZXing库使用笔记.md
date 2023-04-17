# [Android]ZXing库使用笔记

[TOC]

## 概念

### 二维码

二维条码是指在一维条码（条形码）的基础上扩展出另一维具有可读性的条码，使用黑白矩形图案表示二进制数据，被设备扫描后可获取其中所包含的信息。一维条码的宽度记载着数据，而其长度没有记载数据。二维条码的长度、宽度均记载着数据。二维条码有一维条码没有的“定位点”和“容错机制”。容错机制在即使没有辨识到全部的条码、或是说条码有污损时，也可以正确地还原条码上的信息。二维条码的种类很多，不同的机构开发出的二维条码具有不同的结构以及编写、读取方法。
![img](https://upload-images.jianshu.io/upload_images/1915184-f5ddc977ef3c0338?imageMogr2/auto-orient/strip%7CimageView2/2/w/600/format/webp)

二维码

总结起来就是二维码是将有限的信息转成二进制，并且表现为黑白矩阵图，与一维的条形码相比，二维码拥有更好的容错能力。

### ZXing

[`zxing`](https://github.com/zxing/zxing)是谷歌开源的让开发者更方便使用摄像头的库, 但是因为zxing的功能太强大了，包含了很多我们用不上的功能，所以一般都会抽取其中的扫码功能单独使用; 

而 zxing-android-embedded 就是基于 ZXing 的 Android 二维码解码库，专门用于显示/扫描二维码的开源库: https://github.com/journeyapps/zxing-android-embedded

## 基础使用

### 配置

```groovy
// Config for SDK 24+

repositories {
    mavenCentral()
}

dependencies {
    implementation 'com.journeyapps:zxing-android-embedded:4.3.0'
}
```

如MinSDK低于24，请参考官网；

另外还需要开启硬件加速：

```groovy
    <application android:hardwareAccelerated="true" ... >
    //或只在某Activity里开启
    <activity
       android:name=".QRActivity"
       ...
       android:hardwareAccelerated="true"
       ... />
```

### 扫描二维码

定义扫描参数：

```java
ScanOptions options = new ScanOptions();
options.setDesiredBarcodeFormats(ScanOptions.ONE_D_CODE_TYPES);// 扫码的类型,可选：一维码，二维码，一/二维码
options.setPrompt("Scan a barcode");// 设置提示语
options.setCameraId(0);  // 选择摄像头,可使用前置或者后置
options.setBeepEnabled(false); // 是否开启声音,扫完码之后会"哔"的一声
options.setBarcodeImageEnabled(true);// 扫完码之后生成二维码的图片

```

启动扫描和获取结果：

```java
// Register the launcher and result handler
private final ActivityResultLauncher<ScanOptions> barcodeLauncher = registerForActivityResult(new ScanContract(),
        result -> {
            if(result.getContents() == null) {
                Toast.makeText(MyActivity.this, "Cancelled", Toast.LENGTH_LONG).show();
            } else {
                Toast.makeText(MyActivity.this, "Scanned: " + result.getContents(), Toast.LENGTH_LONG).show();
            }
        });

// Launch
public void onButtonClick(View view) {
    barcodeLauncher.launch(new ScanOptions());
}
```

在最新版本 `startActivityForResult`已不推荐，虽然还能使用, 使用方法如下：

在Activity或Fragment调用：

```java
new IntentIntegrator(this)
        .setDesiredBarcodeFormats(IntentIntegrator.ONE_D_CODE_TYPES)
        .setPrompt("请对准二维码")
        .setCameraId(0)
        .setBeepEnabled(false)
        .setBarcodeImageEnabled(true)
        .initiateScan();// 初始化扫码
```

扫完码之后会在onActivityResult方法中回调结果:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    IntentResult result = IntentIntegrator.parseActivityResult(requestCode, resultCode, data);
    if(result != null) {
        if(result.getContents() == null) {
            Toast.makeText(this, "Cancelled", Toast.LENGTH_LONG).show();
        } else {
            Toast.makeText(this, "Scanned: " + result.getContents(), Toast.LENGTH_LONG).show();
        }
    } else {
        super.onActivityResult(requestCode, resultCode, data);
    }
}
```

> `注`: 启动扫描是需要Camera权限的， 因此在启动之前，需要动态申请Camera权限；

更改默认扫描界面的方向：

```groovy
<activity
		android:name="com.journeyapps.barcodescanner.CaptureActivity"
		android:screenOrientation="fullSensor"
		tools:replace="screenOrientation" />
```

```java
ScanOptions options = new ScanOptions();
options.setOrientationLocked(false);
barcodeLauncher.launch(options);
```



### 显示二维码

该库主要功能并不在显示二维码上，但也可以显示一些简单二维码；

```java
try {
  BarcodeEncoder barcodeEncoder = new BarcodeEncoder();
  Bitmap bitmap = barcodeEncoder.encodeBitmap("content", BarcodeFormat.QR_CODE, 400, 400);
  ImageView imageViewQrCode = (ImageView) findViewById(R.id.qrCode);
  imageViewQrCode.setImageBitmap(bitmap);
} catch(Exception e) {

}
```

或：

```java
MultiFormatWriter multiFormatWriter = new MultiFormatWriter();
        try {
            BitMatrix bitMatrix = multiFormatWriter.encode(text, BarcodeFormat.QR_CODE, 200, 200);
            BarcodeEncoder barcodeEncoder = new BarcodeEncoder();
            Bitmap bitmap = barcodeEncoder.createBitmap(bitMatrix);
        } catch (Exception e) {
            Log.e(TAG, "makeQR Error : ", e);
        }
```

也可以通过 `BarcodeEncoder`的`setBackgroundColor` 和`setForegroundColor`方法来设置二维码的背景和内容的颜色，默认是白底黑内容；

## 自定义扫描二维码的界面

默认情况下，扫描界面的Activity是`com.journeyapps.barcodescanner.CaptureActivity`, 但我们可以通过`IntentIntegrator.setCaptureActivity(MyScanActivity.class)`来设置自己定义的Activity；

从其默认的Activity可以看到：

1. `CaptureManager`是用来拉起扫码和处理扫码结果的类；
2. `DecoratedBarcodeView`则是一个显示扫码界面的自定义View；

而 `DecoratedBarcodeView` 对应的布局是： `zxing_barcode_scanner.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <com.journeyapps.barcodescanner.BarcodeView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/zxing_barcode_surface"/>

    <com.journeyapps.barcodescanner.ViewfinderView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/zxing_viewfinder_view"/>

    <TextView android:id="@+id/zxing_status_view"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_gravity="bottom|center_horizontal"
              android:background="@color/zxing_transparent"
              android:text="@string/zxing_msg_default_status"
              android:textColor="@color/zxing_status_text"/>
</merge>

```

所以我们需要只需要在我们Activity的布局里， 包含`DecoratedBarcodeView`View就好，其它部分就由我们定义了；而``对应的布局，我们也可以进行

```xml
	...
	<com.journeyapps.barcodescanner.DecoratedBarcodeView
                    android:id="@+id/zxing_barcode_scanner"
                    android:layout_width="match_parent"
                    android:layout_height="@dimen/qr_scanner_size"
                    android:layout_centerInParent="true"
                    app:zxing_scanner_layout="@layout/qr_scan_view" />
    ...
```

qr_scan_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <com.journeyapps.barcodescanner.BarcodeView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/zxing_barcode_surface"
        app:zxing_framing_rect_width="@dimen/qr_scanner_size"
        app:zxing_framing_rect_height="@dimen/qr_scanner_size"/>

    <com.journeyapps.barcodescanner.ViewfinderView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/zxing_viewfinder_view"
        app:zxing_possible_result_points="@color/zxing_custom_possible_result_points"
        app:zxing_result_view="@color/zxing_custom_result_view"
        app:zxing_viewfinder_laser="@color/zxing_custom_viewfinder_laser"
        app:zxing_viewfinder_mask="@color/zxing_custom_viewfinder_mask"/>
</merge>
```

而自定义的Activity部分：

```java
public class MyScanActivity extends Activity {
    private CaptureManager capture;
    private DecoratedBarcodeView barcodeScannerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_custom_capture);// 自定义布局

        barcodeScannerView = (DecoratedBarcodeView) findViewById(R.id.zxing_barcode_scanner);

        capture = new CaptureManager(this, barcodeScannerView);
        capture.initializeFromIntent(getIntent(), savedInstanceState);
        capture.decode();
    }

    @Override
    protected void onResume() {
        super.onResume();
        capture.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        capture.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        capture.onDestroy();
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        capture.onSaveInstanceState(outState);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String permissions[], @NonNull int[] grantResults) {
        capture.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        return barcodeScannerView.onKeyDown(keyCode, event) || super.onKeyDown(keyCode, event);
    }
}
```

## 相关技巧

在扫描二维码过程中， 它只扫描到 一个结果时，就会结束Activity， 但有时这个扫描到的内容不是我们想要内容；那么在处理扫描结果时， 可再次启动扫描二维码界面，同时启动后台服务去验证之前扫描结果是否为我们所要的，如果是，则可结束扫描二维码的Activity(我们自定义的)；或自定义其开源库的行为即可；