## Dagger2

Dagger2是一个依赖注入框架，主要作用，就是对象的管理，其目的是为了降低程序耦合。

Dagger2采用静态编译的方式编译代码的，会在编译期生成好辅助代码，不会影响运行时性能，这一点非常适合用于移动端。

#### Gradle 配置

```groovy
annotationProcessor 'com.google.dagger:dagger-compiler:2.11' 
implementation 'com.google.dagger:dagger-android:2.11'
implementation 'com.google.dagger:dagger-android-support:2.11'
// if you use the support libraries
annotationProcessor 'com.google.dagger:dagger-android-processor:2.11'
```

#### 基本概念

Dagger 是通过`@Inject`使用具体的某个对象，这个对象呢，是由`@Provides`注解提供，但是呢，这个`@Provides`只能在固定的模块中，也就是`@Module`注解，我们查找的时候，不是直接去找模块，而是去找`@Component`

#### 基本使用

相关类：

```java
public class ObjectA {
}
public class ObjectB {
}
```

1. 创建Moudule及注解实例化对象

   ```
   
   ```

   

2. 



https://www.jianshu.com/p/2cd491f0da01

# Dagger Hilt

