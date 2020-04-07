### [Linux] 配置Android 开发环境

### 一,  安装 Java Jdk

1. 请前往[官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 下载， 比如：[jdk-8u231-linux-x64.tar.gz](javascript: void(0))

2. 移文件到 /usr/local: 

   ```shell
    sudo mv jdk-8u231-linux-x64.tar.gz /usr/local
   ```

3. 打开/usr/local目录， 打开终端解压安装包，没错误提示就是安装成功了：

   ```shell
   sudo tar -zxvf -i jdk-8u231-linux-x64.tar.gz
   ```

4. 删除安装包：

   ```shell
   sudo rm jdk-8u231-linux-x64.tar.gz
   ```

5. 环境变量设置
   编辑~/.bashrc 文件： sudo vim ~/.bashrc, 添加下面的内容：

   ```shell
   #Java
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
   export JRE_HOME=$JAVA_HOME/jre
   export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
   export PATH=$PATH:$JAVA_HOME/bin
   ```

6. 检查是否安装：

   ```shell
   java -version
   ```

   

### 二， 安装Android Studio

1.  [官网](https://developer.android.google.cn/studio/)下载, 没办法点开官网的话， 可点击[这里下载](http://www.android-studio.org/)；

2.  移文件到 /usr/local ，再解压安装， 相关命令参考上面；

3. 环境变量设置
   编辑~/.bashrc 文件： sudo vim ~/.bashrc, 添加下面的内容：

   ```shell
   #Android Studio
   export PATH=$PATH:$START_ANDROID
   export ANDROID_HOME=/home/yufenzhi/Android/Sdk
   export PATH=$PATH:$ANDROID_HOME/tolls:$ANDROID_HOME/platform-tools
   #设置Android Studio的启动项为全局，终端输入studio.sh来启动；
   export START_ANDROID=/opt/android-studio/bin
   ```

### 三， 安装 Typora 

typora是一个写文档非常好用的软件，所以当然少不了它啦；

终端安装命令如下：

```shell
# or use
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -

# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update

# install typora
sudo apt-get install typora
```

### 四，安装git

安装git

```shell
sudo apt-get install git
```

配置git，设置名字和邮箱地址

```shell
git config --global user.name "Tom"
git config --global user.email "abc@sina.com"
```

我们要用github等工具，需要生成 RSA 密钥：

```shell
ssh-keygen -t rsa -C "abc@sina.com"
```

更多详细可参考 [生成SSH秘钥连接github（详细教程）](https://blog.csdn.net/lucky__yang/article/details/80148420)

### 五，安装 Understand 软件

下载地址： [官网](https://scitools.com/download/all-builds/) 或直接点击下载 [ Understand-5.1.1010-Linux-64bit.tgz ](http://builds.scitools.com/all_builds/b1010/Understand/Understand-5.1.1010-Linux-64bit.tgz)

下载后，可参选官网文档进行安装：https://scitools.com/documents/unix_install.php

1. 复制到 /opt/目录； 

2. 进行解压安装：

   ```shell
   tar -xvzf Understand-5.1.1010-Linux-64bit.tgz
   ```

   

3. 添加环境变量：

   ```
   #Understand
   export STIHOME=/opt/scitools
   export PATH=$PATH:$STIHOME/bin/linux64/
   ```

   

4. 运行：./understand

5. 破解: 

   点击中间的命令框Add Permanent Liscense，再点击Add Eval or SDL (RegCode)

    输入证书CODE(32/64 都可用)：09E58CD1FB79

### 六，安装 Kget 下载管理器

下载及安装：https://linuxappfinder.com/package/kget

### 七, 安装 sublime text

官网下载： https://www.sublimetext.com/3

### 八, 安装 PyCharm
下载地址： https://www.jetbrains.com/pycharm/download/download-thanks.html?platform=linux&code=PCC
安装官方文档： https://www.jetbrains.com/help/pycharm/installation-guide.html#

打开：
```shell
cd /opt/pycharm-*/bin
sh pycharm.sh
```
