## [Kotlin笔记] 与Java交互操作

#### Kotlin 调用 java 的场景

Kotlin的

- 可直接调用Java的类；
- 能自动识别Java类的Getter/Setter方法；
- Java类型映射(具体略)， 映射只发生在编译期间，运行时表示保持不变；

图



##### 使用java泛型

- T! 表示 “T 或者 T?” //一般JAVA类映射过来才会使用！

- (Mutable)Collection<T>! 表示“可以可变或不可变，可空或不可空的T的java集合”；

- Array<(out) T>! 表示“可以可变或不可变，可空或不可空的T(或T的子类型)的java集合”；

- Java 的通配符转换成类型投影

  Foo<? extends Bar> 转换成 Foo<out Bar!>!

  Foo<? super Bar> 转换成 Foo<in Bar!>!

- Java的原始类型转换成星投影

  List 转换成 List<*>!, 即 List<out Any?>!

图

##### 数组

与Java不同，Kotlin中的数组是不型变的，这意味着 kotlin 不允许我们把一个 Array<String> 赋值给一个 Array<Any>。 kotlin 也禁止我们把一个子类的数组当做超类的数组传递给kotlin的方法。但对于 Java方法，这是允许的(通过 Array<out String>! 这种形式的平台类型)



另外每种原生类型的数组都有一个特化的类（IntArray, DoubleArray, CharArray...）来处理这种情况，它们与Array类无关，并且会编译成 Java 原生类型数组以获最佳性能；



##### SAM(Single Abstract Method) 转换

SAM的适用对象为： 在java中， 接口里只有一个方法的, 而其它类定义该接口对象作为 唯一参数的方法；

```kotlin
val runnable = Runnable { print("Hello") }

val executor = ThreadPoolExecutor（）
executor?.execute { print("Hello") }
```



注：

- SAM 转换只适用于接口，而不适用于抽象类，即使这些抽象类也只有一个抽象方法；
- 因为 Kotlin具有合适的函数类型， 所以不需要将函数自动转换为 Kotlin 接口的实现，所以Kotlin没SAM转换的说法；



#### Java 调用Kotlin

##### 包级函数

一般用法：

```kotlin
// app.kt 
package org.example
class Util
fun getTime() {/*.....*/}

// Java 调用 上面Kotlin
//调用类时
new org.example.Util(); 
//调用函数时
org.example.Appkt.getTime();
```

其中关于Appkt, 就是 app.kt 去掉"."后，把第一个大写；

也可以使用@file来把Appkt进行重命名：

```kotlin
// app.kt 
@file:JvmName(DemoUtils)
package org.example
class Util
fun getTime() {/*.....*/}

// Java 调用 上面Kotlin
//调用类时
new org.example.Util(); 
//调用函数时
org.example.DemoUtils.getTime();
```

##### @JvmField

使Kotlin编译器不再对该字段生成`getter/setter`并将其作为公开字段（public）

该注解告诉JVM可以在java中访问公共属性一样使用"."来访问；**只修饰属性**；

```kotlin
class User(id: String) {
	@JvmField val ID = id
}

//Java 调用
public String getId(User user) {
	return user.ID;
}
```

@JvmField也可以使用在Companion object的属性或方法中，相当转为成java中 public static final 字段；

```kotlin
class Key(val value: Int) {
    companion object {
        @JvmField
        val COMPARATOR: Comparator<Key> = compareBy<Key> {it.value}
    }
}

//Java
key.COMPARATOR.compareBy(key1, key2)
```

##### @JvmStatic

只用于 object 类或者 companion 类中；

对函数使用该注解，kotlin编译器将生成`另一个静态方法`，
对属性使用该注解，kotlin编译器将生成`其他的setter和getter方法`
这个注解的作用其实就是消除Java调用Kotlin的`companion object`对象时不能直接调用其静态方法和属性的问题.

```kotlin
class C {
    companion object {
        @JvmStatic fun foo() {}
        fun bar() {}
    }
}

//Java
C.foo(); // 运行良好
C.bar(); // 错误：不是静态方法
C.Companion.foo(); // 实例方法仍然保留
C.Companion.bar(); // 唯一奏效的方法
```
