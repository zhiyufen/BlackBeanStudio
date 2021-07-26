# [Unit Test] Mockito学习笔记

[TOC]

![image](https://www.javadoc.io/static/org.mockito/mockito-core/2.23.4/org/mockito/logo.png)

Mockito 是一个用于 Java 单测的 Mock 框架，除了 JUnit 之外，它还可以用于其他单测框架（例如：TestNG）。`Mockito` 通过mock可以模拟各种各样的对象，来代替真正的对象做出希望的响应，能够让我们更加专注地去测试代码逻辑，省去了构造数据所花费的努力。

## 1. 基本使用

### 1.1 配置

V2.6.1上Mockito不再包含 Android支持的，如需要支持，则额外添加上mockito-android； 

```groovy
 repositories {
   jcenter()
 }
 dependencies {
   testCompile "org.mockito:mockito-core:+"
   androidTestCompile "org.mockito:mockito-android:+"
 }
```

### 1.2 基本概念

Mock 可分为两种类型，一种是 **Class Mock**，另一种是 **Partial Mock(Spy)**。两种都能模拟对象；

改变 mock 对象方法（method）的行为叫 **Stub**（插桩），比如修改某方法返回的值； 

一次 Mock 过程称为 **Mock Session**，它会记录所有的 **Stubbing**，基本包含如下三个步骤：

```xml
+----------+      +------+      +--------+
| Mock/Spy | ===> | Stub | ===> | Verify |
+----------+      +------+      +--------+
```

#### 1.2.1 Class Mock

Class Mock 改变了 class 的行为，所以 mock 出来的对象就完全失去了原来的行为， 比如你通过Mock对象调用一个方法，并不会直接运行该方法，只是Mock对象会记录下这次行为(stubbing);
如果没有对 method 进行插桩(Stub)，那么 method 会返回默认值（`null`、`false`、`0`等）。

```java
//Let's import Mockito statically so that the code looks clearer
 import static org.mockito.Mockito.*;

 // 利用 List.class 创建一个 mock 对象 --- mockedList
 List mockedList = mock(List.class);

 //using mock object
 mockedList.add("one");
 mockedList.clear();

 //验证mockedList进行是否调用下面2个方法
 verify(mockedList).add("one");
 verify(mockedList).clear();
```

#### 1.2.2 Partial Mock(Spy)

如果只是想改变一个实例（instance）的行为，我们需要使用 `spy`;

```java
   List list = new LinkedList();
   List spy = spy(list);

   //optionally, you can stub out some methods:
   //改变size方法的返回值为100
   when(spy.size()).thenReturn(100);

   //using the spy calls *real* methods
   spy.add("one");
   spy.add("two");

   //prints "one" - the first element of a list
   System.out.println(spy.get(0));

   //size() method was stubbed - 100 is printed
   System.out.println(spy.size());

   //optionally, you can verify
   verify(spy).add("one");
   verify(spy).add("two");
```

如果方法没有被插桩，那么它的行为就不会被改变，所以有些情况下，我们不能使用 `when(Object)` 进行插桩，只能使用 do 系列方法（`doReturn|Answer|Throw()`）：

```java
   List list = new LinkedList();
   List spy = spy(list);

   //Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
   when(spy.get(0)).thenReturn("foo");

   //You have to use doReturn() for stubbing
   doReturn("foo").when(spy).get(0);
```

#### 1.2.3 Class Mock Vs Spy

`mock` : 没有被 stub 的方法会返回一个默认值, 并不会真实运行方法里的语句；

`spy` : 如果一个方法没有被 stub，那么会执行它真实的行为。

#### 1.2.4 Stub（插桩）

仅仅 Mock 出来一个对象显然是不够的，虽然可以通过验证**方法的执行情况**来测试代码的逻辑，但是多数情况下我们还需要改变方法的返回值，这时就需要用到“插桩”。

Stubbing 表示一次插桩，它的形式是 `when(x).then(y)`，用于指定 mock 的行为。

```java
when(mock.foo()).thenReturn(true);
```

可以通过如下代码获取 mock 对象的所有 stubbing：

```java
Mockito.mockingDetails(mock).getStubbings();
```

Stubbing 的接口规范如下：

```java
public interface Stubbing {
    Invocation getInvocation();
    boolean wasUsed();
}
```

- `getInvocation()` 返回被插桩的方法调用，例如，`mock.foo()` 就是一个 `Invocation`。
- `wasUsed()` 用于表示 stubbing 是否被使用，如果 `mock.foo()` 没有被调用，那么 `wasUsed()` 返回 false，说明存在未被使用的 stubbing，为了保持 `clarity of tests`，最好删除未被使用的 stubbing 代码。

#### 1.2.4 Verify（验证）

Verify 用于验证 mock 行为。Mockito 提供了两个 `verify` 方法：

```java
public static <T> T verify(T mock, VerificationMode mode) {
  return MOCKITO_CORE.verify(mock, mode);
}

// times(1) 是 VerificationMode 的默认值
public static <T> T verify(T mock) {
  return MOCKITO_CORE.verify(mock, times(1));
}
```

可以看到verify(T mock)是默认验证调用1次的；

```java
//只调用几次
verify(mock, times(2)).someMethod("some arg");

//验证从来没调用
verify(mock, never()).someMethod();

//验证至少调用过几次
verify(mock, atLeastOnce()).someMethod("some arg");
verify(mock, atLeast(3)).someMethod("some arg");

//验证最多调用过几次
verify(mock, atMost(3)).someMethod("some arg");

//顺序验证//此验证模式只能用于顺序验证。
inOrder.verify(mock, calls(2)).someMethod( "some arg" );

//只调用一次
verify(mock, only()).someMethod();
//above is a shorthand for following 2 lines of code:
verify(mock).someMethod();
verifyNoMoreInvocations(mock);

//验证在给定时间调用某方法多少次
//passes when someMethod() is called within given time span
verify(mock, timeout(100)).someMethod();
//above is an alias to:
verify(mock, timeout(100).times(1)).someMethod();
//passes as soon as someMethod() has been called 2 times before the given timeout
verify(mock, timeout(100).times(2)).someMethod();
//equivalent: this also passes as soon as someMethod() has been called 2 times before the given timeout
verify(mock, timeout(100).atLeast(2)).someMethod();
//verifies someMethod() within given time span using given verification mode
//useful only if you have your own custom verification modes.
verify(mock, new Timeout(100, yourOwnVerificationMode)).someMethod();

//
//passes after 100ms, if someMethod() has only been called once at that time.
verify(mock, after(100)).someMethod();
//above is an alias to:
verify(mock, after(100).times(1)).someMethod();
//passes if someMethod() is called *exactly* 2 times after the given timespan
verify(mock, after(100).times(2)).someMethod();
//passes if someMethod() has not been called after the given timespan
verify(mock, after(100).never()).someMethod();
//verifies someMethod() after a given time span using given verification mode
//useful only if you have your own custom verification modes.
verify(mock, new After(100, yourOwnVerificationMode)).someMethod();
```

**timeout() vs. after()**

- timeout() 会马上退出当验证通过后(不用管是否到达超时时间)；
- after() 会完全等待超时时间后才退出；

#### 1.2.5 ArgumentMatcher

插桩和验证时除了指定特定的参数，为了更灵活，还可以使用通配符 —— `ArgumentMatcher`。

注：一旦某个方法的参数使用了参数匹配，则该方法所有的参数都得使用参数匹配

```java
//stubbing using built-in anyInt() argument matcher
 thenmockedList.get(anyInt())).thenReturn("element");

 //stubbing using custom matcher (let's say isValid() returns your own matcher implementation):
 when(mockedList.contains(argThat(isValid()))).thenReturn("element");

 //following prints "element"
 System.out.println(mockedList.get(999));

 //you can also verify using an argument matcher
 verify(mockedList).get(anyInt());

 //argument matchers can also be written as Java 8 Lambdas
 verify(mockedList).add(argThat(someString -> someString.length() > 5));
```

自定义参数匹配： ListOfTwoElements；

```java
class ListOfTwoElements implements ArgumentMatcher<List> {
  public boolean matches(List list) {
    return list.size() == 2;
  }

  public String toString() {
    //printed in verification errors
    return "[list of 2 elements]";
  }
}

List mock = mock(List.class);

when(mock.addAll(argThat(new ListOfTwoElements))).thenReturn(true);

mock.addAll(Arrays.asList("one", "two"));

verify(mock).addAll(argThat(new ListOfTwoElements()));
//简化上面
verify(mock).addAll(listOfTwoElements());
```

更多通配符可参考： `ArgumentMatcher`：

```java
public class ArgumentMatchers {
  public static <T> T any();
  public static <T> T any(Class<T> type);
  public static <T> T isA(Class<T> type);
  public static boolean anyBoolean();
  public static byte anyByte();
  public static char anyChar();
  public static int anyInt();
  public static long anyLong();
  public static float anyFloat();
  public static double anyDouble();
  public static short anyShort();
  public static String anyString();

  public static <T> List<T> anyList();
  public static <T> List<T> anyListOf(Class<T> clazz);
  public static <T> Set<T> anySet();
  public static <T> Set<T> anySetOf(Class<T> clazz);
  public static <K, V> Map<K, V> anyMap();
  public static <K, V> Map<K, V> anyMapOf(Class<K> keyClazz, Class<V> valueClazz);
  public static <T> Collection<T> anyCollection();
  public static <T> Collection<T> anyCollectionOf(Class<T> clazz);
  public static <T> Iterable<T> anyIterable();
  public static <T> Iterable<T> anyIterableOf(Class<T> clazz);

  public static boolean eq(boolean value);
  public static byte eq(byte value);
  public static char eq(char value);
  public static double eq(double value);
  public static float eq(float value);
  public static int eq(int value);
  public static long eq(long value);
  public static short eq(short value);
  public static <T> T eq(T value);
  public static <T> T refEq(T value, String... excludeFields);
  public static <T> T same(T value)

  public static <T> T isNull();
  public static <T> T isNull(Class<T> clazz);
  public static <T> T notNull();
  public static <T> T notNull(Class<T> clazz);
  public static <T> T isNotNull();
  public static <T> T isNotNull(Class<T> clazz);
  public static <T> T nullable(Class<T> clazz);

  // String argument matcher
  public static String contains(String substring);
  public static String matches(String regex);
  public static String matches(Pattern pattern);
  public static String endsWith(String suffix);
  public static String startsWith(String prefix);

  // Custom argument matcher
  public static <T> T argThat(ArgumentMatcher<T> matcher);
  public static char charThat(ArgumentMatcher<Character> matcher);
  public static boolean booleanThat(ArgumentMatcher<Boolean> matcher);
  public static byte byteThat(ArgumentMatcher<Byte> matcher);
  public static short shortThat(ArgumentMatcher<Short> matcher);
  public static int intThat(ArgumentMatcher<Integer> matcher);
  public static long longThat(ArgumentMatcher<Long> matcher);
  public static float floatThat(ArgumentMatcher<Float> matcher);
  public static double doubleThat(ArgumentMatcher<Double> matcher);
}
```

## 2. 进阶使用

#### 2.1 doReturn()|doThrow()| doAnswer()|doNothing()|doCallRealMethod() 

对于返回为void的方法，无法使用 when(...).thenxx()来进行stubbing，

下面情况时，可使用 doThrow()`, `doAnswer()`, `doNothing()`, `doReturn()` and `doCallRealMethod() 来stubbing：

- stub Void 方法时；
- stub 使用spy对象的方法时；
- stub 已经被stub过不只一次的方法，以便在测试中修改其行为；

##### doReturn

设定某void方法返回结果, 或者有些情况下无法使用when来设置返回结果时；

stub 使用spy对象的方法时； 

```java
   List list = new LinkedList();
   List spy = spy(list);

   //Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
   when(spy.get(0)).thenReturn("foo");

   //You have to use doReturn() for stubbing:
   doReturn("foo").when(spy).get(0);
```

stub 已经被stub过不只一次的方法，需要使用doReturn来再次更改返回值 ，以便在测试中修改其行为；

```java
   when(mock.foo()).thenThrow(new RuntimeException());

   //Impossible: the exception-stubbed foo() method is called so RuntimeException is thrown.
   when(mock.foo()).thenReturn("bar");

   //You have to use doReturn() for stubbing:
   doReturn("bar").when(mock).foo();
```

##### doThrow

设定某void方法调用时抛出某个异常：

```java
 doThrow(RuntimeException.class).when(mock).someVoidMethod();
```

定义多次调用时抛出异常：

```java
 //BigFailure异常会在第二次调用时，抛出
 doThrow(RuntimeException.class, BigFailure.class).when(mock).someVoidMethod();
```

##### doCallRealMethod

设定某void方法调用时可运行其内部真正的逻辑；

一般来说，要真实运行方法里的语句，最好使用 spy 方式进行，但有时候针对第三方接口等时，用它会更简单；

```java
  Foo mock = mock(Foo.class);
   doCallRealMethod().when(mock).someVoidMethod();

   // this will call the real implementation of Foo.someVoidMethod()
   mock.someVoidMethod();
```

##### doAnswer

设定某void方法的返回值(自定义):

```java
  doAnswer(new Answer() {
      public Object answer(InvocationOnMock invocation) {
          Object[] args = invocation.getArguments();
          Mock mock = invocation.getMock();
          return null;
      }})
  .when(mock).someMethod();
```

##### doNothing

设定某void方法调用时不会运行方法内任何语句； 

需要注意的是，正常ClassMock时，void方法默认就是不会运行方法内任何语句， 但有些情况，这个方法还是很方便的：

在void方法上连续调用：

```java
   doNothing().
   doThrow(new RuntimeException())
   .when(mock).someVoidMethod();

   //does nothing the first time:
   mock.someVoidMethod();

   //throws RuntimeException the next time:
   mock.someVoidMethod();
```

当您监视真实对象(spy)并且希望void方法不执行任何操作时：

```java
 List list = new LinkedList();
   List spy = spy(list);

   //let's make clear() do nothing
   doNothing().when(spy).clear();

   spy.add("one");

   //clear() does nothing, so the list still contains "one"
   spy.clear();
```

#### 2.2 顺序验证(Verification in order)

有时候我们需要确保某对象的方法是按我们的方法进行一步步运行的；

```java
 // A. Single mock whose methods must be invoked in a particular order
 List singleMock = mock(List.class);

 //using a single mock
 singleMock.add("was added first");
 singleMock.add("was added second");

 //create an inOrder verifier for a single mock
 InOrder inOrder = inOrder(singleMock);

 //following will make sure that add is first called with "was added first", then with "was added second"
 inOrder.verify(singleMock).add("was added first");
 inOrder.verify(singleMock).add("was added second");

 // B. Multiple mocks that must be used in a particular order
 List firstMock = mock(List.class);
 List secondMock = mock(List.class);

 //using mocks
 firstMock.add("was called first");
 secondMock.add("was called second");

 //create inOrder object passing any mocks that need to be verified in order
 InOrder inOrder = inOrder(firstMock, secondMock);

 //following will make sure that firstMock was called before secondMock
 inOrder.verify(firstMock).add("was called first");
 inOrder.verify(secondMock).add("was called second");

 // Oh, and A + B can be mixed together at will
```

按顺序验证是灵活的，你不需要验证所有交互，只需要验证你需要的交互即可； 

#### 2.3 verifyZeroInteractions

##### 验证某交互没发生过

```java
	@Test
	public void testMockito7() {
		List mockOne = mock(List.class);
		List mockTwo = mock(List.class);
		List mockThree = mock(List.class);
		
		// mockOne和mockTwo都有调用，mockThree没有
		mockOne.add("one");
		mockTwo.add("two");
 
		// 普通的调用验证
		verify(mockOne).add("one");
 
		// add.("two")从未被调用过
		verify(mockOne, never()).add("two");
 
		// 下面的验证会失败，因为mockOne和mockTwo都有调用
		verifyZeroInteractions(mockOne, mockTwo);
		
		// 下面验证会成功，因为mockThree没有过调用
		verifyZeroInteractions(mockOne, mockThree);
    }
```

##### 查找冗余的交互

```java
//using mocks
 mockedList.add("one");
 mockedList.add("two");

 verify(mockedList).add("one");

 //following verification will fail
 verifyNoMoreInteractions(mockedList);
```

mockedList由于add("two")是冗余调用，所以verifyNoMoreInteractions会失败；

注： verifyNoMoreInteractions（）是交互测试工具箱中一个方便的断言。只有在相关的时候才使用它。滥用它会导致过度指定、不易维护的测试。

#### 2.5 MockitoAnnotation

MockitoAnnotations负责初始化`@Mock`、`@Spy`、`@Captor`、`@InjectMocks`等注解,  这些注解能帮忙初始化相应的Mock对象；

使用注解可带来如下好处：

- 代码更简洁
- 避免重复创建
- 可读性好
- 验证错误更易读（因为注解默认使用field name来标记mock对象）

```java
@Mock
private ArrayList mockedList;

@Before
public void setUp() throws Exception {
    MockitoAnnotations.initMocks(this);
}
```

注：使用注解对象之前，必须确保调用MockitoAnnotations.initMocks(this);进行相关初始化；

除非调用 initMocks()方法进行初始化， 也可使用一些扩展插件(JUnit5), 就可以使用MockitoJUnitRunner 或者 MockitoRule 进行初始化；

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoJUnitRunnerTest {
    @Mock
    AccountData accountData;
}

public class MockitoRuleTest {
    @Mock
    AccountData accountData;
    @Rule
    public MockitoRule mockitoRule = MockitoJUnit.rule();
}
```

其它注解也是一样的用法；

#### 迭代式插桩

有时候我们对同个方法连续插桩让其返回不同的结果； 