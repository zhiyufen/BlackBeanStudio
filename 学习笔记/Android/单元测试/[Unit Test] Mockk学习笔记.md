## [Unit Test] Mockk 学习笔记

[TOC]

配置：

```groovy
testImplementation "io.mockk:mockk:{version}"
```

### Mocks对象

#### mock

对于我们定义的类，可使用mockk方法进行模拟，但mockk默认模式是严格模式，因此Mock对象需要对对象里的每个被调用的方法进行stub（插桩），不然会直接抛异常； 

```kotlin
val car = mockk<Car>()

every { car.drive(Direction.NORTH) } returns Outcome.OK

car.drive(Direction.NORTH) // returns OK

verify { car.drive(Direction.NORTH) }

confirmVerified(car)
```

#### Relaxed mock

mockk来mock对象，默认使用mockito中 [`STRICT_STUBS`](https://javadoc.io/static/org.mockito/mockito-core/3.3.3/org/mockito/quality/Strictness.html#STRICT_STUBS) 模式，也就是如果你一个方法没有进行stub的话，会进行抛出，这样可以更容易地捕获在您不期望调用的方法时调用的方法，或者在使用不同参数调用方法时调用的方法。

Mockito默认的宽大行为可以通过宽松的设置来复制。轻松的mock将为所有方法提供默认存根。 

```kotlin
val car = mockk<Car>(relaxed = true)

car.drive(Direction.NORTH) // returns null

verify { car.drive(Direction.NORTH) }

confirmVerified(car)
```

**注： relaxed mock 对于一些泛型返回类型时并不好用。**

使用relaxUnitFun 来仅仅返回值为空的方法进行宽松：

```kotlin
mockk<ClassBeingMocked>(relaxUnitFun = true)
```

#### Spy

上面使用mock出来的对象，都不会真实运行其方法里的代码，但实际上我们有时候是需要真实运行来测试其方法的逻辑是否按我们预期来跑， 这时就要用 spyk方法来创建模拟对象；

使用spyk模拟出来的对象，如果没有进行stub的话，默认所有方法调用时，都会真实运行其方法内每一行逻辑； 

```kotlin
val car = spyk(Car()) // or spyk<Car>() to call the default constructor

car.drive(Direction.NORTH) // returns whatever the real function of Car returns

verify { car.drive(Direction.NORTH) }

confirmVerified(car)
```

#### Object Mocks

```kotlin
object ObjBeingMocked {
  fun add(a: Int, b: Int) = a + b
}

mockkObject(ObjBeingMocked) // applies mocking to an Object

assertEquals(3, ObjBeingMocked.add(1, 2))

every { ObjBeingMocked.add(1, 2) } returns 55

assertEquals(55, ObjBeingMocked.add(1, 2))
```

mockkObject是Kotlin对象针对Object对象进行Mock的方法，但由于Object对象是全局唯一的，我们使用完需要对其进行释放，以免影响到其它类的测试，所以Object的对象测试流程如下：

```kotlin
@Before
fun beforeTests() {
    mockkObject(ObjBeingMocked)
    every { MockObj.add(1,2) } returns 55
}

@Test
fun willUseMockBehaviour() {
    assertEquals(55, ObjBeingMocked.add(1,2))
}

@After
fun afterTests() {
    unmockkAll()
    // or unmockkObject(ObjBeingMocked) 只释放某个object
}
```

#### Enumeration Mocks

针对枚举类，同时使用mockkObject方法：

```kotlin
enum class Enumeration(val goodInt: Int) {
    CONSTANT(35),
    OTHER_CONSTANT(45);
}

mockkObject(Enumeration.CONSTANT)
every { Enumeration.CONSTANT.goodInt } returns 42
assertEquals(42, Enumeration.CONSTANT.goodInt)
```

 #### Class Mock

针对第三方库的Class类，可使用 mockkClass()：

```kotlin
val car = mockkClass(Car::class)

every { car.drive(Direction.NORTH) } returns Outcome.OK

car.drive(Direction.NORTH) // returns OK

verify { car.drive(Direction.NORTH) }
```

#### Constuctor Mocks

有时候你需要创建一个新的对象(第三方库)， 但你无办法把你mock的对象传入你需要运行的对象来验证(或者创建对象的代码是包含在库时，你无法改变它)；这时你可以强势去stub其构造方法，让任何地方尝试创建对象时，调用其构造方法时，返回你mock的对象：

```kotlin
class MockCls {
  fun add(a: Int, b: Int) = a + b
}

mockkConstructor(MockCls::class)

every { anyConstructed<MockCls>().add(1, 2) } returns 4

assertEquals(4, MockCls().add(1, 2)) // note new object is created

verify { anyConstructed<MockCls>().add(1, 2) }
```

注：有多个构造方法，你需要分别进行mock其构造方法才行； 

WorkManager && WorkRequest;

#### 注解Mock

除了一个个手动mock外，也可使用注解直接mock对象：

```kotlin
class TrafficSystem {
  lateinit var car1: Car
  
  lateinit var car2: Car
  
  lateinit var car3: Car
}

class CarTest {
  @MockK
  lateinit var car1: Car

  @RelaxedMockK
  lateinit var car2: Car

  @MockK(relaxUnitFun = true)
  lateinit var car3: Car

  @SpyK
  var car4 = Car()
  
  @InjectMockKs
  var trafficSystem = TrafficSystem()
  
  @Before
  fun setUp() = MockKAnnotations.init(this, relaxUnitFun = true) // turn relaxUnitFun on for all mocks

  @Test
  fun calculateAddsValues1() {
      // ... use car1, car2, car3 and car4
  }
}
```

@InjectMockKs 创建模拟对象时，会将@MockK（或@RelaxedMockK,@SpyK ）注解创建的mock(car1, car2,car3)注入到用该实例中(trafficSystem); 

- @InjectMockKs 默认只针对 lateinit var 或者 var 这些没有默认值的进行注入；需要修改这个默认行为，需要使用的传入(overrideValues = true)的参数， 同时就像有些变量已经有变量，也会重新赋值；
- 如果需要@InjectMockKs 对 val 变量进行注入的话， 需要使用 (injectImmutable = true); 
- 或者你想把上面2个都适用(overrideValues = true， injectImmutable = true) ，也可直接使用@OverrideMockKs代替@InjectMockKs；

##### Junit5

如果你使用的是 Junit5的话， 就使用 MockKExtension 来代替 MockKAnnotations.init方法来初始化你的mock； 

```kotlin
@ExtendWith(MockKExtension::class)
class CarTest {
  @MockK
  lateinit var car1: Car

  @RelaxedMockK
  lateinit var car2: Car

  @MockK(relaxUnitFun = true)
  lateinit var car3: Car

  @SpyK
  var car4 = Car()

  @Test
  fun calculateAddsValues1() {
      // ... use car1, car2, car3 and car4
  }
}
```

另外在test方法的参数中，同样可以使用注解：

```
@Test
fun calculateAddsValues1(@MockK car1: Car, @RelaxedMockK car2: Car) {
  // ... use car1 and car2
}
```

### Stub（插桩）

对于mockk来说，基本上所有stub都使用 every 方法(不管区分有返回结果还是没返回结果的)

```kotlin
every { xxx.doSomething() } return "123"
```

#### 抛出异常

```kotlin
val mockedFile = mockk<File>()

every { mockedFile.read() } throws RuntimeException()
```

#### DoNothing

对于测试来说，有时我们只关心当前类的逻辑，并不在乎类与其它类交互时实际的运行； 

因此我们使用spyk出来的对象，针对无返回结果的一些方法我们只是需要知道是否被调用过就行，不需要其真正运行：

```kotlin
class MockedClass {
    fun sum(a: Int, b: Int): Unit {
        println(a + b)
    }
}

val obj = mockk<MockedClass>()

justRun { obj.sum(any(), 3) }

obj.sum(1, 1)
obj.sum(1, 2)
obj.sum(1, 3)

verify {
    obj.sum(1, 1)
    obj.sum(1, 2)
    obj.sum(1, 3)
}
```

其它写法：

```kotlin
every { obj.sum(any(), 3) } just Runs
every { obj.sum(any(), 3) } returns Unit
every { obj.sum(any(), 3) } answers { Unit }
```

#### ArgumentMatcher

插桩和验证时除了指定特定的参数，为了更灵活，还可以使用通配符和正则表达式：

```kotlin
val car = mockk<Car>()

every { 
  car.recordTelemetry(
    speed = more(50),
    direction = Direction.NORTH, // here eq() is used
    lat = any(),
    long = any()
  )
} returns Outcome.RECORDED

car.recordTelemetry(60, Direction.NORTH, 51.1377382, 17.0257142)

verify { car.recordTelemetry(60, Direction.NORTH, 51.1377382, 17.0257142) }

confirmVerified(car)
```

更多match通配符，请参考DSL 表格里的 matchers；

#### 连续调用

```kotlin
val car = mockk<Car>()

every { car.door(DoorType.FRONT_LEFT).windowState() } returns WindowState.UP

car.door(DoorType.FRONT_LEFT) // returns chained mock for Door
car.door(DoorType.FRONT_LEFT).windowState() // returns WindowState.UP

verify { car.door(DoorType.FRONT_LEFT).windowState() }

confirmVerified(car)
```

注：如果你的方法返回的是泛型的，那么mockk是无法知道真实类是哪个，需要你使用hint方法显式声明它(大多情况下，框架会捕获强制转换异常并执行自动暗示)，再调用下一个调用； 

```kotlin
every { obj.op2(1, 2).hint(Int::class).op1(3, 4) } returns 5
```

#### 分层Mocking

在mockk 1.9.1及后面版本，可分层moocking

```kotlin
interface AddressBook {
    val contacts: List<Contact>
}

interface Contact {
    val name: String
    val telephone: String
    val address: Address
}

interface Address {
    val city: String
    val zip: String
}

val addressBook = mockk<AddressBook> {
    every { contacts } returns listOf(
        mockk {
            every { name } returns "John"
            every { telephone } returns "123-456-789"
            every { address.city } returns "New-York"
            every { address.zip } returns "123-45"
        },
        mockk {
            every { name } returns "Alex"
            every { telephone } returns "789-456-123"
            every { address } returns mockk {
                every { city } returns "Wroclaw"
                every { zip } returns "543-21"
            }
        }
    )
}
```

#### Capturing

当你需要对参数进行断言时，可使用CapturingSlot` or `MutableList；

```kotlin
val car = mockk<Car>()

val slot = slot<Double>()
val list = mutableListOf<Double>()

every {
  car.recordTelemetry(
    speed = capture(slot), // makes mock match calls with any value for `speed` and record it in a slot
    direction = Direction.NORTH // makes mock and capturing only match calls with specific `direction`. Use `any()` to match calls with any `direction`
  )
} answers {
  println(slot.captured)
  Outcome.RECORDED
}


every {
  car.recordTelemetry(
    speed = capture(list),
    direction = Direction.SOUTH
  )
} answers {
  println(list)
  Outcome.RECORDED
}

car.recordTelemetry(speed = 15, direction = Direction.NORTH) // prints 15
car.recordTelemetry(speed = 16, direction = Direction.SOUTH) // prints 16

verify(exactly = 2) { car.recordTelemetry(speed = or(15, 16), direction = any()) }

confirmVerified(car)
```

#### Consecutive calls

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

#### 私有方法Mocking

如果你需要stub私有方法， 需要用到动态调用.

```kotlin
class Car {
    fun drive() = accelerate()

    private fun accelerate() = "going faster"
}

val mock = spyk<Car>(recordPrivateCalls = true)

every { mock["accelerate"]() } returns "going not so fast"

assertEquals("going not so fast", mock.drive())

verifySequence {
    mock.drive()
    mock["accelerate"]()
}
```

注：如果你要验证私有方法是否调用，你需要使用spyk创建对象并传入recordPrivateCalls = true； 

此外，更详细的语法允许您获取和设置属性，并结合相同的动态调用：

```kotlin
val mock = spyk(Team(), recordPrivateCalls = true)

every { mock getProperty "speed" } returns 33
every { mock setProperty "acceleration" value less(5) } just runs
justRun { mock invokeNoArgs "privateMethod" }
every { mock invoke "openDoor" withArguments listOf("left", "rear") } returns "OK"

verify { mock getProperty "speed" }
verify { mock setProperty "acceleration" value less(5) }
verify { mock invoke "openDoor" withArguments listOf("left", "rear") }
```



### Verification

#### Verify

验证某方法是否被调用过：

```
val mockedFile = mockk<File>()

mockedFile.read()
verify { mockedFile.read() }
```

#### 验证调用次数

未调用过某个方法(never):

```kotlin
// MockK
verify(inverse = true) { mockedFile.write() }
```

至少调用几次(atLeast/atLeastOnce):

```
// MockK
verify(atLeast = 3) { mockedFile.read() }
verify(atLeast = 1) { mockedFile.write() }
```

最多调用几次（atMost/atMostOnce）:

```
// MockK
verify(atMost = 3) { mockedFile.read() }
verify(atMost = 1) { mockedFile.write() }
```

精确调用几次（time）:

```kotlin
// MockK
verify(exactly = 3) { mockedFile.read() }
```

#### 验证调用顺序

- verifyAll 验证{}里是否都被调用了(但不要求调用顺序)；
- verifySequence 验证{}里是否都按顺序调用了； 
- verifyOrder 验证{}里是否都按顺序调用了； 
- wasNot Called 验证方法或某组方法都没有调用过；

```kotlin
class MockedClass {
    fun sum(a: Int, b: Int) = a + b
}

val obj = mockk<MockedClass>()
val slot = slot<Int>()

every {
    obj.sum(any(), capture(slot))
} answers {
    1 + firstArg<Int>() + slot.captured
}

obj.sum(1, 2) // returns 4
obj.sum(1, 3) // returns 5
obj.sum(2, 2) // returns 5

verifyAll {
    obj.sum(1, 3)
    obj.sum(1, 2)
    obj.sum(2, 2)
}

verifySequence {
    obj.sum(1, 2)
    obj.sum(1, 3)
    obj.sum(2, 2)
}

verifyOrder {
    obj.sum(1, 2)
    obj.sum(2, 2)
}

val obj2 = mockk<MockedClass>()
val obj3 = mockk<MockedClass>()
verify {
    listOf(obj2, obj3) wasNot Called
}

confirmVerified(obj)
```

#### 验证超时

有些需要验证并发操作(异步)，就是我们只需要确保在某段时间内调用某个方法就行， 超时的话，会进行报错；

```kotlin
mockk<MockCls> {
    every { sum(1, 2) } returns 4

    Thread {
        Thread.sleep(2000)
        sum(1, 2)
    }.start()

    verify(timeout = 3000) { sum(1, 2) }
}
```

#### 验证确认

有些需要确保所有调用都使用 verify... 进行验证，则使用confirmVerified； 如果有些调用没验证的话，会直接抛异常；

```kotlin
confirmVerified(mock1, mock2)
```

#### 例外记录

如果有些调用的验证不重要，你可能通过excludeRecords对其进行例外其验证：

```kotlin
excludeRecords { mock.operation(any(), 5) }
```

这个例外只有在verifyAll`, `verifySequence` or `confirmVerified 里验证时，会有用，也就是不会验证例外方法的。

```kotlin
val car = mockk<Car>()

every { car.drive(Direction.NORTH) } returns Outcome.OK
every { car.drive(Direction.SOUTH) } returns Outcome.OK

excludeRecords { car.drive(Direction.SOUTH) }

car.drive(Direction.NORTH) // returns OK
car.drive(Direction.SOUTH) // returns OK

verify {
    car.drive(Direction.NORTH)
}

confirmVerified(car) // car.drive(Direction.SOUTH) was excluded, so confirmation is fine with only car.drive(Direction.NORTH)
```

### 协程相关测试

为了支持协程相关的测试，需要引入支持库：

```groovy
testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:x.x"
```

由于Mockk是使用函数文本来stub的，因此进行stub挂起方法时，需要进行一些小改动；

Mockk提供了前缀为 **co** 的函数来mock挂起函数: coEvery`, `coVerify`, `coMatch`, `coAssert`, `coRun`, `coAnswers和coInvoke； 

```kotlin
val car = mockk<Car>()

coEvery { car.drive(Direction.NORTH) } returns Outcome.OK

car.drive(Direction.NORTH) // returns OK

coVerify { car.drive(Direction.NORTH) }
```

### 其它

#### 扩展函数

扩展函数分成三个类型：

- class-wide
- object-wide
- module-wide

针对 class 和 object的， 和正常mock一样：

```kotlin
data class Obj(val value: Int)

class Ext {
    fun Obj.extensionFunc() = value + 5
}

with(mockk<Ext>()) {
    every {
        Obj(5).extensionFunc()
    } returns 11

    assertEquals(11, Obj(5).extensionFunc())

    verify {
        Obj(5).extensionFunc()
    }
}
```

module-wide(略), 如有需要，请查看官方文档； 

#### Mock 静态方法

有时候Kotlin需要和java的一些静态方法进行工作， 如你所见，java静态方法是全局只有一份的，因此无法像类一样正常mock； 这时需要用到 mockkStatic:

```kotlin
package com.name.app;

class Writer {
  public static File getFile(String path) {
    return File(path);
  }
}

//mockkStatic("com.name.app.Writer")
mockkStatic(Writer::class)

```

与Object一样，这个mock是 **spies** 的，也就是如果方法没stub，方法会运行实际的逻辑；

对于这类，用完记得unmockkObject；

#### 可变参数

```kotlin
    interface ClsWithManyMany {
        fun manyMany(vararg x: Any): Int
    }

    val obj = mockk<ClsWithManyMany>()

    every { obj.manyMany(5, 6, *varargAll { it == 7 }) } returns 3

    println(obj.manyMany(5, 6, 7)) // 3
    println(obj.manyMany(5, 6, 7, 7)) // 3
    println(obj.manyMany(5, 6, 7, 7, 7)) // 3

    every { obj.manyMany(5, 6, *anyVararg(), 7) } returns 4

    println(obj.manyMany(5, 6, 1, 7)) // 4
    println(obj.manyMany(5, 6, 2, 3, 7)) // 4
    println(obj.manyMany(5, 6, 4, 5, 6, 7)) // 4

    every { obj.manyMany(5, 6, *varargAny { nArgs > 5 }, 7) } returns 5

    println(obj.manyMany(5, 6, 4, 5, 6, 7)) // 5
    println(obj.manyMany(5, 6, 4, 5, 6, 7, 7)) // 5

    every {
        obj.manyMany(5, 6, *varargAny {
            if (position < 3) it == 3 else it == 4
        }, 7)
    } returns 6
    
    println(obj.manyMany(5, 6, 3, 4, 7)) // 6
    println(obj.manyMany(5, 6, 3, 4, 4, 7)) // 6
```

#### 扩展matcher

一个最简单的方法就是使用MockKMatcherScope或MockKVerificationScope：

```kotlin
  fun MockKMatcherScope.seqEq(seq: Sequence<String>) = match<Sequence<String>> {
        it.toList() == seq.toList()
    }
```

另外也可直接实现matcher接口(略)，

#### 全局配置

如果有需要，可针对全局设置一些设置属性：

1. 在 src/main/resources 创建io/mockk/settings.properties文件；

2. 然后在该文件根据需要添加下面的属性：

   ```groovy
   relaxed=true|false
   relaxUnitFun=true|false
   recordPrivateCalls=true|false
   stackTracesOnVerify=true|false
   stackTracesAlignment=left|center
   ```

### DSL 表格

很多这里没有列出，请参考：[Mockk官网](https://mockk.io/)

#### Matchers

| Matcher                                                 | Description                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| `any()`                                                 | matches any argument                                         |
| `allAny()`                                              | special matcher that uses `any()` instead of `eq()` for matchers that are provided as simple arguments |
| `isNull()`                                              | checks if the value is null                                  |
| `isNull(inverse=true)`                                  | checks if the value is not null                              |
| `ofType(type)`                                          | checks if the value belongs to the type                      |
| `match { it.startsWith("string") }`                     | matches via the passed predicate                             |
| `coMatch { it.startsWith("string") }`                   | matches via the passed coroutine predicate                   |
| `matchNullable { it?.startsWith("string") }`            | matches nullable value via the passed predicate              |
| `coMatchNullable { it?.startsWith("string") }`          | matches nullable value via the passed coroutine predicate    |
| `eq(value)`                                             | matches if the value is equal to the provided value via the `deepEquals` function |
| `eq(value, inverse=true)`                               | matches if the value is not equal to the provided value via the `deepEquals` function |
| `neq(value)`                                            | matches if the value is not equal to the provided value via the `deepEquals` function |
| `refEq(value)`                                          | matches if the value is equal to the provided value via reference comparison |
| `refEq(value, inverse=true)`                            | matches if the value is not equal to the provided value via reference comparison |
| `nrefEq(value)`                                         | matches if the value is not equal to the provided value via reference comparison |
| `cmpEq(value)`                                          | matches if the value is equal to the provided value via the `compareTo` function |
| `less(value)`                                           | matches if the value is less than the provided value via the `compareTo` function |
| `more(value)`                                           | matches if the value is more than the provided value via the `compareTo` function |
| `less(value, andEquals=true)`                           | matches if the value is less than or equal to the provided value via the `compareTo` function |
| `more(value, andEquals=true)`                           | matches if the value is more than or equal to the provided value via the `compareTo` function |
| `range(from, to, fromInclusive=true, toInclusive=true)` | matches if the value is in range via the `compareTo` function |
| `and(left, right)`                                      | combines two matchers via a logical and                      |
| `or(left, right)`                                       | combines two matchers via a logical or                       |
| `not(matcher)`                                          | negates the matcher                                          |
| `capture(slot)`                                         | captures a value to a `CapturingSlot`                        |
| `capture(mutableList)`                                  | captures a value to a list                                   |
| `captureNullable(mutableList)`                          | captures a value to a list together with null values         |
| `captureLambda()`                                       | captures a lambda                                            |
| `captureCoroutine()`                                    | captures a coroutine                                         |
| `invoke(...)`                                           | calls a matched argument                                     |
| `coInvoke(...)`                                         | calls a matched argument for a coroutine                     |
| `hint(cls)`                                             | hints the next return type in case it’s gotten erased        |
| `anyVararg()`                                           | matches any elements in a vararg                             |
| `varargAny(matcher)`                                    | matches if any element matches the matcher                   |
| `varargAll(matcher)`                                    | matches if all elements match the matcher                    |
| `any...Vararg()`                                        | matches any elements in vararg (specific to primitive type)  |
| `varargAny...(matcher)`                                 | matches if any element matches the matcher (specific to the primitive type) |
| `varargAll...(matcher)`                                 | matches if all elements match the matcher (specific to the primitive type) |

一些特殊matchers， 只用于验证模式的。

| Matcher                      | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `withArg { code }`           | matches any value and allows to execute some code            |
| `withNullableArg { code }`   | matches any nullable value and allows to execute some code   |
| `coWithArg { code }`         | matches any value and allows to execute some coroutine code  |
| `coWithNullableArg { code }` | matches any nullable value and allows to execute some coroutine cod |

https://notwoods.github.io/mockk-guidebook/docs/mocking/answers/