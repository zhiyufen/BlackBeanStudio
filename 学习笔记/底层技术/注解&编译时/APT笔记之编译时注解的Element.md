# APT笔记之编译时注解的Element

[TOC]

&emsp;&emsp;Element用于建模java编程语言元素的接口，位于javax.lang.model.element包; 
&emsp;&emsp;Element表示一个程序元素，比如包、类或者方法。每个元素都表示一个静态的语言级构造（不表示虚拟机的运行时构造）。
&emsp;&emsp;Element是编译时注解处理器技术的基础

## Element接口方法

&emsp;&emsp;Element是一个继承于AnnotatedConstruct的接口，其暴露给外部使用的接口：

|                           接口方法                           |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                    TypeMirror asType\(\)                     |                     返回此元素定义的类型                     |
|                   ElementKind getKind\(\)                    | 返回此元素的类型，判断是哪种元素：包、类、接口、方法、字段…  |
|                Set<Modifier> getModifiers\(\)                |      返回此元素的修饰符 ： public static final等关键字       |
|                    Name getSimpleName\(\)                    |        返回此元素的简单名称,不带包名。比如activity名         |
|               Element getEnclosingElement\(\)                | 返回封装此元素的父元素，如果此元素的声明在词法上直接封装在另一个元素的声明中，则返回那个封装元素； 如果此元素是顶层类型，则返回它的包如果此元素是一个包，则返回 null； 如果此元素是一个泛型参数，则返回 null\. |
|        List<? extends Element> getEnclosedElements()         | 返回该元素直接包含的子元素,通常对一个PackageElement而言，它可以包含TypeElement；对于一个TypeElement而言，它可能包含属性VariableElement，方法ExecutableElement |
| <A extends Annotation> A getAnnotation\(Class<A> annotationType\) | 返回此元素针对指定类型的注解（如果存在这样的注解），否则返回 null。注解可以是继承的，也可以是直接存在于此元素上的 |

PS: Element接口所代表的元素只在编译期可见，用于保存元素在编译期的各种状态，而Type接口所代表的元素是运行期可见，用于保存元素在运行期的各种状态。

## Element子类

&emsp;&emsp;Element 有五个直接子接口，它们分别代表一种特定类型的元素；五个子类各有各的用处并且有各种独立的方法，在使用的时候可以强制将Element对象转换成其中的任一一种，但是前提是满足条件的转换，不然会抛出异常。

| 子类                 | 类说明                                                       |
| -------------------- | ------------------------------------------------------------ |
| TypeElement          | 一个类或接口程序元素,包括包名，类(或方法，或参数)名/类型 ,在生成动态代码的时候，我们往往需要知道变量/方法参数的类型，以便写入正确的类型声明 |
| VariableElement      | 一个字段、enum 常量、方法或构造方法参数、局部变量或异常参数  |
| ExecutableElement    | 某个类或接口的方法、构造方法或初始化程序（静态或实例），包括注解类型元素 |
| PackageElement       | 一个包程序元素                                               |
| TypeParameterElement | 一般类、接口、方法或构造方法元素的泛型参数                   |

&emsp;&emsp;为了更好理解上面的子类，请参考下图的说明：
![element](D:\My_SAssustant\SAssistant学习\培训资料\APT-PPT\Element.png)

### 类图
![Element类图](https://img-blog.csdnimg.cn/20190304103602653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWFpaGFu,size_16,color_FFFFFF,t_70)

### TypeElement
&emsp;&emsp; TypeElement定义的一个类或接口程序元素，相当于当前注解所在的class对象；

| 方法                                                        | 说明                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| NestingKind getNestingKind\(\);                             | 返回此类型元素的嵌套种类                                     |
| Name getQualifiedName\(\);                                  | 返回此类型元素的完全限定名称。更准确地说，返回规范 名称。对于没有规范名称的局部类和匿名类，返回一个空名称 |
| TypeMirror getSuperclass\(\);                               | 返回此类型元素的直接超类。如果此类型元素表示一个接口或者类 java\.lang\.Object，则返回一个种类为 NONE 的 NoType |
| List<? extends TypeMirror> getInterfaces\(\);               | 返回直接由此类实现或直接由此接口扩展的接口类型               |
| List<? extends TypeParameterElement> getTypeParameters\(\); | 按照声明顺序返回此类型元素的形式类型参数                     |

### VariableElement
&emsp;&emsp;

| 方法                             | 说明           |
| -------------------------------- | -------------- |
| Object getConstantValue\(\);     | 变量初始化的值 |
| Element getEnclosingElement\(\); | 获取相关类信息 |

##### ExecutableElement
&emsp;&emsp; 表示一个方法本；

| 方法                                              | 说明                                                  |
| ------------------------------------------------- | ----------------------------------------------------- |
| List<? extends VariableElement> getParameters\(\) | 用于获取方法的参数元素，每个元素是一个VariableElement |
| TypeMirror getReturnType\(\)                      | 获取方法元素的返回值，返回衣蛾TypeMirror表示          |

本文作笔记于
    https://blog.csdn.net/baiaihan/article/details/88038201