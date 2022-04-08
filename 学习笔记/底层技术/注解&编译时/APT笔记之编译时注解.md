# APT笔记之编译时注解

[TOC]

&emsp;&emsp;注解处理器(APT), 按照处理时期，注解又分为两种类型，一种是运行时注解，另一种是编译时注解; 我们来学习一下很多开源框架都使用编译时注解；

## 1. 基本概念
&emsp;&emsp;在程序编译时，我们需要使用注解处理器来根据注解来生成Java文件（注：生成时，不能修改已存在的Java文件），这些注解处理器生成的Java文件与我们手动写的一样，在后面一起被编译；

&emsp;&emsp;注解处理器是一个在javac中的，用来编译时扫描和处理的注解的工具。你可以为特定的注解，注册你自己的注解处理器。

## 2. 理解 AbstractProcessor
&emsp;&emsp;对于我们自定义的注解处理器都要继承 AbstractProcessor抽象类；AbstractProcessor抽象类实现Processor接口并持有一个本地变量ProcessingEnvironment；

### 2.1 ProcessingEnvironment
&emsp;&emsp; 该接口不需要我们实现，在使用注解处理工具时会传入该对象，因此我们只需要关注其接口返回的工具类就行；
```java
	public interface ProcessingEnvironment {
        /**
         * 返回用来在元素上进行操作的某些实用工具方法的实现。
         *
         * Elements是一个工具类，可以处理相关Element
		 *（包括ExecutableElement, PackageElement, TypeElement, TypeParameterElement, VariableElement）
         */
        Elements getElementUtils();
 
        /**
         * 返回用来报告错误、警报和其他通知的 Messager,简单来说就是打log。
         */
        Messager getMessager();
 
        /**
         *  用来创建新源、类或辅助文件的 Filer。
         */
        Filer getFiler();
 
        /**
         *  返回用来在类型上进行操作的某些实用工具方法的实现；可用来处理TypeMirror的工具类。
         */
        Types getTypeUtils();
 
        // 返回任何生成的源和类文件应该符合的源版本。
        SourceVersion getSourceVersion();
 
        // 返回当前语言环境；如果没有有效的语言环境，则返回 null。
        Locale getLocale();
 
        // 返回传递给注释处理工具的特定于 processor 的选项
        Map<String, String> getOptions();
    }
```
### 2.2 AbstractProcessor的主要方法说明

1. 特殊的**init()**方法:
   它会被注解处理工具调用，并输入ProcessingEnviroment参数。ProcessingEnviroment提供很多有用的工具类Elements,Types和Filer。我们可以通过该方法提前获取工具类；

   ```java
       public synchronized void init(ProcessingEnvironment processingEnv)
   ```

2. **process()**方法：

   每个注解处理器的主函数main()， 你在这里写你的扫描、评估和处理注解的代码，以及生成Java文件。输入参数RoundEnviroment，可以让你查询出包含特定注解的被注解元素。

   ```java
   public abstract boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv);
   ```

   参数说明：
   		**annotations**： 注解处理需要执行一次或者多次。每次执行时，处理器方法被调用，并且传入了当前要处理的注解类型。
           **roundEnv**： 这个对象提供当前或者上一次注解处理中被注解标注的源文件元素。简单点说，就是可以获得所有被标注的元素，无论是类，参数，函数还是变量；

3. **getSupportedAnnotationTypes**()方法:
   用于指定这个注解处理器是注册给哪个注解的。注意，它的返回值是一个字符串的集合，包含本处理器想要处理的注解类型的合法全称；

   ```java
   public Set<String> getSupportedAnnotationTypes()
   ```

4. **getSupportedSourceVersion**()方法：
   用来指定你使用的Java版本。通常这里返回SourceVersion.latestSupported()。然而，如果你有足够的理由只支持Java 7的话，你也可以返回SourceVersion.RELEASE_7。我推荐你使用前者。

   ```java
   public Set<String> getSupportedOptions()
   ```

5. **getSupportedOptions**()方法:
   可通过命令行传递给处理器的操作选项;

   ```java
   public Set<String> getSupportedOptions()
   ```

对于后面三个方法， 在JAVA7及之后，我们不需要再重写该方法，可使用@SupportedAnnotationTypes注解直接定义我们要解释的注解路径，用@SupportedSourceVersion来指定JAVA版本， 如：

```java
//但在JAVA7及之后，我们不需要再重写该方法，可使用@SupportedAnnotationTypes注解直接定义我们要解释的注解路径，用@SupportedSourceVersion来指定JAVA版本， 如：
@SupportedSourceVersion(SourceVersion.RELEASE_7)
@SupportedAnnotationTypes("com.java.yufen.zhi.annotation.ZyfAnnotation")
public class MyProcessor extends AbstractProcessor{
	...
}
```

注：这些方法， 主要用的前面三个， 其它的一般作为了解即可；

## 3. 自定义注解创建及使用
### 3.1 创建相应库及配置
&emsp;&emsp;我们在某应用下，分别创建两个Java Library：zyf-annotation(定义Annotation用)，zyf-compiler(创建相应注解处理器);

其中两库的build.gradle如下：
```groovy
apply plugin: 'java'

ext {
    bintrayName = 'zyf-annotation'
    libraryName = 'zyf-annotation'
    libraryDescription = 'The annotation used in app api'
    libraryVersion = "1.0.0"
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
}

sourceCompatibility = "8"
targetCompatibility = "8"

```

```groovy
apply plugin: 'java'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'
    compileOnly 'com.google.auto.service:auto-service:1.0-rc4'
    compile 'com.squareup:javapoet:1.8.0'
    implementation project(path: ':zyf-annotation')
}

sourceCompatibility = "8"
targetCompatibility = "8"

```

&emsp;&emsp;其中开源库javapoet是Square专门开发用于辅助生成Java文件的库, 该在 java-library中是无法使用的，我们的zyf-compiler在AS用java-library生成后，需要把build.gradle中修改 ： apply plugin: 'java-library' --> apply plugin: 'java'；


&emsp;&emsp;在app的build.gradle中配置好使用这两个模块,以便使用自定义的注解：
```xml
dependencies {
    ...
    compile project(':zyf-annotation')
    annotationProcessor project(path: ':zyf-compiler')
    ...
}
```

### 3.2 创建自定义注解

&emsp;&emsp;在zyf-annotation库中，自定义一个注解：ZyfAnnotation；
```java
package com.java.yufen.zhi.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface ZyfAnnotation {
    String name() default "";
}

```
&emsp;&emsp;该自定义注解是给类注解的(@Target(ElementType.TYPE)), 另外存在Java源文件，以及经编译器后生成的Class字节码文件，但在运行时VM不再保留注释(@Retention(RetentionPolicy.CLASS))；

### 3.3 使用自定义注解

&emsp;&emsp;在 app 模块中使用该注解；

```java
package blackbean.rxjavademo.APT;

import com.java.yufen.zhi.annotation.ZyfAnnotation;

import blackbean.rxjavademo.ARouter.ARouterBaseActivity;

/**
 * Created by Yufen Zhi on 2018/8/18.
 */
@ZyfAnnotation(name = "Jayce")
public class APTTesting extends ARouterBaseActivity{
    private String mBook;
    private int page;

    public APTTesting(String book, int page) {
        this.mBook = book;
        this.page = page;
    }
}
```

### 3.4 创建注解处理器
&emsp;&emsp;上面自定义注解能使用，看似已经完成， 但编译器怎么知道这种注解怎么处理呢， 这就需要创建注解处理器来处理了；

#### 3.4.1 创建注解处理器类
&emsp;&emsp;在zyf-compiler库进行创建 继承于 AbstractProcessor 的 MyProcessor类， 然后在process方法中进行处理注解里的值或方式；

#### 3.4.2 配置注解处理器
&emsp;&emsp;创建好注解处理器后，我们需要告诉编译器在编译的时候使用哪个注解处理器，有两种 方法配置：

&emsp;&emsp;i. 在zyf-compiler的main目录下创建目录resources/META-INF/services,并在该目录下创建文件javax.annotation.processing.Processor，在该文件中写入注解处理器的全限定类名(这里是com.java.yufen.zhi.compiler.MyProcessor)；

&emsp;&emsp;ii. 通过Google提供的auto-service开源库，通过该方式我们只需要在注解处理器上添加注解@AutoService(Processor.class)即可。
```
//build.gradle
dependencies {
    ....
    compile 'com.google.auto.service:auto-service:1.0-rc3'
    compile 'com.google.auto:auto-common:0.8'
	...
}
//Process
@AutoService(Process.class)
public class MyProcessor extends AbstractProcessor{
	....
}

```

整个MyProcessor类的代码如下：

```java

@AutoService(Processor.class)
@SupportedSourceVersion(SourceVersion.RELEASE_7)
@SupportedAnnotationTypes("com.java.yufen.zhi.annotation.ZyfAnnotation")
public class MyProcessor extends AbstractProcessor{

    //来处理TypeMirror的工具类
    private Types mTypeUtils;
	
	//用于{@link Element}处理的工具类
    private Elements mElementUtils;
	
	//用于文件创建
    private Filer mFiler;
	
	//可用于编译时，信息打印
    private Messager mMessager;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);

        //Initial the tool
        mTypeUtils = processingEnvironment.getTypeUtils();
        mElementUtils = processingEnvironment.getElementUtils();
        mFiler = processingEnvironment.getFiler();
        mMessager = processingEnvironment.getMessager();
        mMessager.printMessage(Diagnostic.Kind.NOTE,"[yufen]init......");
    }

    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        mMessager.printMessage(Diagnostic.Kind.NOTE,"[yufen]process..start");
        Set<? extends Element> annotatedElements = roundEnvironment.getElementsAnnotatedWith(ZyfAnnotation.class);

        for (Element element : annotatedElements) {
            mMessager.printMessage(Diagnostic.Kind.NOTE,"[yufen]analysisAnnotation");
            analysisAnnotation(element);
        }

        mMessager.printMessage(Diagnostic.Kind.NOTE,"[yufen]process..finish");
        return true;
    }

    private void analysisAnnotation(Element element) {
        ZyfAnnotation annotation = element.getAnnotation(ZyfAnnotation.class);
        //通过ZyfAnnotation可获取该注解的相应元素的值
        String value = annotation.name();

        //获取包名
        String packageName = mElementUtils.getPackageOf(element).getQualifiedName().toString();
        //定义方法
        MethodSpec mMethod = MethodSpec.methodBuilder("GetMessage")
                .addModifiers(Modifier.PUBLIC)
                .addStatement("return $S", value)
                .returns(String.class)
                .build();

        //生成类
        TypeSpec mClass = TypeSpec.classBuilder(value + "Activity$$Steven")
                .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
                .addMethod(mMethod)
                .build();

        //生成一个厅级的文件描述对象
        JavaFile javaFile = JavaFile.builder(packageName, mClass)
                .build();

        //输出文件
        try {
            javaFile.writeTo(mFiler);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

&emsp;&emsp; 这时候我们对整个app进行Rebuild, 在编译的log中，我们可以看到我们在注解处理器类中通过mMessager输出的Log：

PS: 上面在analysisAnnotation方法中， 我们在获取注解的相关信息后使用Javapoet框架生成的代码，后面会介绍该框架的使用；

```
注: ARouter::Compiler >>> Generated root, name is ARouter$$Root$$app <<<
注: [yufen]init......
注: [yufen]process..start
注: [yufen]analysisAnnotation
注: [yufen]analysisAnnotation
注: [yufen]process..finish
注: [yufen]process..start
注: [yufen]process..finish
注: [yufen]process..start
注: [yufen]process..finish
注: 某些输入文件使用或覆盖了已过时的 API。
```
&emsp;&emsp;另外在MyProcessor注解处理器中，我们对@ZyfAnnotation注解的处理，以注解里的值（比如“XXX”）来创建命名为 “XXXActivity$$Steven"的类, 同时创建GetMessage方法来返回”XXX“的字符串；

&emsp;&emsp; 在 使用@ZyfAnnotation的app模块中， 在路径： app\build\generated\source\debug 下就能找到自动生成的类，该类也会一起编译最终的apk；
```java
//D:\Learn_code\app\build\generated\source\apt\debug\blackbean\rxjavademo\APT\JayceActivity$$Steven.java
package blackbean.rxjavademo.APT;

import java.lang.String;

public final class JayceActivity$$Steven {
  public String GetMessage() {
    return "Jayce";
  }
}
```

本文笔记来自于下面大神的文章：

> https://blog.csdn.net/kaifa1321/article/details/79683246
> https://www.jianshu.com/p/07ef8ba80562
> https://blog.csdn.net/industriously/article/details/53932425
> https://blog.csdn.net/kaifa1321/article/details/79622715

