# Android笔记之注解

[TOC]

## 概念

​		注解（也被称为元数据）为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据。注解类型定义指定了一种新的类型，一种特殊的接口类型。 在关键词 interface 前加 @ 符号也就是用 @interface 来区分注解的定义和普通的接口声明。

#### 本质

**注解实际上就是一种代码标签，它作用的对象是代码**。它可以给特定的注解代码标注一些额外的信息。然而这些信息可以选择不同保留时期，比如源码期、编译期、运行期。然后在不同时期，可以通过某种方式获取标签的信息来处理实际的代码逻辑，这种方式常常就是我们所说的**反射**。

#### 用处

1. 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息;
2. 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理;
3. 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取

## Java注解
&emsp;&emsp;在Java中，注解(Annotation)引入始于Java5，用来描述Java代码的元信息，通常情况下注解不会直接影响代码的执行，尽管有些注解可以用来做到影响代码执行.

#### 元注解
&emsp;&emsp;元注解的作用就是负责注解其他注解;

| 注解类型   | 含义                                                         |
| ---------- | :----------------------------------------------------------- |
| Documented | 表示含有该注解类型的元素\(带有注释的\)会通过javadoc或类似工具进行文档化 |
| Inherited  | 表示注解类型能被自动继承                                     |
| Retention  | 表示注解类型的存活时长                                       |
| Target     | 表示注解类型所适用的程序元素的种类                           |
| Repeatable | 表示注解在重复使用                                           |

#### Documented
&emsp;&emsp;@Documented是一个标记注解，没有成员。使用带有该元注解的注解时，如果类型声明是用Documented来注解的，这种类型的注解被作为被标注的程序成员的公共API；

```java
@Documented
@Retention(SOURCE)
@Target(TYPE)
public @interface AutoService {
  /** Returns the interface implemented by this service provider. */
  Class<?> value();
}

```

#### Inherited
&emsp;&emsp; @Inherited是一个标记注解；@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

&emsp;&emsp;当@Inherited annotation类型标注的annotation的Retention是 RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

比如ViewPager中@DecorView注解:

```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    @Inherited
    public @interface DecorView {
    }
```

#### Retention
&emsp;&emsp;@Retention表示该注解类型的注解保留的时长。当注解类型声明中没有@Retention元注解，则默认保留策略为RetentionPolicy.CLASS。关于保留策略(RetentionPolicy)是枚举类型，共定义3种保留方式；

|  取值   |                             描述                             |
| :-----: | :----------------------------------------------------------: |
| SOURCE  |        仅存在Java源文件，经过编译器后便丢弃相应的注解        |
|  CLASS  | 存在Java源文件，以及经编译器后生成的Class字节码文件，但在运行时VM不再保留注释 |
| RUNTIME | 存在源文件、编译生成的Class字节码文件，以及保留在运行时VM中，可通过反射性地读取注解 |

比如@SuppressWarnings注解：

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}

```

#### Target
&emsp;&emsp;@Target表示该注解类型的所使用的程序元素类型。当注解类型声明中没有@Target元注解，则默认为可适用所有的程序元素。如果存在指定的@Target元注解，则编译器强制实施相应的使用限制。关于程序元素(ElementType)是枚举类型，共定义8种程序元素：

```java
ElementType.ANNOTATION_TYPE   //给一个注解进行注解
ElementType.CONSTRUCTOR       //给构造方法进行注解
ElementType.FIELD             //给属性进行注解
ElementType.LOCAL_VARIABLE    //给局部变量进行注解
ElementType.METHOD            //给方法进行注解
ElementType.PACKAGE           //给一个包进行注解
ElementType.PARAMETER         //给一个方法内的参数进行注解
ElementType.TYPE              //给一个类型进行注解，比如类、接口、枚举
```
#### @Repeatable

&emsp;&emsp;Repeatable 是可重复的意思，@Repeatable 是 Java 1.8 才加进来的， 通常是注解的值可以同时取多个。

```java
@interface Persons {
    Person[]  value();
}
  
  
@Repeatable(Persons.class)
@interface Person{
    String role default "";
}
  
  
@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{
  
}

```


### Java内置注解

```java
@Deprecated

@Override

@SuppressWarnings

@SafeVarargs

@FunctionalInterface
```

#### @Deprecated
&emsp;&emsp;是一个标记注释（所谓标记注释，就是在源程序中加入这个标记后，并不影响程序的编译，但有时编译器会显示一些警告信息）；该注解表示这个并不建议使用这个类成员。因为这个类成员在未来的JDK版本中可能被删除。之所以在现在还保留，是因为给那些已经使用了这些类成员的程序一个缓冲期。

&emsp;&emsp;该注解可以用来标记类，方法，属性，另外如何标注这些注解，那么在使用时，编辑器/编译器会进行警告。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

#### @Override

&emsp;&emsp;该注解是标识某一个方法是否覆盖了它的父类的方法；

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

#### @SuppressWarnings

&emsp;&emsp;当我们的一个方法调用了弃用的方法或者进行不安全的类型转换，编译器会生成警告。我们可以为这个方法增加@SuppressWarnings注解，来抑制编译器生成警告；可以修饰的元素为类，方法，方法参数，属性，局部变量；

```
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    /**
     * The set of warnings that are to be suppressed by the compiler in the
     * annotated element.  Duplicate names are permitted.  The second and
     * successive occurrences of a name are ignored.  The presence of
     * unrecognized warning names is <i>not</i> an error: Compilers must
     * ignore any warning names they do not recognize.  They are, however,
     * free to emit a warning if an annotation contains an unrecognized
     * warning name.
     *
     * <p> The string {@code "unchecked"} is used to suppress
     * unchecked warnings. Compiler vendors should document the
     * additional warning names they support in conjunction with this
     * annotation type. They are encouraged to cooperate to ensure
     * that the same names work across multiple compilers.
     * @return the set of warnings to be suppressed
     */
    String[] value();
}

```

```java
    //该方法并没有被使用,但它作为JS API方法，会被H5调用；
    @SuppressWarnings("unused")
    public void jsAPI() {
        //nothing
    }
```

##### 抑制警告关键字

|           关键字           | 描述                                                         |
| :------------------------: | :----------------------------------------------------------- |
|            all             | 抑制所有警告                                                 |
|           boxing           | to suppress warnings relative to boxing/unboxing operations  \(抑制装箱，拆箱操作时候的警告 |
|            cast            | to suppress warnings relative to cast operations  抑制映射相关的警告 |
|          dep\-ann          | to suppress warnings relative to deprecated annotation  抑制启用注释的警告 |
|        deprecation         | to suppress warnings relative to deprecation  抑制过期方法警告 |
|        fallthrough         | to suppress warnings relative to missing breaks in switch statements  抑制确定switch中缺失breaks的警告 |
|          finally           | to suppress warnings relative to finally block that don’t return  抑制finally模块没有返回的警告 |
|           hiding           | to suppress warnings relative to locals that hide variable   |
|     incomplete\-switch     | to suppress warnings relative to missing entries in a switch statement \(enum case\)  忽略没有完整的switch语句 |
|            nls             | to suppress warnings relative to non\-nls string literals  忽略非nls 格式的字符 |
|            null            | to suppress warnings relative to null analysis  忽略对null的操作 |
|          rawtypes          | to suppress warnings relative to un\-specific types when using generics on class params  使用泛型类型时忽略没有指定相应的类型 |
|        restriction         | to suppress warnings relative to usage of discouraged or forbidden references |
|           serial           | to suppress warnings relative to missing serialVersionUID field for a serializable class  忽略在序列化类中没有声明serialVersionUID变量 |
|       static\-access       | to suppress warnings relative to incorrect static access  抑制不正确的静态访问方式警告 |
|     synthetic\-access      | to suppress warnings relative to unoptimized access from inner classes  抑制子类没有按最优方法访问内部类的警告 |
|         unchecked          | to suppress warnings relative to unchecked operations  抑制没有进行类型检查操作的警告 |
| unqualified\-field\-access | to suppress warnings relative to field access unqualified  抑制没有权限访问的域的警告 |
|           unused           | to suppress warnings relative to unused code  抑制没被使用过的代码的警告 |

## Kotlin 注解

略（请阅读Kotlin笔记之注解）

## Android注解

### 配置环境

&emsp;&emsp;使用Android注解前需要导入相关的包

```groovy
compile 'com.android.support:support-annotations:latest.integration'
//或AndroidX
implementation 'androidx.annotation:annotation:1.0.0'
```

&emsp;&emsp;注意：很多包已经引入了support-annotations,没必要再次引入， 比如appcompat；

#### @SafeVarargs
&emsp;&emsp;参数安全类型注解。它的目的是提醒开发者不要用参数做一些不安全的操作,它的存在会阻止编译器产生 unchecked 这样的警告。它是在 Java 1.7 的版本中加入的。

####  @FunctionalInterface
&emsp;&emsp;函数式接口注解，这个是 Java 1.8 版本引入的新特性。函数式编程很火，所以 Java 8 也及时添加了这个特性。函数式接口 (Functional Interface) 就是一个具有一个方法的普通接口。


### 使用

&emsp;&emsp;Android注解给我们提供了三种主要和其他注释供我们使用：

```java
IntDef和StringDef注解；
资源类型注解；
Null注解；
其他实用注解
```
#### IntDef和StringDef注解

&emsp;&emsp;使用这两注解代替枚举；具体例子使用方法如下:

```java
    @Retention(RetentionPolicy.SOURCE)
    @IntDef({INSTALLATION_NONE, INSTALLATION_IN_PROGRESS, INSTALLATION_COMPLETED, INSTALLATION_FAILED,INSTALLATION_WAITING})
    public @interface Type {}
	
    public static final int INSTALLATION_NONE = -1;
    public static final int INSTALLATION_IN_PROGRESS = 0;
    public static final int INSTALLATION_COMPLETED = 1;
    public static final int INSTALLATION_FAILED = 2;
    public static final int INSTALLATION_WAITING = 3;
	
	@Type
    private final int mType;//对变量进行Type注解
	
	//对返回结果进行Type注解
	@Type
    public int getType() {
        return mType;
    }
	//对传入参数进行Type注解，
	//也就是传入参数不是 INSTALLATION_NONE, INSTALLATION_IN_PROGRESS, INSTALLATION_COMPLETED, INSTALLATION_FAILED,INSTALLATION_WAITING 当中的话，将不允许；
	public void setType(@Type int type) {
        mType = type;
        return this;
    }

```

#### 资源类型注解

&emsp;&emsp;资源注解是为了防止我们在使用程序资源的时候，错误传递资源类型给函数，导致程序错误！

```java
AnimRes	              动画
AnimatorRes	animator  资源类型
AnyRes	              任何资源类型
ArrayRes	          数组资源类型
AttrRes	              属性资源类型
BoolRes	bool          类型资源类型
ColorRes	          颜色资源类型
DimenRes	          长度资源类型
DrawableRes	          图片资源类型
IdRes	              资源id
InterpolatorRes	      动画插值器
LayoutRes	          layout资源
MenuRes	menu          资源
RawRes	              raw资源
StringRes	          字符串资源
StyleRes	          style资源
StyleableRes	      Styleable资源类型
TransitionRes	      transition资源类型
XmlRes	              xml资源
```

比如我们常用的 AlertDialog 的 setTitle 规定使用StringRes，如果你传入其它资源或者int具体数，会报错；

```java
        /**
         * Set the title using the given resource id.
         *
         * @return This Builder object to allow for chaining of calls to set methods
         */
        public Builder setTitle(@StringRes int titleId) {
            P.mTitle = P.mContext.getText(titleId);
            return this;
        }
```

#### Null注解

&emsp;&emsp;null注解对应的有两个详细的注解：

```
@NonNull：不能为空
@Nullable：可以为空
```

#### Threading 注解

&emsp;&emsp;@Thread注解是帮助我们指定方法在特定线程中执行处理，如果和指定的线程不一致，抛出异常；Threading 注解类型：

```
@UiThread： UI线程
@MainThread ：主线程
@WorkerThread： 子线程
@BinderThread ： Binder 线程
```

#### Value Constraints注解

```
@Size         //定义长度大小，可选择最小和最大长度使用

@IntRange     //指定int类型范围的注解

@FloatRange   //定的是float类型的数据对象
```

#### @CallSuper 注解

&emsp;&emsp; @CallSuper注解主要是用来强调在覆盖父类方法时，需要实现父类的方法，及时调用对应的super.***方法,当用@CallSuper修饰了该方法，如果子类覆盖的后没有实现对呀的super方法会抛出异常. 
比如Activity的onCreate方法就加这个注解，我们在重写该方法时， 必须调用super.onCreate(）：

```java
    @MainThread
    @CallSuper
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        if (DEBUG_LIFECYCLE) Slog.v(TAG, "onCreate " + this + ": " + savedInstanceState);
	....

```

#### @CheckResult 注解

&emsp;&emsp;假设你定义了一个方法返回一个值，你期望调用者用这个值做些事情，那么你可以使用@CheckResult注解标注这个方法，强制用户定义一个相应的返回值，使用它！



## 自定义注解

### 定义
&emsp;&emsp;注解通过 @interface 关键字进行定义。
```java
public @interface TestAnnotation {

}
```
###	注解的属性

&emsp;&emsp;注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    int id();
    String msg();
}
```

&emsp;&emsp;上面代码定义了 TestAnnotation 这个注解中拥有 id 和 msg 两个属性。在使用的时候，我们应该给它们进行赋值。赋值的方式是在注解的括号内以 value=”” 形式，多个属性之前用 "," 隔开。

```java
@TestAnnotation(id=3,msg="hello annotation")
public class Test {
  
}
```

&emsp;&emsp;需要注意的是，在注解中定义属性时它的类型必须是 8 种基本数据类型外加 类、接口、注解及它们的数组。注解中属性可以有默认值，默认值需要用 default 关键值指定。

### 注解的提取
&emsp;&emsp;当开发者使用了Annotation 修饰了类、方法、Field 等成员之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT（Annotation Processing Tool)。
&emsp;&emsp;简单核心来讲分两个步骤：

1. 注解通过反射获取。首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解。
2. 然后通过 getAnnotation() 方法来获取 Annotation 对象或getAnnotations() 方法。

```java
	void onBtnTestClick(View view) {
        Method[] methods = getClass().getDeclaredMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(TestAnnotation.class)) {
                TestAnnotation ano = method.getAnnotation(TestAnnotation.class);
                mTv.append(ano.id() + "---->" + ano.msg());
            }
        }
    }

```
具体如何使用APT，请参考 Androi笔记之APT编译时注解

本文笔记来自于下面的文章：

> https://blog.csdn.net/wzgiceman/article/details/53406248
> https://blog.csdn.net/wzgiceman/article/details/53483665
> https://blog.csdn.net/walk_man_3/article/details/79480326
> https://www.cnblogs.com/peida/archive/2013/04/24/3036689.html
> https://zhuanlan.zhihu.com/p/63602769