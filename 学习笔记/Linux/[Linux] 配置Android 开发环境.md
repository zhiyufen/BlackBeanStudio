### [Linux] 配置Android 开发环境

### 一,  安装 Java Jdk

1. 请前往[官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 下载， 比如：[jdk-8u231-linux-x64.tar.gz](javascript: void(0))

2. 移文件到 /usr/local: 

   ```
    sudo mv jdk-8u231-linux-x64.tar.gz /usr/local
   ```

3. 打开/usr/local目录， 打开终端解压安装包，没错误提示就是安装成功了：

   ```
   sudo tar -zxvf -i jdk-8u231-linux-x64.tar.gz
   ```

4. 删除安装包：

   ```
   sudo rm jdk-8u231-linux-x64.tar.gz
   ```

5. 环境变量设置
   编辑~/.bashrc 文件： sudo vim ~/.bashrc, 添加下面的内容：

   ```
   #Java
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
   export JRE_HOME=$JAVA_HOME/jre
   export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
   export PATH=$PATH:$JAVA_HOME/bin
   ```

6. 检查是否安装：

   ```
   java -version
   ```

   

### 二， 安装Android Studio

1.  [官网](https://developer.android.google.cn/studio/)下载, 没办法点开官网的话， 可点击[这里下载](http://www.android-studio.org/)；

2.  移文件到 /usr/local ，再解压安装， 相关命令参考上面；

3. 环境变量设置
   编辑~/.bashrc 文件： sudo vim ~/.bashrc, 添加下面的内容：

   ```
   #Android Studio
   export PATH=$PATH:$START_ANDROID
   export ANDROID_HOME=/home/yufenzhi/Android/Sdk
   export PATH=$PATH:$ANDROID_HOME/tolls:$ANDROID_HOME/platform-tools
   #设置Android Studio的启动项为全局，终端输入studio.sh来启动；
   export START_ANDROID=/opt/android-studio/bin
   ```
