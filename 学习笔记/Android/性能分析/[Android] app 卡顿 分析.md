## [Android] app 滑动卡顿 分析

#### 1. UI卡顿原理

- Android系统每隔16ms就会重新绘制一次Activity，因此，我们的应用必须在16ms内完成屏幕刷新的全部逻辑操作，每一帧只能停留16ms，否则就会出现掉帧现象(也就是用户看到的卡顿现象)。
- 16ms = 1000/60hz，相当于60fps(每秒帧率)。这是因为人眼与大脑之间的协作无法感知超过60fps的画面更新。因此，如果应用没有在16ms内完成屏幕刷新的全部逻辑操作，就会发生卡顿。
- Android不允许在UI线程中做耗时的操作，否则有可能发生ANR的可能，默认情况下，在Android中Activity的最长执行时间是5秒，BroadcastReceiver的最长执行时间则是10秒，Service前台20s、后台200s未完成启动。如果超过默认最大时长，则会产生ANR。

Android系统每隔16ms就会发出VSYNC信号，触发对UI进行渲染，VSYNC是Vertical Synchronization(垂直同步)的缩写，可以简单的把它认为是一种定时中断。在Android 4.1中开始引入VSYNC机制。为什么是16ms？因为Android设定的刷新率是60FPS(Frame Per Second)，也就是每秒60帧的刷新率，约16ms刷新一次。这就意味着，我们需要在16ms内完成下一次要刷新的界面的相关运算，以便界面刷新更新。举个例子，当运算需要24ms完成时，16ms时就无法正常刷新了，而需要等到32ms时刷新，这就是丢帧了。丢帧越多，给用户的感觉就越卡顿。

正常流畅刷新图示：

![image](https://user-gold-cdn.xitu.io/2019/11/26/16ea83e59a79899a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

卡顿图示：

![image2](https://user-gold-cdn.xitu.io/2019/11/26/16ea83ea26c99105?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

以上内容来自于：https://juejin.im/post/5de336d96fb9a0716d5c0814

[使用 CPU Profiler 检查 CPU 活动](https://developer.android.com/studio/profile/cpu-profiler#configurations)