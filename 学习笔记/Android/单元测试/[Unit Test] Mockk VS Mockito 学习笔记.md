# Mockk VS Mockito 学习笔记

[Toc]

在我们使用Mockito写kotlin代码的单元测试，会遇到很多问题，或者说写得很不顺，其实可使用mockk来写Kotlin的单元测试代码；

这里仅仅说明Mockk与Mockito的区别，不熟悉Mockito的可能会看不懂。

## 一、Mockito存在的问题

- Mockito 不支持对 final class、匿名内部类以及基本类型（如 int）的 mock。
- Mockito 不支持对静态方法、 final 方法、私有方法、equals() 和 hashCode() 方法进行 mock。
- Kotlin中的类默认是 final, 除非 我们进行改为 open，否则无法Mock； Mockito中的when方法与Kotlin的when关键字冲突，因此写成 'when' ; 可读性有点差；

## 二、Mockk VS Mockito的用法

MockK 是一个用 Kotlin 写的 Mocking 框架，它解决了所有上述提到的 Mockito 中存在的问题。

配置：

```groovy
testImplementation "io.mockk:mockk:1.9.3"
```

更多信息请参考：[Mockk官网](https://mockk.io/)
下面主要写 mockk与 mockito的用法区别；

### 1.  Mock对象

对于普通的class类， 两者的创建方法类似：

```kotlin
// Mockito
val mockedFile = mock(File::class.java)

mockedFile.read() // does nothing
```

```kotlin
// MockK
val mockedFile = mockk<File>()

mockedFile.read() // throws because the method was not stubbed
```

#### Relaxed mock

mockk来mock对象，默认使用mockito中 [`STRICT_STUBS`](https://javadoc.io/static/org.mockito/mockito-core/3.3.3/org/mockito/quality/Strictness.html#STRICT_STUBS) 模式，也就是如果你一个方法没有进行stub的话，会进行抛出，这样可以更容易地捕获在您不期望调用的方法时调用的方法，或者在使用不同参数调用方法时调用的方法。

Mockito默认的宽大行为可以通过宽松的设置来复制。轻松的mock将为所有方法提供默认存根。

```kotlin
// 也使用使用 @RelaxedMockK 注解
val mockedFile = mockk<File>(relaxed = true)

mockedFile.read() // will not throw
```

或使用relaxUnitFun 来仅仅返回值为空的方法进行宽松：

```
// MockK
val mockedFile = mockk<File>(relaxUnitFun = true)

mockedFile.read() // returns Unit, will not throw
mockedFile.exists() // throws as the method returns Boolean
```

###  2. Stubing

Mockito 提供 when和 do系列方法来对一个方法进行stubing; 

对于Mockito， 一般使用是：

```kotlin
val mockedFile = mock(File::class.java)

`when`(mockedFile.read()).thenReturn("hello world")

//有时不能使用上面的方法,比如 void 方法
val mockedFile = mock(File::class.java)

doReturn("hello world").`when`(mockedFile).read()
```

而对于 Mockk来说， 所有stubing都可能 使用 every 方法， 比如：

```kotlin
val mockedFile = mockk<File>()

every { mockedFile.read() } returns "hello world"
```

#### 2.1 thenReturn/doReturn

进行stubing返回值的：
Mockito:

```kotlin
val mockedFile = mock(File::class.java)

`when`(mockedFile.read()).thenReturn("hello world")
doReturn("hello world").`when`(mockedFile).read()
```

而Mockk中使用中缀函数：

```kotlin
val mockedFile = mockk<File>()

every { mockedFile.read() } returns "hello world"
```

#### 2.2 thenThrow/doThrow

除了返回值，有时需要stubing来抛出某个异常：

Mcokito:

```kotlin
val mockedFile = mock(File::class.java)

`when`(mockedFile.read()).thenThrow(RuntimeException())
doThrow(RuntimeException()).`when`(mockedFile).read()
```

Mockk:

```kotlin
val mockedFile = mockk<File>()

every { mockedFile.read() } throws RuntimeException()
```

#### 2.3 doNothing

对于返回  void 的方法来说，在Mockito里， when是不起作用的；针对 void 方法，会用 doNothing 来定义该方法不执行方法内的语句：

```kotlin
val mockedFile = mock(File::class.java)

doNothing().`when`(mockedFile).write(any())
```

对于Mockk来说，有2种方法来处理这样的：

```kotlin
val mockedFile = mockk<File>()

every { mockedFile.write(any()) } returns Unit
```

```kotlin
val mockedFile = mockk<File>()

every { AppleDemoCardAgent.postDemoCard(mContext, intent) } just Runs
```

#### 2.4 thenAnswer/then/doAnswer

Mockito 可 Answer接口来自定义方法的处理逻辑：

```kotlin
val mockedFile = mock(File::class.java)

`when`(mockedFile.write(any())).thenAnswer { invocation ->
  println("called with arguments: " + invocation.arguments.joinToString())
  Unit
}
`when`(mockedFile.write(any())).then { invocation ->
  println("called with arguments: " + invocation.arguments.joinToString())
  Unit
}
doAnswer { invocation ->
  println("called with arguments: " + invocation.arguments.joinToString())
  Unit
}.`when`(mockedFile).write(any())
```

Mockk：

```kotlin
val mockedFile = mockk<File>()

every { mockedFile.write(any()) } answers { call ->
  println("called with arguments: " + call.invocation.args.joinToString())
  Unit
}
```

#### 2.5 Consecutive calls(迭代式)

Mockito提供的方法：

```kotlin
val mockedFile = mock(File::class.java)

// Chain multiple calls
`when`(mockedFile.read()).thenReturn("read 1").thenReturn("read 2").thenReturn("read 3")
// Shorthand
`when`(mockedFile.read()).thenReturn("read 1", "read 2", "read 3")
doReturn("read 1", "read 2", "read 3").`when`(mockedFile).read()
// Use different answer types
`when`(mockedFile.read())
  .thenReturn("successful read")
  .thenThrow(RuntimeException())
```

Mockk:

```kotlin
val mockedFile = mockk<File>()

// Chain multiple calls
every { mockedFile.read() } returns "read 1" andThen "read 2" andThen "read 3"
// Shorthand using a list
every { mockedFile.read() } returnsMany listOf("read 1", "read 2", "read 3")
every { mockedFile.read() } andThenMany listOf("read 1", "read 2", "read 3")
// Use different answer types
every { mockedFile.read() } returns "successful read" andThenThrows RuntimeException()
```

#### 2.6 协程

由于Mockk是使用函数文本来stub的，因此进行stub挂起方法时，需要进行一些小改动；

Mockk提供了前缀为 **co** 的函数来作为其它函数的等价， 比如 coEvery{} 对应的 every {}；

```kotlin
val mockedFile = mockk<File>()

coEvery { mockedFile.readAsync() } returns "hello world"
coEvery { mockedFile.writeAsync(any()) } coAnswers { call ->
  doAsyncWork()
  Unit
}
```

 其它：coEvery/coJustRun/coVerify/coVerifyAll/coVerifyOrder/coVerifySequence/coExcludeRecords/coMatch/coMatchNullable/coWithArg/coWithNullableArg/coAnswers/coAndThen/coInvoke

### 3. any*

Mockito中的有一系列的参数匹配，如： anyBoolean(), anyInt(), anyList(); 但在Mockk中，一切使用 any() 来匹配；

如果需要判断 是为 null 或不为 null 值的，则使用 isNull 匹配符： isNull()`, `isNull(inverse = true)

### 4. verify

验证某个方法是否调用：
Mockito:

```kotlin
// Mockito
val mockedFile = mock(File::class.java)

mockedFile.read()
verify(mockedFile).read()
```

Mockk:

```kotlin
// MockK
val mockedFile = mockk<File>()

mockedFile.read()
verify { mockedFile.read() }
```

#### 4.1 验证模式

**未调用过某个方法(never)**
Mockito:

```kotlin
// Mockito
verify(mockedFile, never()).write()
```

Mockk:

```kotlin
// MockK
verify(inverse = true) { mockedFile.write() }
```

**至少调用几次(atLeast/atLeastOnce)**

Mockito:

```kotlin
// Mockito
verify(mockedFile, atLeast(3)).read()
verify(mockedFile, atLeastOnce()).write()
```

Mockk:

```
// MockK
verify(atLeast = 3) { mockedFile.read() }
verify(atLeast = 1) { mockedFile.write() }
```

**最多调用几次（atMost/atMostOnce）**

Mockito:

```kotlin
// Mockito
verify(mockedFile, atMost(3)).read()
verify(mockedFile, atMostOnce()).write()
```

Mockk：

```
// MockK
verify(atMost = 3) { mockedFile.read() }
verify(atMost = 1) { mockedFile.write() }
```

**精确调用几次（time）**

Mockito:

```kotlin
// Mockito
verify(mockedFile, times(3)).read()
```

Mockk:

```kotlin
// MockK
verify(exactly = 3) { mockedFile.read() }
```

**超时（timeout）**

Mockito:

```kotlin
// Mockito
verify(mockedFile, timeout(100)).readAsync()
```

Mockk:

```kotlin
// MockK
verify(timeout = 100) { mockedFile.readAsync() }
```

**所有方法没有交互(verifyNoInteractions)**

Mockito:

```kotlin
// Mockito
verifyNoInteractions(mockOne)
verifyNoInteractions(mockTwo, mockThree)
```

Mockk:

```kotlin
// MockK
verify { mockOne wasNot Called }
verify { listOf(mockTwo, mockThree) wasNot Called }
```

**所有方法没有更多交互(verifyNoMoreInteractions)**

Mockito:

```kotlin
// Mockito
verifyNoMoreInteractions(mockOne, mockTwo)
```

Mockk:

```kotlin
// MockK
confirmVerified(mockOne, mockTwo)
```

**协程**

Mockk:

```kotlin
val mockedFile = mockk<File>()

coVerify { mockedFile.readAsync() }
```

其它 co 方法请参考上面；

### 5. argThat

当我们需要根据某个对象来自定义是否能匹配时，则需使用 argThat

比如要传入`Person`这个对象，测试方法要根据性别来做出不同的返回:

```java
doReturn(mockValue).when(mockObject).mockMethod(argThat(new ArgumentMatcher<Person>() {
    @Override
    public boolean matches(Object o) {
        return "male".equals(((Person) o).getSex());
    }
}));
```



Mockito:

```kotlin
`when`(
  mockedCar.drive(argThat { engine -> engine.dieselEngine })
).thenReturn(true)
```

Mockk:

```kotlin
every {
  mockedCar.drive(match { engine -> engine.dieselEngine })
} returns true
```

### 6. ArgumentCaptor

当你需要对参数进行断言时，在Mockito中，可使用 ArgumentCaptor ； ArgumentCaptor会持有栈，并传递给mock方法， 稍后你可以调用它进行断言：

Mockito:

```kotlin
val personArgument = ArgumentCaptor.forClass(Person::class.java)

verify(mockPhone).call(personArgument.capture())

assertEquals("Sarah Jane", personArgument.value.name)
```

在Mocckk有一个类似的CapturingSlot：

Mockk:

```kotlin
val personSlot = slot<Person>()

every { mockPhone.call(capture(personSlot)) }

assertEquals("Sarah Jane", personSlot.captured.name)
```

作为CapturingSlot的替代方案，可以使用MutableList存储捕获的参数。只需传递一个MutableList的实例来捕获，而不是slot。这允许您记录所有捕获的值，因为CapturingSlot只记录最近的值。



> https://notwoods.github.io/mockk-guidebook/
>
> https://ithelp.ithome.com.tw/articles/10219718