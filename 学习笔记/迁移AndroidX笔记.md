## 简介

&emsp;&emsp;在Android9.0（API28）后，Google会逐步放弃android.support.xxx包的升级和维护，转为使用AndroidX；所以我们需要尽快转移使用AndroidX;

## 迁移AndroidX

#### 更改gradle.properties

```xml
//启用 AndroidX
android.useAndroidX=true
//将依赖包也迁移到AndroidX
android.enableJetifier=true
```

#### 迁移
在 AndroidStudio 3.2 或更高版本（截图中 AndroidStudio 为 3.2 版本）中执行操作： Refactor > Migrate to AndroidX;
![Android_1](https://github.com/zhiyufen/BlackBeanStudio/blob/master/%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/AndoridX_1.png)

[AndroidX官方迁移文档](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Ftopic%2Flibraries%2Fsupport-library%2Frefactor%23migrate)

目前只在小项目上尝试迁移，工具迁移后是可以运行，后面遇到问题再补上；
也可以看一下大神记录的：[Android:你好,androidX！再见,android.support](https://www.jianshu.com/p/41de8689615d)
