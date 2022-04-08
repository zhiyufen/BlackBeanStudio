## Kotlin 注解

[TOC]

在Kotlin中注解核心概念和Java一样，**注解就是为了给代码提供元数据**。并且注解是不直接影响代码的执行。**一个注解允许你把额外的元数据关联到一个声明上，然后元数据就可以被某种方式(比如运行时反射方式以及一些源代码工具)访问**。

在Kotlin中的声明注解的方式和Java稍微不一样，在Java中主要是通过 **@interface**关键字来声明，而在Kotlin中只需要通过 **annotation class** 来声明, 需要注意的是在Kotlin中编译器禁止为注解类指定类主体,因为在Kotlin中注解只是用来定义关联的声明和表达式的元数据的结构。

声明：

```kotlin
package com.mikyou.annotation
//和一般的声明很类似，只是在class前面加上了annotation修饰符
annotation class TestAnnotation(val value: String)
```

#### @Deprecated

@Deprecated注解在原来的Java基础增强了一个**ReplaceWith**功能. 可以直接在使用了老的API时，编译器可以根据**ReplaceWith**中的新API，自动替换成新的API。这一点在Java中是做不到的，你只能点击进入这个API查看源码来正确使用新的API。

```kotlin
//@Deprecated注解比Java多了ReplaceWith功能, 这样当你在调用remove方法，编译器会报错。使用代码提示会自动IntelliJ IDEA不仅会提示使用哪个函数提示替代它，而且会快速自动修正。
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"), level = DeprecationLevel.ERROR)//定义的级别是ERROR级别的，这样当你在调用remove方法，编译器会报错。
@kotlin.internal.InlineOnly
public inline fun <T> MutableList<T>.remove(index: Int): T = removeAt(index)
```

### 元注解

Kotlin中的元注解类定义于`kotlin.annotation`包中，主要有: **@Target**、**@Retention**、**@Repeatable**、**@MustBeDocumented** 4种元注解相比Java中5种元注解: **@Target**、**@Retention**、**@Repeatable**、**@Documented**、**@Inherited**少了 **@Inherited**元注解。

#### @Target元注解

也就是这个标签作用于哪些代码中目标对象，可以同时指定多个作用的目标对象。

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)//可以给标签自己贴标签
@MustBeDocumented
//注解类构造器参数是个vararg不定参数修饰符，所以可以同时指定多个作用的目标对象
public annotation class Target(vararg val allowedTargets: AnnotationTarget)
```

@Target注解中可以同时指定一个或多个目标对象: **AnnotationTarget**枚举类

```kotlin
public enum class AnnotationTarget {
    CLASS, //表示作用对象有类、接口、object对象表达式、注解类
    ANNOTATION_CLASS,//表示作用对象只有注解类
    TYPE_PARAMETER,//表示作用对象是泛型类型参数(暂时还不支持)
    PROPERTY,//表示作用对象是属性
    FIELD,//表示作用对象是字段，包括属性的幕后字段
    LOCAL_VARIABLE,//表示作用对象是局部变量
    VALUE_PARAMETER,//表示作用对象是函数或构造函数的参数
    CONSTRUCTOR,//表示作用对象是构造函数，主构造函数或次构造函数
    FUNCTION,//表示作用对象是函数，不包括构造函数
    PROPERTY_GETTER,//表示作用对象是属性的getter函数
    PROPERTY_SETTER,//表示作用对象是属性的setter函数
    TYPE,//表示作用对象是一个类型，比如类、接口、枚举
    EXPRESSION,//表示作用对象是一个表达式
    FILE,//表示作用对象是一个File
    @SinceKotlin("1.1")
    TYPEALIAS//表示作用对象是一个类型别名
}
```

#### @Retention元注解

应用于一个注解上表示该注解保留存活时间，不管是Java还是Kotlin一般都有三种时期: **源代码时期(SOURCE)**、**编译时期(BINARY)**、**运行时期(RUNTIME)**。

```java
@Target(AnnotationTarget.ANNOTATION_CLASS)//目标对象是注解类
//接收一个参数，该参数有个默认值，默认是保留在运行时期
public annotation class Retention(val value: AnnotationRetention = AnnotationRetention.RUNTIME)
```

@Retention元注解取值主要来源于**AnnotationRetention**枚举类:

```kotlin
public enum class AnnotationRetention {
    SOURCE,//源代码时期(SOURCE): 注解不会存储在输出class字节码中
    BINARY,//编译时期(BINARY): 注解会存储出class字节码中，但是对反射不可见
    RUNTIME//运行时期(RUNTIME): 注解会存储出class字节码中，也会对反射可见, 默认是RUNTIME
}
```

#### @MustBeDocumented元注解

该注解比较简单主要是为了标注一个注解类作为公共API的一部分，并且可以保证该注解在生成的API文档中存在。

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)//目标对象只能是注解类
public annotation class MustBeDocumented
```

#### Kotlin为啥不需要@Inherited元注解

关于这个问题实际上在Kotlin官网的discuss中就有人提出了这个问题，具体感兴趣的可以去看看:[Inherited annotations and other reflections enchancements](https://link.zhihu.com/?target=https%3A//discuss.kotlinlang.org/t/inherited-annotations-and-other-reflections-enchancements/6209). 这里大概说下原因，我们都知道在Java中，无法找到子类方法是否重写了父类的方法。因此不能继承父类方法的注解。然而Kotlin目前不需要支持这个@Inherited元注解，因为Kotlin可以做到，如果反射提供了`override`标记而且很容易做到。

### 预置注解

在Kotlin中最大的一个特点就是可以和Java做到极高的互操作性，我们知道Kotlin的语法和Java语法还是有很大的不同，要想做到与Java做到很大兼容性可能需要携带一些额外信息，供编译器或者运行时做类似兼容转换。其中注解就起到了很大的作用，在Kotlin内置很多个注解为了解决Java中的调用Kotlin API的一些调用习惯和控制API的调用。它们就是Kotlin中的@Jvm系列的注解；

#### @JvmDefault

```kotlin
@SinceKotlin("1.2")//从Kotlin的1.2版本第一次出现该注解
@RequireKotlin("1.2.40", versionKind = RequireKotlinVersionKind.COMPILER_VERSION)
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY)//目标对象是函数和属性
annotation class JvmDefault
```

在Kotlin中的接口中可以增加非抽象成员，那么该注解就是为非抽象的接口成员生成默认的方法。

使用`-Xjvm-default = enable`，会为每个`@JvmDefault`注解标注的方法生成接口中的默认方法。在此模式下，使用`@JvmDefault`注解现有方法可能会破坏二进制兼容性，因为它将有效地从`DefaultImpls`类中删除该方法。

使用`-Xjvm-default = compatibility`，除了默认接口方法之外，还会生成兼容性访问器。在`DefaultImpls`类中，它通过合成访问器调用默认接口方法。在此模式下，使用`@JvmDefault`注解现有方法是二进制兼容的，但在字节码中会产生更多方法。从接口成员中移除此注解会使在两种模式中的二进制不兼容性发生变化。

##### 使用注解前后反编译java代码对比

未使用@JvmDefault注解:

```kotlin
interface ITeaching {
    fun speak() = println("open the book")
}

class ChineseTeacher : ITeaching

fun main(args: Array<String>) {
    ChineseTeacher().speak()
}
```

反编译成Java代码:

```java
public interface ITeaching {
   void speak();
    //可以看到在接口为speak函数生成一个DefaultImpls静态内部类
   public static final class DefaultImpls {
      public static void speak(ITeaching $this) {
         String var1 = "open the book";
         System.out.println(var1);
      }
   }
}

public final class ChineseTeacher implements ITeaching {
   public void speak() {
       //注意:这里却是直接调用ITeaching中静态内部类DefaultImpls的speak方法。
      ITeaching.DefaultImpls.speak(this);
   }
}

public final class JvmDefaultTestKt {
   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
       //这里会调用ChineseTeacher中的speak方法
      (new ChineseTeacher()).speak();
   }
}
```

在没有添加 **@JvmDefault**注解，Kotlin会自动生成一个叫做 DefaultImpl静态内部类，用于保存静态方法的默认实现，并使用自身接收器类型来模拟属于对象的方法。然后，对于扩展该接口的每种类型，如果类型没有实现方法本身，则在编译时，Kotlin将通过调用将方法连接到默认实现。

这样一来确实带来一个很大好处就是在JDK1.8之前的版本JVM上提供了在接口上也能定义具体的实现方法功能。但是这样也存在一些问题：

**第一问题:** 比如它和现在的Java的处理方式不兼容，这样会导致互操作性极度下降。我们甚至可以在Java中直接去调用自动生成的`DefaultImpls`,类似这样的调用`ITeaching.DefaultImpls.speak(new ChineseTeacher());`,这样内部的细节居然也能暴露给外部，这样更会调用者一脸懵逼。

**第二问题:** Java 8中存在默认方法的主要原因之一是能够向接口添加方法而无需侵入每个子类。 然而Kotlin实现不支持这个原因是必须在每个具体类型上生成默认调用。 向接口添加新方法导致必须重新编译每个实现者。

基于上述问题,Kotlin推出了 **@JvmDefault**注解

使用@JvmDefault注解:

```kotlin
//注意: 可能一开始使用该注解会报错，需要在gradle中配置jvm参数:-jvm-target=1.8 -Xjvm-default=enable
interface ITeaching {
    @JvmDefault
    fun speak() = println("open the book")
}

class ChineseTeacher : ITeaching

fun main(args: Array<String>) {
    ChineseTeacher().speak()
}
```

反编译成Java代码:

```java
public interface ITeaching {
    //添加注解后外层的静态内部类被消除
   @JvmDefault
   default void speak() {
      String var1 = "open the book";
      System.out.println(var1);
   }
}

//内部并没有类似上面的speak方法调用的桥接委托
public final class ChineseTeacher implements ITeaching {
}

public final class JvmDefaultTestKt {
   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      (new ChineseTeacher()).speak();
   }
}
```

#### @JvmField

```
@Target(AnnotationTarget.FIELD)//作用对象是字段，包括属性的幕后字段
@Retention(AnnotationRetention.BINARY)//注解保留期是源码阶段
@MustBeDocumented
public actual annotation class JvmField
```

应用于一个字段，把这个属性暴露成一个没有访问器的公有Java字段；以及Companion Object对象中。

场景一：
在Kotlin中默认情况下，Kotlin类不会公开字段而是会公开属性.Kotlin会为属性的提供幕后字段，这些属性将会以字段形式存储它的值。一起来看个例子：

```kotlin
//Person类中定义一个age属性，age属性默认是public公开的，但是反编译成Java代码，你就会看到它的幕后字段了。
class Person {
    var age = 18
        set(value) {
            if (value > 0) field = value
        }
}
```

反编译成Java代码:

```java
public final class Person {
   private int age = 18;//这个就是Person类中的幕后字段，可以看到age字段是private私有的。

  //外部访问通过setter和getter访问器来操作。由于Kotlin自动生成setter、getter访问器，所以外部可以直接类似公开属性操作，
  //实际上内部还是通过setter、getter访问器来实现
   public final int getAge() {
      return this.age;
   }

   public final void setAge(int value) {
      if (value > 0) {
         this.age = value;
      }
   }
}
```

但是如果在Kotlin需要生成一个公开的字段怎么实现呢？那就要借助`@JvmField`注解了，它会自动将该字段的setter、getter访问器消除掉，并且把这个字段修改为public:

```kotlin
class Person {
    @JvmField
    var age = 18
}
```

反编译成的Java代码

```java
public final class Person {
   @JvmField
   public int age = 18;//消除了setter、getter访问器，并且age字段为public公开
}
```

场景二：`@JvmField`另一个经常使用的场景就是用于`Companion Object`伴生对象中。

未使用`@JvmField`注解

```kotlin
class Person {
    companion object {
        val MAX_AGE = 120
    }
}
```

反编译成Java代码：

```java
public final class Person {
   private static final int MAX_AGE = 120;//注意: 这里默认是private私有的MAX_AGE，所以在Java中调用无法直接通过Person类名.变量名访问
   public static final Person.Companion Companion = new Person.Companion((DefaultConstructorMarker)null);
   public static final class Companion {
   //在Java中调用无法直接通过Person类名.变量名访问, 
   //而是通过静态内部类Companion的getMAX_AGE间接访问，类似这样Person.Companion.getMAX_AGE();
      public final int getMAX_AGE() {
         return Person.MAX_AGE;
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

但是如果使用该注解就能直接通过Person类名.变量名访问

```kotlin
class Person {
    companion object {
        @JvmField
        val MAX_AGE = 120
    }
}
//在Java中调用
public static void main(String[] args) {
    System.out.println(Person.MAX_AGE);//可以直接调用，因为它已经变成了public了
}
```

反编译成Java代码

```java
public final class Person {
   @JvmField
   public static final int MAX_AGE = 120;//公有的MAX_AGE的，外部可以直接调用
   public static final Person.Companion Companion = new Person.Companion((DefaultConstructorMarker)null);
   public static final class Companion {
      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

#### @JvmMultifileClass

```kotlin
@Target(AnnotationTarget.FILE)
@MustBeDocumented
@OptionalExpectation
public expect annotation class JvmMultifileClass()
```

该注解主要是为了生成多文件的类（使用 ***expect*** 定义预期声明）;

在Kotlin分别定义两个顶层函数在两个不同文件中，可通过该注解将多个文件中的类方法合并到一个类中。

```kotlin
//存在于IOUtilA文件中
@file:JvmName("IOUtils")
@file:JvmMultifileClass

package com.mikyou.annotation

import java.io.IOException
import java.io.Reader

fun closeReaderQuietly(input: Reader?) {
    try {
        input?.close()
    } catch (ioe: IOException) {
        // ignore
    }
}

//存在于IOUtilB文件中
@file:JvmName("IOUtils")
@file:JvmMultifileClass

package com.mikyou.annotation

import java.io.IOException
import java.io.InputStream

fun closeStreamQuietly(input: InputStream?) {
    try {
        input?.close()
    } catch (ioe: IOException) {
        // ignore
    }
}
//在Java中使用
public class Test {
    public static void main(String[] args) {
        //即使存在于不同文件中，但是对于外部Java调用仍然是同一个类IOUtils
        IOUtils.closeReaderQuietly(null);
        IOUtils.closeStreamQuietly(null);
    }
}
```

#### @JvmName

```kotlin
//作用的目标有: 函数、属性getter方法、属性setter方法、文件
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY_GETTER, AnnotationTarget.PROPERTY_SETTER, AnnotationTarget.FILE)
@Retention(AnnotationRetention.BINARY)
@MustBeDocumented
public actual annotation class JvmName(actual val name: String)//有个name参数，将生成传入指定name的名称
```

将改变由Kotlin默认生成的Java方法、字段或类名；

```kotlin
class Student {
    @get:JvmName(name = "getStudentName")//修改属性的getter函数名称
    @set:JvmName(name = "setStudentName")//修改属性的setter函数名称
    var name: String = "Tim"

    @JvmName("getStudentScore")//修改函数名称
    fun getScore(): Double {
        return 110.5
    }
}
//修改生成的类名，默认Kotlin会生成以文件名+Kt后缀组合而成的类名
@file:JvmName("IOUtils")//注意:该注解一定要在第一行，package顶部
package com.mikyou.annotation

import java.io.IOException
import java.io.Reader


fun closeReaderQuietly(input: Reader?) {
    try {
        input?.close()
    } catch (ioe: IOException) {
        // ignore
    }
}
```

反编译后的Java代码

```java
public final class Student {
   @NotNull
   private String name = "Tim";

   @JvmName(name = "getStudentName")
   @NotNull
   //已经修改成传入getStudentName
   public final String getStudentName() {
      return this.name;
   }

   @JvmName(name = "setStudentName")
   //已经修改成传入setStudentName
   public final void setStudentName(@NotNull String var1) {
      Intrinsics.checkParameterIsNotNull(var1, "<set-?>");
      this.name = var1;
   }

   @JvmName(name = "getStudentScore")
   //已经修改成传入getStudentScore
   public final double getStudentScore() {
      return 110.5D;
   }
}
```

#### @JvmOverloads

```kotlin
//作用对象是函数和构造函数
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.CONSTRUCTOR)
@MustBeDocumented
@OptionalExpectation
public expect annotation class JvmOverloads()
```

指导Kotlin编译器为带默认参数值的函数(包括构造函数)生成多个重载函数。

- 注解使用 该注解使用最多就是用于带默认值函数的重载，在Android中我们在自定义View的时候一般会重载多个构造器，需要加入该注解，如果不加默认只定义一个构造器，那么当你直接在XML使用这个自定义View的时候会抛出异常。

  

```kotlin
class ScrollerView @JvmOverloads constructor(
    context: Context,
    attr: AttributeSet? = null,
    defStyle: Int = 0
) : View(context, attr, defStyle) {
    //...
}
```

反编译后的Java代码

```java
public final class ScrollerView extends View {
   @JvmOverloads
   public ScrollerView(@NotNull Context context, @Nullable AttributeSet attr, int defStyle) {
      Intrinsics.checkParameterIsNotNull(context, "context");
      super(context, attr, defStyle);
   }

   // $FF: synthetic method
   @JvmOverloads
   public ScrollerView(Context var1, AttributeSet var2, int var3, int var4, DefaultConstructorMarker var5) {
      if ((var4 & 2) != 0) {
         var2 = (AttributeSet)null;
      }
      if ((var4 & 4) != 0) {
         var3 = 0;
      }
      this(var1, var2, var3);
   }

   @JvmOverloads
   public ScrollerView(@NotNull Context context, @Nullable AttributeSet attr) {
      this(context, attr, 0, 4, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public ScrollerView(@NotNull Context context) {
      this(context, (AttributeSet)null, 0, 6, (DefaultConstructorMarker)null);
   }
   //...
}
```

#### @JvmPackageName

```kotlin
@Target(AnnotationTarget.FILE)//作用于文件
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
@SinceKotlin("1.2")//Kotlin1.2版本加入
internal annotation class JvmPackageName(val name: String)
```

更改从使用该注解标注的文件生成的.class文件的JVM包的完全限定名称。 这不会影响Kotlin客户端在此文件中查看声明的方式，但Java客户端和其他JVM语言客户端将看到类文件，就好像它是在指定的包中声明的那样。 如果使用此批注对文件进行批注，则它只能包含函数，属性和类型声明，但不能包含。

```kotlin
//以Collection源码为例
@file:kotlin.jvm.JvmPackageName("kotlin.collections.jdk8")

package kotlin.collections
```

可以看到该类会编译生成到`kotlin.collections.jdk8`包名下

#### @JvmStatic

```kotlin
//作用于函数、属性、属性的setter和getter
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY, AnnotationTarget.PROPERTY_GETTER, AnnotationTarget.PROPERTY_SETTER)
@MustBeDocumented
@OptionalExpectation
public expect annotation class JvmStatic()
```

能被用在对象声明或者Companion object伴生对象的方法上，把它们暴露成一个Java的静态方法;

- 注解使用 @JvmStatic这个注解一般经常用于伴生对象的方法上，供给Java代码调用

  

```kotlin
class Data {
    companion object {
        fun getDefaultDataName(): String {
            return "default"
        }
    }
}
//在java中调用,只能是Data.Companion.getDefaultDataName()调用
public class Test {
    public static void main(String[] args) {
        System.out.println(Data.Companion.getDefaultDataName());
    }
}
```

反编译后Java代码

```java
public final class Data {
   public static final Data.Companion Companion = new Data.Companion((DefaultConstructorMarker)null);
   public static final class Companion {
      @NotNull
      public final String getDefaultDataName() {
         return "default";
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

使用`@JvmStatic`注解后

```kotlin
class Data {
    companion object {
        @JvmStatic
        fun getDefaultDataName(): String {
            return "default"
        }
    }
}
//在java中调用,可以直接这样Data.getDefaultDataName()调用
public class Test {
    public static void main(String[] args) {
        System.out.println(Data.getDefaultDataName());
    }
}
```

反编译后的Java代码

```java
public final class Data {
   public static final Data.Companion Companion = new Data.Companion((DefaultConstructorMarker)null);

   @JvmStatic
   @NotNull
   //注意它会在Data类内部自动生成一个getDefaultDataName，然后内部还是通过Companion.getDefaultDataName()去调用。
   public static final String getDefaultDataName() {
      return Companion.getDefaultDataName();
   }

   public static final class Companion {
      @JvmStatic
      @NotNull
      public final String getDefaultDataName() {
         return "default";
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

#### @JvmSuppressWildcards和@JvmWildcard

```kotlin
//作用于类、函数、属性、类型
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY, AnnotationTarget.TYPE)
@MustBeDocumented
@OptionalExpectation
//指定suppress为true表示不生成，false为生成通配符，默认是true不生成
public expect annotation class JvmSuppressWildcards(val suppress: Boolean = true)

@Target(AnnotationTarget.TYPE)
@Retention(AnnotationRetention.BINARY)
@MustBeDocumented
public actual annotation class JvmWildcard
```

用于指示编译器生成或省略类型参数的通配符:

- JvmSuppressWildcards用于参数的泛型是否生成或省略通配符;
- JvmWildcard用于返回值的类型是否生成或省略通配符;

```kotlin
interface ICovert {
    fun covertData(datas: List<@JvmSuppressWildcards(suppress = false) String>)//@JvmSuppressWildcardsd用于参数类型

    fun getData(): List<@JvmWildcard String>//@JvmWildcard用于返回值类型
}
class CovertImpl implements ICovert {
    @Override
    public void covertData(List<? extends String> datas) {//参数类型生成通配符

    }
    @Override
    public List<? extends String> getData() {//返回值类型生成通配符
        return null;
    }
}
```

#### @JvmSynthetic

```kotlin
//作用于函数、属性的setter,getter以及字段
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY_GETTER, AnnotationTarget.PROPERTY_SETTER, AnnotationTarget.FIELD)
@OptionalExpectation
public expect annotation class JvmSynthetic()
```

它在生成的类文件中将适当的元素标记为合成,并且编译器标记为合成的任何元素都将无法从Java语言中访问。

合成属性(Synthetic属性): JVM字节码标识的ACC_SYNTHETIC属性用于标识该元素实际上不存在于原始源代码中，而是由编译器生成。

它一般用于支持代码生成，允许编译器生成不应向其他开发人员公开但需要支持实际公开接口所需的字段和方法。我们可以将其视为超越private或protected级别。

- 注解使用

```kotlin
class Synthetic {
    @JvmSynthetic
    val name: String = "Tim"
    var age: Int
        @JvmSynthetic
        set(value) {
        }
        @JvmSynthetic
        get() {
            return 18
        }
}
```

反编译后的Java代码

```java
public final class Synthetic {
   // $FF: synthetic field
   @NotNull
   private final String name = "Tim";

   @NotNull
   public final String getName() {
      return this.name;
   }

   // $FF: synthetic method//我们经常看到这些注释，就是通过@Synthetic注解生成的
   public final int getAge() {
      return 18;
   }

   // $FF: synthetic method
   public final void setAge(int value) {
   }
}
```

通过反编译代码可能看不到什么，我们直接可以通过**javap -v xxx.class**查阅生成的字节码文件描述

```java
public final int getAge();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_FINAL, ACC_SYNTHETIC//添加ACC_SYNTHETIC标识
    Code:
      stack=1, locals=1, args_size=1
         0: bipush        18
         2: ireturn
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       3     0  this   Lcom/mikyou/annotation/Synthetic;
      LineNumberTable:
        line 12: 0

  public final void setAge(int);
    descriptor: (I)V
    flags: ACC_PUBLIC, ACC_FINAL, ACC_SYNTHETIC//添加ACC_SYNTHETIC标识
    Code:
      stack=0, locals=2, args_size=2
         0: return
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lcom/mikyou/annotation/Synthetic;
            0       1     1 value   I
      LineNumberTable:
        line 9: 0
```

#### @Throws

```kotlin
//作用于函数、属性的getter、setter函数、构造器函数
@Target(AnnotationTarget.FUNCTION, AnnotationTarget.PROPERTY_GETTER, AnnotationTarget.PROPERTY_SETTER, AnnotationTarget.CONSTRUCTOR)
@Retention(AnnotationRetention.SOURCE)
public annotation class Throws(vararg val exceptionClasses: KClass<out Throwable>)//这里是异常类KClass不定参数，可以同时指定一个或多个异常
```

用于Kotlin中的函数,属性的setter或getter函数，构造器函数抛出异常;

```kotlin
@Throws(IOException::class)
fun closeQuietly(output: Writer?) {
    output?.close()
}
```

#### @Transient

该注解充当了Java中的transient关键字

#### @Strictfp

该注解充当了Java中的strictfp关键字

#### @Synchronized

该注解充当了Java中的synchronized关键字

#### @Volatile

该注解充当了Java中的volatile关键字

### Kotlin 自定义注解遇到的坑

#### @AutoService不起作用

使用Kotlin编写Processor类，为了少写META-INF/services/javax.annotation.processing.Processor，我们会导入auto-service及使用下面的注解来代替：

```groovy
    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'
    implementation 'com.google.auto.service:auto-service:1.0-rc4'
```



```kotlin
@AutoService(Processor::class)
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@SupportedAnnotationTypes("xxxxx")
class MyProcessor: AbstractProcessor() {
```

auto-service似乎找不到已经添加@AutoService注解的类；

解决方法是修改build.gradle: 加上kotlin-kapt插件及改annotationProcessor为kapt;

```groovy
plugins {
    id 'java'
    id 'kotlin'
    id 'kotlin-kapt'
}

dependencies {
    implementation project(path: ':annotation')

    kapt 'com.google.auto.service:auto-service:1.0-rc4'
    implementation 'com.google.auto.service:auto-service:1.0-rc4'

    implementation 'com.google.auto:auto-common:0.10'

    implementation 'com.squareup:javapoet:1.12.1'
}
```

当然也可以使用原始方法创建META-INF/services/javax.annotation.processing.Processor来解决；