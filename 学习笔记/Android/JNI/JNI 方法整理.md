### JNI 方法整理

调用JNI方法需要注意： 必须保证其参数为非空的，也就是调用时需要我们自己来检查空判断；

[TOC]

#### Interface Function Table

所有的方法都通过 JNIEnv 结构体进行偏移来访问的：

```c++
typedef const struct JNINativeInterface *JNIEnv;
```

方法列表如下(略)：

#### 版本信息



参考文章： https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/functions.html