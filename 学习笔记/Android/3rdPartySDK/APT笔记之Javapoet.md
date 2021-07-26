# APT笔记之Javapoet

&emsp;&emsp;javapoet是配合APT在项目编译期间动态生成代码，并且使用其API可以自动生成导包语句。这可以减少我们在项目开发中模板化代码的编写，减轻程序员开发所需要的时间，提高编码效率。

[Javapoet github地址](https://github.com/square/javapoet)
## 配置
&emsp;&emsp;只需要在gradle中添加下面的库即可；
```xml
dependencies {
    compile 'com.squareup:javapoet:1.8.0'
}

```


## 代码基础编写

&emsp;&emsp;JavaPoet定义了一系列类来尽可能优雅的描述java源文件的结构,主要的类可以分为以下5大种：Spec,Name,CodeBlock,JavaFile, CodeWrite.

1. Spec 用来描述Java中基本的元素:
	```
	AnnotationSpec  //注解元素
	FieldSpec       //字段元素
	MethodSpec      //构造方法和一般方法元素
	ParameterSpec   //参数元素
	TypeSpec        //类和接口元素
	```
2. Name 用来描述类型的引用，包括Void，原始类型（int，long等）和Java类等。
	```
	TypeName 
	ArrayTypeName
	ClassName
	ParameterizedTypeName  //泛型
	TypeVariableName
	WildcardTypeName
	```
3. CodeBlock 用来描述代码块的内容，包括普通的赋值，if判断，循环判断等。
4. JavaFile 完整的Java文件，JavaPoet的主要的入口。
5. CodeWriter 读取JavaFile并转换成可阅读可编译的Java源文件。



### 编写HelloWord

&emsp;&emsp;我们要编写下面的代码：

```java
package com.example.helloworld;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```
&emsp;&emsp;那么使用javapoet代码来生成我们需要的代码：

```java
MethodSpec main = MethodSpec.methodBuilder("main") //定义方法名
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC) //定义修饰符
    .returns(void.class) //定义方法返回类型
    .addParameter(String[].class, "args") //定义方法的参数类型和变量名
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!") //添加方法体的内容
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld") //定义类名
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL) //定义类的修饰符
    .addMethod(main) //把上面定义的方法添加到类中；
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld) //定义包名
    .build();

javaFile.writeTo(System.out); //将类输出
```

&emsp;&emsp; 我们来看看Modifier还哪些修饰符：

```java
public enum Modifier {

    // See JLS sections 8.1.1, 8.3.1, 8.4.3, 8.8.3, and 9.1.1.
    // java.lang.reflect.Modifier includes INTERFACE, but that's a VMism.

    /** The modifier {@code public} */          PUBLIC,
    /** The modifier {@code protected} */       PROTECTED,
    /** The modifier {@code private} */         PRIVATE,
    /** The modifier {@code abstract} */        ABSTRACT,
    /**
     * The modifier {@code default}
     * @since 1.8
     */
     DEFAULT,
    /** The modifier {@code static} */          STATIC,
    /** The modifier {@code final} */           FINAL,
    /** The modifier {@code transient} */       TRANSIENT,
    /** The modifier {@code volatile} */        VOLATILE,
    /** The modifier {@code synchronized} */    SYNCHRONIZED,
    /** The modifier {@code native} */          NATIVE,
    /** The modifier {@code strictfp} */        STRICTFP;

    /**
     * Returns this modifier's name in lowercase.
     */
    public String toString() {
        return name().toLowerCase(java.util.Locale.US);
    }
}
```
&emsp;&emsp;除了transient，strictfp；上面的关键字在平时我们写代码是不是都熟悉。
&emsp;&emsp;strictfp： 即 strict float point (精确浮点)；strictfp 关键字可应用于类、接口或方法。使用 strictfp 关键字声明一个方法时，该方法中所有的float和double表达式都严格遵守FP-strict的限制,符合IEEE-754规范。当对一个类或接口使用 strictfp 关键字时，该类中的所有代码，包括嵌套类型中的初始设定值和代码，都将严格地进行计算。严格约束意味着所有表达式的结果都必须是 IEEE 754 算法对操作数预期的结果，以单精度和双精度格式表示。

&emsp;&emsp;transient： 当串行化某个对象时，如果该对象的某个变量是transient，那么这个变量不会被串行化进去。也就是说，假设某个类的成员变量是transient，那么当通过ObjectOutputStream把这个类的某个实例保存到磁盘上时，实际上transient变量的值是不会保存的。因为当从磁盘中读出这个对象的时候，对象的该变量会没有被赋值。

### 代码控制流程编写
&emsp;&emsp;Javapoet并没有对方法体进行建模，代替的，可以使用字符串来直接作为方法体：

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addCode(""
        + "int total = 0;\n"
        + "for (int i = 0; i < 10; i++) {\n"
        + "  total += i;\n"
        + "}\n")
    .build();
```

生成的代码如下：

```java
void main() {
  int total = 0;
  for (int i = 0; i < 10; i++) {
    total += i;
  }
}
```

&emsp;&emsp;但这样代码量多时， 缩进换行是一个很麻烦的事， 因此Javapoet提供一些API来更方便添加方法体; 因此上面的代码可以写成：
```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addStatement("int total = 0")
    .beginControlFlow("for (int i = 0; i < 10; i++)") //For循环控制开始
    .addStatement("total += i")
    .endControlFlow() //控制结束
    .build();
```

&emsp;&emsp;对于一些控制语句是可能有无限的控制流程的，比如if/else, try/catch， 这时就可以使用 nextControlFlow() 来处理；

if/else例子：
```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addStatement("long now = $T.currentTimeMillis()", System.class)
    .beginControlFlow("if ($T.currentTimeMillis() < now)", System.class)
    .addStatement("$T.out.println($S)", System.class, "Time travelling, woo hoo!")
    .nextControlFlow("else if ($T.currentTimeMillis() == now)", System.class)
    .addStatement("$T.out.println($S)", System.class, "Time stood still!")
    .nextControlFlow("else")
    .addStatement("$T.out.println($S)", System.class, "Ok, time still moving forward")
    .endControlFlow()
    .build();
```

生成的代码如下：
```java
void main() {
  long now = System.currentTimeMillis();
  if (System.currentTimeMillis() < now)  {
    System.out.println("Time travelling, woo hoo!");
  } else if (System.currentTimeMillis() == now) {
    System.out.println("Time stood still!");
  } else {
    System.out.println("Ok, time still moving forward");
  }
}
```

### 占位符
&emsp;&emsp;对于字符串的拼接，在使用时是很分散的，太多的拼接的操作符，不太容易观看。为了去解决这个问题，Javapoet提供一个语法，它接收占位符去输出一个文本值，就像 String.format()。


##### $L 常量值

```java
private MethodSpec computeRange(String name, int from, int to, String op) {
  return MethodSpec.methodBuilder(name)
      .returns(int.class)
      .addStatement("int result = 0")
      .beginControlFlow("for (int i = $L; i < $L; i++)", from, to)
      .addStatement("result = result $L i", op)
      .endControlFlow()
      .addStatement("return result")
      .build();
}
```

##### $S 字符串
&emsp;&emsp;对于代码需要包含字符串时，可使用该$S,

```java
public static void main(String[] args) throws Exception {
  TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
      .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
      .addMethod(whatsMyName("slimShady"))
      .addMethod(whatsMyName("eminem"))
      .addMethod(whatsMyName("marshallMathers"))
      .build();

  JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
      .build();

  javaFile.writeTo(System.out);
}

private static MethodSpec whatsMyName(String name) {
  return MethodSpec.methodBuilder(name)
      .returns(String.class)
      .addStatement("return $S", name)
      .build();
}
```

生成的代码如下：
```java
public final class HelloWorld {
  String slimShady() {
    return "slimShady";
  }

  String eminem() {
    return "eminem";
  }

  String marshallMathers() {
    return "marshallMathers";
  }
}
```

##### $T 类
```java
MethodSpec today = MethodSpec.methodBuilder("today")
    .returns(Date.class)
    .addStatement("return new $T()", Date.class)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(today)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```

生成的代码如下：

```java
package com.example.helloworld;

import java.util.Date;

public final class HelloWorld {
  Date today() {
    return new Date();
  }
}
```

&emsp;&emsp;可以看到上面可以导入JDK和SDK的导包，但如果是自定义的包和类呢？这时候就需要用到ClassName；

```java
ClassName hoverboard = ClassName.get("com.mattel", "Hoverboard");
ClassName list = ClassName.get("java.util", "List");
ClassName arrayList = ClassName.get("java.util", "ArrayList");
TypeName listOfHoverboards = ParameterizedTypeName.get(list, hoverboard);

MethodSpec beyond = MethodSpec.methodBuilder("beyond")
    .returns(listOfHoverboards)
    .addStatement("$T result = new $T<>()", listOfHoverboards, arrayList)
    .addStatement("result.add(new $T())", hoverboard)
    .addStatement("result.add(new $T())", hoverboard)
    .addStatement("result.add(new $T())", hoverboard)
    .addStatement("return result")
    .build();
```

生成的代码如下：

```java
package com.example.helloworld;

import com.mattel.Hoverboard;
import java.util.ArrayList;
import java.util.List;

public final class HelloWorld {
  List<Hoverboard> beyond() {
    List<Hoverboard> result = new ArrayList<>();
    result.add(new Hoverboard());
    result.add(new Hoverboard());
    result.add(new Hoverboard());
    return result;
  }
}
```
&emsp;&emsp; JavaPoet 同样支持导入静态包，

```java
ClassName hoverboard = ClassName.get("com.mattel", "Hoverboard");
ClassName list = ClassName.get("java.util", "List");
ClassName arrayList = ClassName.get("java.util", "ArrayList");
TypeName listOfHoverboards = ParameterizedTypeName.get(list, hoverboard);
ClassName namedBoards = ClassName.get("com.mattel", "Hoverboard", "Boards");

MethodSpec beyond = MethodSpec.methodBuilder("beyond")
    .returns(listOfHoverboards)
    .addStatement("$T result = new $T<>()", listOfHoverboards, arrayList)
    .addStatement("result.add($T.createNimbus(2000))", hoverboard)
    .addStatement("result.add($T.createNimbus(\"2001\"))", hoverboard)
    .addStatement("result.add($T.createNimbus($T.THUNDERBOLT))", hoverboard, namedBoards)
    .addStatement("$T.sort(result)", Collections.class)
    .addStatement("return result.isEmpty() ? $T.emptyList() : result", Collections.class)
    .build();

TypeSpec hello = TypeSpec.classBuilder("HelloWorld")
    .addMethod(beyond)
    .build();

JavaFile.builder("com.example.helloworld", hello)
    .addStaticImport(hoverboard, "createNimbus")
    .addStaticImport(namedBoards, "*")
    .addStaticImport(Collections.class, "*")
    .build();
```
&emsp;&emsp;JavaPoet会先导入静态包，然后相应匹配和管理所有的调用，并根据需要导入其它类型：

生成的代码如下：
```java
package com.example.helloworld;

import static com.mattel.Hoverboard.Boards.*;
import static com.mattel.Hoverboard.createNimbus;
import static java.util.Collections.*;

import com.mattel.Hoverboard;
import java.util.ArrayList;
import java.util.List;

class HelloWorld {
  List<Hoverboard> beyond() {
    List<Hoverboard> result = new ArrayList<>();
    result.add(createNimbus(2000));
    result.add(createNimbus("2001"));
    result.add(createNimbus(THUNDERBOLT));
    sort(result);
    return result.isEmpty() ? emptyList() : result;
  }
}

```

##### $N 名字
&emsp;&emsp;有时候生成的代码是我们自己需要引用的，这时候可以使用 $N来调用根据生成的方法名。
```java
public String byteToHex(int b) {
  char[] result = new char[2];
  result[0] = hexDigit((b >>> 4) & 0xf);
  result[1] = hexDigit(b & 0xf);
  return new String(result);
}

public char hexDigit(int i) {
  return (char) (i < 10 ? i + '0' : i - 10 + 'a');
}
```
&emsp;&emsp;比如byteToHex方法需要调用hexDigit方法；就需要用$N；

```java
ethodSpec hexDigit = MethodSpec.methodBuilder("hexDigit")
    .addParameter(int.class, "i")
    .returns(char.class)
    .addStatement("return (char) (i < 10 ? i + '0' : i - 10 + 'a')")
    .build();

MethodSpec byteToHex = MethodSpec.methodBuilder("byteToHex")
    .addParameter(int.class, "b")
    .returns(String.class)
    .addStatement("char[] result = new char[2]")
    .addStatement("result[0] = $N((b >>> 4) & 0xf)", hexDigit)
    .addStatement("result[1] = $N(b & 0xf)", hexDigit)
    .addStatement("return new String(result)")
    .build();
```


### 代码块格式字符串
&emsp;&emsp;代码块可以能通过多种方式来指定其占位符的值，但每个操作只能使用一种方式；比如我们要生成 "I ate 3 tacos"的字符串

##### 相对参数
```java
CodeBlock.builder().add("I ate $L $L", 3, "tacos")

```

##### 位置参数
```java
CodeBlock.builder().add("I ate $2L $1L", "tacos", 3)

```

##### 命名参数
&emsp;&emsp; 还提供$argumentName:X的格式，其中X为占位符， argumentName可以为a-z, A-Z, 0-9, 和 _ ，但必须小写开头；
```java
Map<String, Object> map = new LinkedHashMap<>();
map.put("food", "tacos");
map.put("count", 3);
CodeBlock.builder().addNamed("I ate $count:L $food:L", map)
```

### 抽象方法定义
&emsp;&emsp;可使用Modifiers.ABSTRACT来生成抽象方法：

```java
MethodSpec flux = MethodSpec.methodBuilder("flux")
    .addModifiers(Modifier.ABSTRACT, Modifier.PROTECTED)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addMethod(flux)
    .build();
```

生成的代码如下：

```java
public abstract class HelloWorld {
  protected abstract void flux();
}
```

### 构造方法定义

```java
MethodSpec flux = MethodSpec.constructorBuilder()
    .addModifiers(Modifier.PUBLIC)
    .addParameter(String.class, "greeting")
    .addStatement("this.$N = $N", "greeting", "greeting")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(String.class, "greeting", Modifier.PRIVATE, Modifier.FINAL)
    .addMethod(flux)
    .build();
```

生成的代码如下：
```java
public class HelloWorld {
  private final String greeting;

  public HelloWorld(String greeting) {
    this.greeting = greeting;
  }
}
```

### 参数定义
&emsp;&emsp; 参数可通过ParameterSpec.builder()或者MethodSpec的addParameter()来定义参数：
```java
arameterSpec android = ParameterSpec.builder(String.class, "android")
    .addModifiers(Modifier.FINAL)
    .build();

MethodSpec welcomeOverlords = MethodSpec.methodBuilder("welcomeOverlords")
    .addParameter(android)
    .addParameter(String.class, "robot", Modifier.FINAL)
    .build();
```

生成的代码如下：
```java
void welcomeOverlords(final String android, final String robot) {
}
```

### 字段定义
&emsp;&emsp;字段的可通过 FieldSpec来生成:
```java
FieldSpec android = FieldSpec.builder(String.class, "android")
    .addModifiers(Modifier.PRIVATE, Modifier.FINAL)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(android)
    .addField(String.class, "robot", Modifier.PRIVATE, Modifier.FINAL)
    .build();
```

生成的代码如下：
```java
public class HelloWorld {
  private final String android;

  private final String robot;
}
```

### 接口定义
&emsp;&emsp;对于接口的方法默认必须是 public abstract, 而接口变量默认必须是 public static final; 因此这些都需要显式定义当定义接口时;
```java
TypeSpec helloWorld = TypeSpec.interfaceBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC)
    .addField(FieldSpec.builder(String.class, "ONLY_THING_THAT_IS_CONSTANT")
        .addModifiers(Modifier.PUBLIC, Modifier.STATIC, Modifier.FINAL)
        .initializer("$S", "change")
        .build())
    .addMethod(MethodSpec.methodBuilder("beep")
        .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
        .build())
    .build();
```

生成的代码如下：
```java
public interface HelloWorld {
  String ONLY_THING_THAT_IS_CONSTANT = "change";

  void beep();
}
```

### Enums定义
&emsp;&emsp;Enums定义需要用 enumBuilder来创建enum类型, 然后使用 addEnumConstant()方法添加各自的值;
```java
TypeSpec helloWorld = TypeSpec.enumBuilder("Roshambo")
    .addModifiers(Modifier.PUBLIC)
    .addEnumConstant("ROCK")
    .addEnumConstant("SCISSORS")
    .addEnumConstant("PAPER")
    .build();
```

生成的代码如下：
```java
public enum Roshambo {
  ROCK,

  SCISSORS,

  PAPER
}
```
&emsp;&emsp;当enum值需要重写方法或者调用父类构造方法时,也是支持的:

```java
TypeSpec helloWorld = TypeSpec.enumBuilder("Roshambo")
    .addModifiers(Modifier.PUBLIC)
    .addEnumConstant("ROCK", TypeSpec.anonymousClassBuilder("$S", "fist")
        .addMethod(MethodSpec.methodBuilder("toString")
            .addAnnotation(Override.class)
            .addModifiers(Modifier.PUBLIC)
            .addStatement("return $S", "avalanche!")
            .returns(String.class)
            .build())
        .build())
    .addEnumConstant("SCISSORS", TypeSpec.anonymousClassBuilder("$S", "peace")
        .build())
    .addEnumConstant("PAPER", TypeSpec.anonymousClassBuilder("$S", "flat")
        .build())
    .addField(String.class, "handsign", Modifier.PRIVATE, Modifier.FINAL)
    .addMethod(MethodSpec.constructorBuilder()
        .addParameter(String.class, "handsign")
        .addStatement("this.$N = $N", "handsign", "handsign")
        .build())
    .build();
```

生成的代码如下：
```java
public enum Roshambo {
  ROCK("fist") {
    @Override
    public String toString() {
      return "avalanche!";
    }
  },

  SCISSORS("peace"),

  PAPER("flat");

  private final String handsign;

  Roshambo(String handsign) {
    this.handsign = handsign;
  }
}
```

### 匿名内部类定义
&emsp;&emsp; 与上面enum重写父类方法一样，可使用TypeSpec.anonymousInnerClass()来

```java
//定义匿名内部类
TypeSpec comparator = TypeSpec.anonymousClassBuilder("")
    .addSuperinterface(ParameterizedTypeName.get(Comparator.class, String.class))
    .addMethod(MethodSpec.methodBuilder("compare")
        .addAnnotation(Override.class)
        .addModifiers(Modifier.PUBLIC)
        .addParameter(String.class, "a")
        .addParameter(String.class, "b")
        .returns(int.class)
        .addStatement("return $N.length() - $N.length()", "a", "b")
        .build())
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addMethod(MethodSpec.methodBuilder("sortByLength")
        .addParameter(ParameterizedTypeName.get(List.class, String.class), "strings")
        .addStatement("$T.sort($N, $L)", Collections.class, "strings", comparator)
        .build())
    .build();
```

生成的代码如下：
```java
void sortByLength(List<String> strings) {
  Collections.sort(strings, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
      return a.length() - b.length();
    }
  });
}
```

### 注解定义
&emsp;&emsp; 使用MethodSpec中addAnnotation()方法即可；
```java
MethodSpec toString = MethodSpec.methodBuilder("toString")
    .addAnnotation(Override.class)
    .returns(String.class)
    .addModifiers(Modifier.PUBLIC)
    .addStatement("return $S", "Hoverboard")
    .build();
```

生成的代码如下：
```java
  @Override
  public String toString() {
    return "Hoverboard";
  }
```

&emsp;&emsp；使用AnnotationSpec.builder()来设置批注的注解

```java
MethodSpec logRecord = MethodSpec.methodBuilder("recordEvent")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addAnnotation(AnnotationSpec.builder(Headers.class)
        .addMember("accept", "$S", "application/json; charset=utf-8")
        .addMember("userAgent", "$S", "Square Cash")
        .build())
    .addParameter(LogRecord.class, "logRecord")
    .returns(LogReceipt.class)
    .build();
```

生成的代码如下：
```java
@Headers(
    accept = "application/json; charset=utf-8",
    userAgent = "Square Cash"
)
LogReceipt recordEvent(LogRecord logRecord);
```
### Javadoc定义

```java
MethodSpec dismiss = MethodSpec.methodBuilder("dismiss")
    .addJavadoc("Hides {@code message} from the caller's history. Other\n"
        + "participants in the conversation will continue to see the\n"
        + "message in their own history unless they also delete it.\n")
    .addJavadoc("\n")
    .addJavadoc("<p>Use {@link #delete($T)} to delete the entire\n"
        + "conversation for all participants.\n", Conversation.class)
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .addParameter(Message.class, "message")
    .build();
```

生成的代码如下：
```java
/**
   * Hides {@code message} from the caller's history. Other
   * participants in the conversation will continue to see the
   * message in their own history unless they also delete it.
   *
   * <p>Use {@link #delete(Conversation)} to delete the entire
   * conversation for all participants.
   */
  void dismiss(Message message);
```
