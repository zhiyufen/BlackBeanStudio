## Android Studio导入系统源码

Android Studio导入整个系统源码需要对源码进行编译，生成AS的项目配置文件。

#### 生成AS的项目配置文件

如果你整编过源码，查看out/host/linux-x86/framework/idegen.jar是否存在，如果不存在，进入源码根目录执行如下的命令：

```shell
source build/envsetup.sh
lunch [选择整编时选择的参数或者数字]
mmm development/tools/idegen/
```

如果没整编过源码，可以直接执行如下命令单编idegen模块：

```shell
source build/ensetup.sh
make idegen
```

idegen模块编译成功后，会在 out/host/linux-x86/framework目录下生成idegen.jar，执行如下命令：

```shell
sudo development/tools/idegen/idegen.sh
```

这时会在源码根目录生成android.iml 和 android.ipr 两个文件，这两个文件一般是只读模式，这里建议改成可读可写，否则，在更改一些项目配置的时候可能会出现无法保存的情况。

```shell
sudo chmod 777 android.iml
sudo chmod 777 android.ipr
```

#### 配置AS的项目配置文件

由于要将所有源码导入AS会导致第一次加载很慢，可以在android.iml中修改excludeFolder配置，将不需要看的源码排除掉。等源码项目加载完成后，还可以通过AS对Exclude的Module进行调整。如果你的电脑的性能很好，可以不用进行配置。
在android.iml中搜索excludeFolder，在下面加入这些配置。

```xml
<excludeFolder url="file://$MODULE_DIR$/bionic" />
<excludeFolder url="file://$MODULE_DIR$/bootable" />
<excludeFolder url="file://$MODULE_DIR$/build" />
<excludeFolder url="file://$MODULE_DIR$/cts" />
<excludeFolder url="file://$MODULE_DIR$/dalvik" />
<excludeFolder url="file://$MODULE_DIR$/developers" />
<excludeFolder url="file://$MODULE_DIR$/development" />
<excludeFolder url="file://$MODULE_DIR$/device" />
<excludeFolder url="file://$MODULE_DIR$/docs" />
<excludeFolder url="file://$MODULE_DIR$/external" />
<excludeFolder url="file://$MODULE_DIR$/hardware" />
<excludeFolder url="file://$MODULE_DIR$/kernel" />
<excludeFolder url="file://$MODULE_DIR$/out" />
<excludeFolder url="file://$MODULE_DIR$/pdk" />
<excludeFolder url="file://$MODULE_DIR$/platform_testing" />
<excludeFolder url="file://$MODULE_DIR$/prebuilts" />
<excludeFolder url="file://$MODULE_DIR$/sdk" />
<excludeFolder url="file://$MODULE_DIR$/system" />
<excludeFolder url="file://$MODULE_DIR$/test" />
<excludeFolder url="file://$MODULE_DIR$/toolchain" />
<excludeFolder url="file://$MODULE_DIR$/tools" />
<excludeFolder url="file://$MODULE_DIR$/.repo" />
```

#### **导入系统源代码到AS中**

在AS安装目录的bin目录下，打开studio64.vmoptions文件，根据自己电脑的实际情况进行设置，这里修改为如下数值：

```
-Xms1024m
-Xmx1024m
```

如果你是在VirtualBox中下载的系统源码，那么将VirtualBox中的系统源码拷贝到共享文件夹中，这样源码就会自动到Windows或者Mac上，
通过AS的Open an existing Android Studio project选项选择android.ipr 就可以导入源码，这里我用了大概7分钟就导入完毕。导入后工程目录切换为Project选项就可以查看源码。
![img](https://s2.ax1x.com/2019/05/27/VZW21s.png)

#### **配置项目的JDK、SDK**

由于我们下载的是9.0的AOSP源码，SDK版本也应该对应为API 28，如果没有就去SDK Manager下载即可。
点击File -> Project Structure–>SDKs配置项目的JDK、SDK。
创建一个新的JDK,这里取名为1.8(No Libraries)，删除其中classpath标签页下面的所有jar文件。
![img](https://s2.ax1x.com/2019/05/27/VZWytg.png)

接着设置将Android SDK的Java SDK设置为1.8(No Libraries)，这样Android源码使用的Java就是Android源码中的。
![img](https://s2.ax1x.com/2019/05/27/VZWgpj.png)

确保的项目的SDK为源码对应的SDK。
![img](https://s2.ax1x.com/2019/05/27/VZWRcn.png)

#### **Exclude不需要的代码目录**

File -> Project Structure -> Modules中可以通过Excluded来筛选代码目录，比如我们选择bionic目录，点击Excluded，bionic目录会变为橙色，bionic字段会出现在右侧视图中，说明该目录已经被Excluded掉，通俗来讲就是被排除在工程之外。如果不希望bionic目录被Excluded掉，再次点击Excluded，bionic目录会变为灰色。

![img](https://s2.ax1x.com/2019/05/27/VZWWXq.png)

本文来自于： http://liuwangshu.cn/framework/aosp/4-import-aosp.html