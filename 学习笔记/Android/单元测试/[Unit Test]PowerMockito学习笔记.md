## PowerMockito学习笔记

[TOC]

PowerMockito为了解决EasyMock,Mockito等工具的缺点： 不能mock静态、final、私有方法等；而PowerMock能够完美的弥补其不足。

PowerMock是一个扩展了其它如EasyMock等mock框架的、功能更加强大的框架。PowerMock使用一个自定义类加载器和字节码操作来模拟静态方法，构造函数，final类和方法，私有方法，去除静态初始化器等等。

### 基本实现原理：

当某个测试方法被注解@PrepareForTest标注以后，在运行测试用例时，会创建一个新的org.powermock.core.classloader.MockClassLoader实例，然后加载该测试用例使用到的类（系统类除外）。

PowerMock会根据你的mock要求，去修改写在注解@PrepareForTest里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除final方法的final标识，在静态方法的最前面加入自己的虚拟实现等。

如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class文件，而是修改调用系统类的class文件，以满足mock需求。

### 配置

```groovy
    //powerMockito
    testImplementation "org.powermock:powermock-module-junit4:2.0.9"
    testImplementation "org.powermock:powermock-module-junit4-rule:2.0.9"
    testImplementation "org.powermock:powermock-api-mockito2:2.0.9"
    testImplementation "org.powermock:powermock-classloading-xstream:2.0.9"
```



### 基本用法

#### 注解

PowerMock有两个重要的注解：

- @RunWith(PowerMockRunner.class)
- @PrepareForTest( { YourClassWithEgStaticMethod.class })

你需要mock的对象，都要写进@PrepareForTest的列表参数里；

#### Mock方法内部new出来的对象

```java
//测试目标代码 Source.java：
public boolean callInternalInstance(String path) {
    File file = new File(path);
    return file.exists();
}
  
//测试用例代码：   
@Test
@PrepareForTest(Source.class)
public void testCallInternalInstance() throws Exception
{
    File file = PowerMockito.mock(File.class);
    Source underTest = new Source();
    PowerMockito.whenNew(File.class).withArguments("bbb").thenReturn(file);
    PowerMockito.when(file.exists()).thenReturn(true);
    Assert.assertTrue(underTest.callInternalInstance("bbb"));
}
```

注：当使用PowerMockito.whenNew方法时，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是需要mock的new对象代码所在的类。

#### Mock普通对象的final方法

```java
//测试目标代码：
public class Source {
    public boolean callFinalMethod(SourceDepend refer) {
        return refer.isAlive();
    }
}
public class SourceDepend {
    public final boolean isAlive() {
        return false;
    }
}
  
//测试用例代码：
@Test
@PrepareForTest(SourceDepend.class)
public void testCallFinalMethod()
{
    SourceDepend depencency = PowerMockito.mock(SourceDepend.class);
    Source underTest = new Source();
    PowerMockito.when(depencency.isAlive()).thenReturn(true);
    Assert.assertTrue(underTest.callFinalMethod(depencency));
}
```

注：当需要mock final方法的时候，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是final方法所在的类。 

#### Mock普通类的静态方法

```java
//测试目标代码 SourceDepend.java：
public boolean callStaticMethod() {
      return SourceDepend.isExist();
}
public static boolean isExist()
{
    return false;
}
 
//测试用例代码：
@Test
@PrepareForTest(SourceDepend.class)
public void testCallStaticMethod() {
    Source underTest = new Source();
    PowerMockito.mockStatic(SourceDepend.class);
    PowerMockito.when(SourceDepend.isExist()).thenReturn(true);
    Assert.assertTrue(underTest.callStaticMethod());
}
```

注：当需要mock静态方法的时候，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是静态方法所在的类。

#### Mock 私有方法

```java
//测试目标代码：
public boolean callPrivateMethod() {
    return isExist();
}
 
private boolean isExist() {
    return false;
}

//测试用例代码： 
@Test
@PrepareForTest(Source.class)
public void testCallPrivateMethod() throws Exception
{
    Source underTest = PowerMockito.mock(Source.class);
    PowerMockito.when(underTest.callPrivateMethod()).thenCallRealMethod();
    PowerMockito.when(underTest, "isExist").thenReturn(true);
    Assert.assertTrue(underTest.callPrivateMethod());
}
```

注：和Mock普通方法一样，只是需要加注解@PrepareForTest(ClassUnderTest.class)，注解里写的类是私有方法所在的类。 

#### Mock系统类的静态和final方法 

```java
//测试目标代码：  
public boolean callSystemFinalMethod(String str)
{
    return str.isEmpty();
}
 
public String callSystemStaticMethod(String str)
{
    return System.getProperty(str);
}
  
//测试用例代码：
@Test
@PrepareForTest(Source.class)
public void testCallSystemStaticMethod()
{
    Source underTest = new Source();
    PowerMockito.mockStatic(System.class);
    PowerMockito.when(System.getProperty("aaa")).thenReturn("bbb");
    Assert.assertEquals("bbb", underTest.callSystemStaticMethod("aaa"));
}
```

注：和Mock普通对象的静态方法、final方法一样，只不过注解@PrepareForTest里写的类不一样 ，注解里写的类是需要调用系统方法所在的类。

本文笔记来自于： https://www.cnblogs.com/hunterCecil/p/5721468.html