#### Q: 当出现andorid studio 同步不成功时，错误信息提示如下：
```
e: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException:
unable to find valid certification path to requested target 
```
这时候是因为我们证书过期或无导致无法下载的；

#### A: 证书无效，那么我们就找有效的证书来使用；

1， 首先在中国，我们可以替换为国内的源代码，替换如下：
```
   repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
        google()
   }
```

这时候也许还是出现一样的问题；

2， 那我们把 maven.aliyun.com 网站的证书导入到 Android Studio；

a, 导出网站证书： 在Chrome浏览上打开 maven.aliyun.com 网站 --> 右键 --> 检查 --> Security --> 点击　＂View certiticate＂　--> 点击　＂详细路径＂　--> 复制文件　--> 按步骤导出到证书到某个文件; 

b, 导入证书到Android Studio: 打开 Android Studio --> Files --> Settings --> Server certiticates --> 点击 "+" 进行导入刚刚的证书即可；

*******

#### Q: Task :main:compileDebugJavaWithJavac FAILED

由于 butterknife 需要 Java 8, 因此在common 模块配置下面的配置：Java8
```
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
```

但编译时， 报出下面的问题：
```
> Task :main:compileDebugJavaWithJavac FAILED
javacTask: 源发行版 1.8 需要目标发行版 1.8

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':main:compileDebugJavaWithJavac'.
> Compilation failed; see the compiler error output for details.
```

#### A: 在其它模块也配置Java8 即可，
