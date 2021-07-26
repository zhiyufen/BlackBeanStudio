##  如何分析NDK的Crash

通常是项目中用到了 NDK 引发某类致命的错误导致闪退。因为 NDK 是使用 C/C++ 来进行开发，熟悉 C/C++ 的程序员都知道，指针和内存管理是最重要也是最容易出问题的地方，稍有不慎就会遇到诸如内存地址访问错误、使用野指针、内存泄露、堆栈溢出、初始化错误、类型转换错误、数字除0等常见的问题，导致最后都是同一个结果：程序崩溃。它不会像在 Java 层产生的异常时弹出“xxx程序无响应，是否立即关闭”之类的提示框。当发生 NDK 错误后，logcat 打印出来的那堆日志根据看不懂，更别想从日志当中定位错误的根源。

当 NDK 程序在发生 Crash 时，它会在路径 */data/tombstones/* 下产生导致程序 Crash 的文件 tombstone_xx。并且 Google 还在 NDK 包中为我们提供了一系列的调试工具，例如 **addr2line**、**objdump**、**ndk-stack**。

##### Linux 信号机制

----

在介绍 Tombstone 之前，我们首先补充一个 Linux 信号机制的知识。

信号机制是 Linux 进程间通信的一种重要方式，Linux 信号一方面用于正常的进程间通信和同步，如任务控制(SIGINT, SIGTSTP,SIGKILL, SIGCONT，……)；另一方面，它还负责监控系统异常及中断。 当应用程序运行异常时， Linux 内核将产生错误信号并通知当前进程。 当前进程在接收到该错误信号后，可以有三种不同的处理方式。

- 忽略该信号。
- 捕捉该信号并执行对应的信号处理函数(signal handler)。
- 执行该信号的缺省操作(如 SIGSEGV， 其缺省操作是终止进程)。

当 Linux 应用程序在执行时发生严重错误，一般会导致程序 crash。其中，Linux 专门提供了一类 crash 信号，在程序接收到此类信号时，缺省操作是将 crash 的现场信息记录到 core 文件，然后终止进程。

Crash 信号列表：

| Signal  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| SIGSEGV | Invalid memory reference.                                    |
| SIGBUS  | Access to an undefined portion of a memory object.           |
| SIGFPE  | Arithmetic operation error, like divide by zero.             |
| SIGILL  | Illegal instruction, like execute garbage or a privileged instruction |
| SIGSYS  | Bad system call.                                             |
| SIGXCPU | CPU time limit exceeded.                                     |
| SIGXFSZ | File size limit exceeded.                                    |



#### Tombstone

Android Native 程序本质上就是一个 Linux 程序，因此当它在执行时发生严重错误，也会导致程序 crash，然后产生一个记录 crash 的现场信息的文件，而这个文件在 Android 系统中就是 tombstone 文件。tombstone 文件位于路径 */data/tombstones/* 下；

而三*星手机是在 log/dumpstate_app_native-xxxxxx\FS\data\tombstones下的；

现在我们做一下ndk Crash出来，原来的代码如下：

```c++
const char ivValues[] =  {    //16bit 增强的iv向量，用于AES算法增强
        33, 32, 25, 25, 35, 27, 55, 12, 15,32,
        23, 45, 26, 32, 5, 16
};
//把ivValue数组值返回给Java
jbyteArray getIvValue(JNIEnv *env, jobject obj) {
    jbyteArray keys = env->NewByteArray(sizeof(ivValues));
    jboolean *isCopy = 0;
    jbyte *bytes = env->GetByteArrayElements(keys, isCopy);

    for (int i = 0; i < sizeof(ivValues); ++i) {
        bytes[i] = ivValues[i];
    }
    env->SetByteArrayRegion(keys, 0, sizeof(ivValues), bytes);
    env->ReleaseByteArrayElements(keys, bytes, 0);
    return keys;
}
```

那么我们做一下数组越界访问内存：

```
// make a crash
for (int i = 0; i < sizeof(ivValues) + 10000; ++i) {
        bytes[i] = ivValues[i];
    }
```

我们从tombstones文件中拿到的信息如下：

```
*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
Build fingerprint: 'samsung/c7proltezc/c7proltechn:8.0.0/R16NW/C7010ZCT3CSC1:eng/test-keys'
Revision: '4'
ABI: 'arm64'
pid: 29624, tid: 29624, name: hiyufen.jni.app  >>> com.zhiyufen.jni.app <<<
signal 11 (SIGSEGV), code 2 (SEGV_ACCERR), fault addr 0x72d148d000
    x0   00000072d29d0560  x1   0000007fda0e25e0  x2   0000000000000000  x3   00000072ed50a91e
    x4   0000007fda0e25b8  x5   00000072d29d0570  x6   0000000000000000  x7   0000000000000000
    x8   00000072d148d000  x9   0000000000007be9  x10  0000000000000000  x11  0000000000000002
    x12  00000072ed60d648  x13  0000000000000000  x14  00000072ed4e8120  x15  0000000000000000
    x16  00000072ed5c1600  x17  00000072f1f71930  x18  0000000000000020  x19  00000072ed6c4a00
    x20  00000072d1469bac  x21  00000072ed6c4a00  x22  0000007fda0e299c  x23  00000072d3032203
    x24  0000000000000004  x25  00000072ed6c4a98  x26  0000000000000000  x27  0000000000000009
    x28  0000000000000001  x29  0000007fda0e26f0  x30  00000072d1469be8
    sp   0000007fda0e26c0  pc   00000072d1469c14  pstate 0000000080000000
    v0   0000007fda0e4bc00000000000000000  v1   00000000000000002e74736973726570
    v2   000000000000000000006e6576657473  v3   00000000000000000000000000000000
    v4   00000000000000008020000000000000  v5   00000000000000004000000000000000
    v6   00000000000000000000000000000000  v7   00000000000000008020080280200802
    v8   00000000000000000000000000000000  v9   00000000000000000000000000000000
    v10  00000000000000000000000000000000  v11  00000000000000000000000000000000
    v12  00000000000000000000000000000000  v13  00000000000000000000000000000000
    v14  00000000000000000000000000000000  v15  00000000000000000000000000000000
    v16  40100401401004014010040140100401  v17  400040004000000040404000a800a800
    v18  40000000400000004000000000000000  v19  000000000000000000000000ebad8083
    v20  000000000000000000000000ebad8084  v21  000000000000000000000000ebad8085
    v22  000000000000000000000000ebad8086  v23  000000000000000000000000ebad8087
    v24  000000000000000000000000ebad8088  v25  000000000000000000000000ebad8089
    v26  000000000000000000000000ebad808a  v27  000000000000000000000000ebad808b
    v28  000000000000000000000000ebad808c  v29  000000000000000000000000ebad808d
    v30  000000000000000000000000ebad808e  v31  000000000000000000000000ebad808f
    fpsr 00000013  fpcr 00000000

backtrace:
    #00 pc 000000000000fc14  /data/app/com.zhiyufen.jni.app-9SCC-3xEOG8zT3fhlSkpLw==/lib/arm64/libnative-lib.so (_Z10getIvValueP7_JNIEnvP8_jobject+104)
    #01 pc 000000000000f09c  /data/app/com.zhiyufen.jni.app-9SCC-3xEOG8zT3fhlSkpLw==/oat/arm64/base.odex (offset 0xf000)

stack:
         0000007fda0e2640  0000000000000000
         0000007fda0e2648  00000072ed6c4a98  [anon:libc_malloc]
         0000007fda0e2650  0000000000000004
         0000007fda0e2658  00000072d3032203  /data/app/com.zhiyufen.jni.app-9SCC-3xEOG8zT3fhlSkpLw==/oat/arm64/base.vdex
         0000007fda0e2660  0000007fda0e299c
```

 我们看到是主线程中发生的signal 11 (SIGSEGV)信号的Crash(pid==tid), 是属于无效的内存引用，说明代码有访问到无效内存地址了；

```
pid: 29624, tid: 29624, name: hiyufen.jni.app  >>> com.zhiyufen.jni.app <<<
signal 11 (SIGSEGV), code 2 (SEGV_ACCERR), fault addr 0x72d148d000
```

这时候我们需要用到addr2line工具

addr2line工具是一个可以将指令的地址和可执行映像转换为文件名、函数名和源代码行数的工具。这在内核执行过程中出现崩溃时，可用于快速定位出出错的位置，进而找出代码的bug。其用法请参考：[addr2line用法](https://www.jianshu.com/p/c2e2b8f8ea0d)

1. 找到addr2line的位置， 或者添加其路径到Path环境变量中，C:\Users\yufen\AppData\Local\Android\Sdk\ndk\20.1.5948944\toolchains\aarch64-linux-android-4.9\prebuilt\windows-x86_64\bin\aarch64-linux-android-addr2line.exe

2. 找到debug下的含符号表的so文件；MyJniApp\app\build\intermediates\merged_native_libs\debug\out\lib\arm64-v8a\libnative-lib.so

3. 那么根据上面backtrace的信息，我们找一下000000000000fc14地址对应的代码在哪里：

   ```
   D:\Learning\MyBook\NDK\dumpstate_app_native-2020-03-26-10-00-39>aarch64-linux-android-addr2line -e libnative-lib.so 000000000000fc14
   D:/Learning/MyBook/NDK/Code/JNI/MyJniApp/app/src/main/cpp/native-lib.cpp:113
   ```

   我可以得到该行代码是在native-lib.cpp文件113行：

   ```
   111    // make fetal
   112    for (int i = 0; i < sizeof(ivValues) +1000000; ++i) {
   113        bytes[i] = ivValues[i]; //访问无效内存的代码；
   114   }
   ```

##### objdump

通过addr2line命令，其实我们已经找到了我们代码中出错的位置，已经可以帮助程序员定位问题所在了。但是，这个方法只能获取代码行数，并没有显示函数信息; 

objdump命令是Linux下的反汇编目标文件或者可执行文件的命令,更多的objdump使用请参考：[objdump命令的使用](https://blog.csdn.net/beyondioi/article/details/7796414)

1. 导出so的函数表信息：

   ```
   D:\Learning\MyBook\NDK\dumpstate_app_native-2020-03-26-10-00-39>aarch64-linux-android-objdump -S -D libnative-lib.so > dump.log
   ```

   会生成一个文件：dump.log

2. 根据下面的backtrace， 我们找到出问题时地址是000000000000fc14；

   ```
   backtrace:
       #00 pc 000000000000fc14  /data/app/com.zhiyufen.jni.app-9SCC-3xEOG8zT3fhlSkpLw==/lib/arm64/libnative-lib.so (_Z10getIvValueP7_JNIEnvP8_jobject+104)
       #01 pc 000000000000f09c  /data/app/com.zhiyufen.jni.app-9SCC-3xEOG8zT3fhlSkpLw==/oat/arm64/base.odex (offset 0xf000)
   ```

   那么我们打开dump.log，直接搜索 fc14 字符串, 可以找到下面的内容：

   ```
   jbyteArray getIvValue(JNIEnv *env, jobject obj) {
       fbac:	d10103ff 	sub	sp, sp, #0x40
       fbb0:	a9037bfd 	stp	x29, x30, [sp,#48]
       fbb4:	9100c3fd 	add	x29, sp, #0x30
       fbb8:	f81f83a0 	stur	x0, [x29,#-8]
       fbbc:	f81f03a1 	stur	x1, [x29,#-16]
       // 分配一个Java的byte数组；
       jbyteArray keys = env->NewByteArray(sizeof(ivValues));
   	
   	//省略部分代码
   	....
   
       // make fetal
       for (int i = 0; i < sizeof(ivValues) +1000000; ++i) {
       fbec:	b90007ff 	str	wzr, [sp,#4]
       fbf0:	b98007e8 	ldrsw	x8, [sp,#4]
       fbf4:	d2884a09 	mov	x9, #0x4250                	// #16976
       fbf8:	f2a001e9 	movk	x9, #0xf, lsl #16
       fbfc:	eb09011f 	cmp	x8, x9
       fc00:	540001c2 	b.cs	fc38 <_Z10getIvValueP7_JNIEnvP8_jobject+0x8c>
       fc04:	900000e8 	adrp	x8, 2b000 <__deregister_frame_info_bases+0x60>
       fc08:	91105d08 	add	x8, x8, #0x417
           bytes[i] = ivValues[i];
       fc0c:	b98007e9 	ldrsw	x9, [sp,#4]
       fc10:	8b090108 	add	x8, x8, x9
       fc14:	3940010a 	ldrb	w10, [x8]  //找到fc14，在这里，
       fc18:	f94007e8 	ldr	x8, [sp,#8]
       fc1c:	b98007e9 	ldrsw	x9, [sp,#4]
       fc20:	8b090108 	add	x8, x8, x9
       fc24:	3900010a 	strb	w10, [x8]
   	
   	//省略部分代码
   	....
   
   000000000000fc7c <JNI_OnLoad>:
   static const JNINativeMethod AESMethod[] = {
           {"getKeyValue", "()[B", (void *)getKeyValue},
           {"getIvValue", "()[B", (void *)getIvValue}
   };
   ```

   找到fc14的位置，找它的所在的函数，就发现它是jbyteArray getIvValue(JNIEnv *env, jobject obj) 函数里的；这里就方便我们看问题了；

##### ndk-stack

如果你觉得上面的方法太麻烦的话，ndk-stack可以帮你减轻操作步聚，直接定位到代码出错的位置。

- 实时分析日志

  去ndk找到 ndk-stack命令的目录，添加其到环境变量中：C:\Users\yufen\AppData\Local\Android\Sdk\ndk\20.1.5948944\prebuilt\windows-x86_64\bin

  ```
  adb logcat | ndk-stack -sym D:\Learning\MyBook\NDK_VIP\Code\JNI\MyJniApp\app\build\intermediates\merged_native_libs\debug\out\lib\arm64-v8a -dump 
  ```

- 获取日志再分析

  ```
  //使用下面保存log或者测试人员共享过来的log文件
  adb logcat > crash.log
  ndk-stack -sym D:\Learning\MyBook\NDK_VIP\Code\JNI\MyJniApp\app\build\intermediates\merged_native_libs\debug\out\lib\arm64-v8a -dump crash.log
  ```

  我本地是ndk版本是20.1.5948944，在window上运行会出现 WindowsError2的错误，根据更新R21就好；