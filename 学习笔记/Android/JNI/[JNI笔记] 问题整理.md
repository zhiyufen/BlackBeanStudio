## [JNI笔记] 问题整理

##### 1，cpp 文件内容在AndroidStudio里报红，但编译没问题。

**问题**：Android studio 提示该cpp文件不在项目，里面的内容都报红，但CMakeLists.txt里已经包含：

```shell
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        src/main/cpp/native-lib.cpp)
```

**解决方案**： 后面直接把工程目录XXX\app\.externalNativeBuild\cmake下的debug和release两个目录删掉后，然后同步及清理工程后就可以build新加的cpp文件了