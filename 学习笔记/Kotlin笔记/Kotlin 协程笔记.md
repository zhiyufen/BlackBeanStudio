## Kotlin 协程笔记

[TOC]

### 一、协程基础（Coroutine Basics）

#### 第一个协程

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新协程，并继续执行之后的代码
        delay(1000L) // 非阻塞式地延迟一秒
        println("World!") // 延迟结束后打印
    }
    println("Hello,") //主线程继续执行，不受协程 delay 所影响
    Thread.sleep(2000L) // 主线程阻塞式睡眠2秒，以此来保证JVM存活
}
```

输出结果：

```
Hello,
World!
```

 本质上，协程可以称为**轻量级线程**。协程在 CoroutineScope （协程作用域）的上下文中通过 launch、async 等协程构造器（coroutine builder）来启动。在上面的例子中，在 GlobalScope ，即**全局作用域**内启动了一个新的协程，这意味着该协程的生命周期只受整个应用程序的生命周期的限制，即只要整个应用程序还在运行中，只要协程的任务还未结束，该协程就可以一直运行 ; 

协程并不会绑定到某个线程上，它可能在线程A进行挂起，但在线程B上恢复运行；

- **launch**:  是一个协程生成器，它会启动一个新的协程并进行独立运行；
- **delay**:  是一个特殊的挂起函数， 它会挂起当前协程一定的时间。 但该挂起并不会阻塞到其它协程和底层的线程的运行；

> 开发者需要明白，协程是运行于线程上的，一个线程可以运行多个（可以是几千上万个）协程。线程的调度行为是由 OS 来操纵的，而协程的调度行为是可以由开发者来指定并由编译器来实现的。当协程 A 调用 delay(1000L) 函数来指定延迟1秒后再运行时，协程 A 所在的线程只是会转而去执行协程 B，等到1秒后再把协程 A 加入到可调度队列里。所以说，线程并不会因为协程的延时而阻塞，这样可以极大地提高线程的并发灵活度 

#### 桥接阻塞与非阻塞

```kotlin
import kotlinx.coroutines.*

fun main() { 
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    runBlocking {     // but this expression blocks the main thread
        delay(2000L)  // ... while we delay for 2 seconds to keep JVM alive
    } 
}
```

 输出结果与第一个程序一致，这段代码只使用了非阻塞延迟。主线程调用了 runBlocking 函数，直到 runBlocking 内部的所有协程执行完成后，之后的代码才会继续执行 ;

或者使用如下的写法：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // start main coroutine
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main coroutine continues here immediately
    delay(2000L)      // delaying for 2 seconds to keep JVM alive
}
```

 这里 `runBlocking { ... }` 作为用于启动顶层主协程的适配器。 

**runBlocking**: 是一个协程生成器，并作为桥接启动`runBlocking`的方法和`runBlocking { ... }`里协程的，简单说，`runBlocking`语句会阻塞当前方法的线程(并不会阻塞由该线程启动的其它协程)，直到`runBlocking`里协程运行完后，才会往下执行`runBlocking{..}`外的语句；所以一般`runBlocking`是作为 启动顶层主协程的生成器；一般来说，生产环境的代码不会使用`runBlocking`， 因为阻塞当前线程是不划算的；

 这也是为挂起函数编写单元测试的一种方法： 

```kotlin
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // here we can use suspending functions using any assertion style that we like
    }
}
```

>runBlocking 代码块默认运行于其声明所在的线程，而 launch 代码块默认运行于线程池中，可以通过打印当前线程名来进行区分 

#### 等待作业

`runBlocking`只保证当前协程域里的协程运行完成，如果`runBlocking`启动了其它协程域的协程的话，是无法保证的；
比如： 如果改这里delay时间为500L，只输出“Hello,”，而launch的生命周期早早结束。也就是延迟一段时间来等待另一个协程运行并不是一个好的选择，可以显式（非阻塞的方式）地等待协程执行完成 ：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = GlobalScope.launch { // launch a new coroutine and keep a reference to its Job
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // wait until child coroutine completes   
}
```

这样就能保证协程程序运行完成后再结果main；

#### 结构化并发（Structured concurrency）

协程遵循[结构化并发](https://www.jianshu.com/p/3dc8ba43c28b)的原则，也就是新的协程只在协程域中进行启动，每个协程都有自己作用域，只有所有协程都完成后协程域的生命周期才会结束；或者主动结束协程域时，所有协程都会结束；

当使用 GlobalScope.launch 时，会创建一个顶层协程，是针对整个应用的生命周期的，尽管它是轻量级的，但仍然会造成一些内存资源消耗；当我们忘记持有这个新顶层协程的引用，没显式关闭它，它一直在跑或delay中，这样就会造成泄漏，但如果每个都要去持有它，显式关闭它又是一个很麻烦的事；所以一般情况不会使用GlobalScope.launch；

这时一个更好的方案就是使用结构化并发来代替使用 GlobalScope.launch： 在我们正在执行的操作的特定范围内启动协同程序。 

**规则**：<u>外部协程只有在其作用域中启动的所有协程完成后才会完成。</u>

也就是我们在协程生成器的作用域中启动的协程无需显式持有其引用并调用join来等待； 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine in the scope of runBlocking
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

#### 域的创建（Scope builder）

除了不同协程构建器提供的协程域，你也可以使用`coroutineScope`创建声明自己的域。它创建的一个协程作用域会直到所有开启的子协程完成后才结束。

##### [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) VS [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)

**共同点**： 2个都会等其协程体以及所有子协程完成后才会完成； 

**不同点**：runBlocking会阻塞当前线程来等待， 是一个常规函数；而coroutineScope仅仅是挂起，会释放底层线程用于其他用途(比如之前启动的协程)， 是一个挂起函数，也有一个含义就是coroutineScope只能在协程内或挂起函数里使用； 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { 
        delay(200L)
        println("Task from runBlocking")
    }
    
    coroutineScope { // Creates a coroutine scope
        launch {
            delay(500L) 
            println("Task from nested launch")
        }
    
        delay(100L)
        println("Task from coroutine scope") // This line will be printed before the nested launch
    }
    coroutineScope { // Creates a coroutine scope
        println("Task from coroutine scope2") 
    }
    
    println("Coroutine scope is over") // This line is not printed until the nested launch completes
}
```

输出结果：

```
Task from coroutine scope
Task from runBlocking
Task from nested launch
Task from coroutine scope2
Coroutine scope is over
```

从结果来看，当运行到coroutineScope创建时， 当前线程会挂起，并不会往下执行，直到当前协程完成后才恢复往下执行； 

> 如不太明白"Coroutine scope is over"为什么是最后打印，请参考[深层次揭示runBlocking与coroutineScope之间的异同点](https://blog.csdn.net/webor2006/article/details/119747506)

#### 提取重构函数(Extract function refactoring)/挂起函数

把协程内运行的语句提取成一个函数，该函数需使用“suspend”声明函数，称为挂起函数； 挂起函数可以在协程内使用，并且内部可再调用其它挂起函数； 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch { doWorld() }
    println("Hello,")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

#### 域的创建和结构并发性

域是可以在挂起函数中创建来执行更多并发操作；

```kotlin
// Sequentially executes doWorld followed by "Done"
fun main() = runBlocking {
    doWorld()
    println("Done")
}

// Concurrently executes both sections
suspend fun doWorld() = coroutineScope { // this: CoroutineScope
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}
```

输出结果：

```
Hello
World 1
World 2
Done
```

结果为什么这样，不理解的话，请参考上面的： 4. 结构化并发（Structured concurrency）

####  协程的持有对象

协程的创建会返回一个`job`对象，我们通过持有该对象进行一些操作，比如显式等待该协程结果等，

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch { // launch a new coroutine and keep a reference to its Job
        delay(1000L)
        println("World!")
    }
    println("Hello")
    job.join() // wait until child coroutine completes
    println("Done")     
}
```

输出结果：

```
Hello
World!
Done
```

#### 协程是轻量级(Coroutines ARE light-weight)

运行下面的代码：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    repeat(100_000) { // launch a lot of coroutines
        launch {
            delay(5000L)
            print(".")
        }
    }
}
```

起动10万个协程，能正常运行，但改10万个线程的话，可能会造成内存不足； 

#### 全局协程类似于守护线程

GlobalScope 中启动了一个会长时间运行的协程:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    GlobalScope.launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // just quit after delay
//sampleEnd    
}
```

输出结果：

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```

launch 函数依附的协程作用域是 GlobalScope，而非 runBlocking 所隐含的作用域。在 GlobalScope 中启动的协程无法使进程保持活动状态，它们就像守护线程（当主线程消亡时，守护线程也将消亡）

### 二、取消和超时(Cancellation and Timeouts)

#### 取消执行中的协程(Cancelling coroutine execution)

在一个长时间运行的应用程序中，你可能需要对后台协程进行细粒度的控制。比如，用户已经退出某个页面，这个页面启动了一个协程来获取某个结果 ，但这个结果已经不需要的，这时我们就需要取消这个协程； 

launch函数会返回一个 job, 通过job可以取消该协程：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancel() // cancels the job
    job.join() // waits for job's completion 
    println("main: Now I can quit.")
//sampleEnd    
}
```

输出的结果如下：

```
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```

可以看到，一旦调用job.cancel()方法时，launch里不再运行了； 

另外可使用 cancelAndJoin() 代替 cancel 和 join方法结合使用； 

#### 取消是协作的(Cancellation is cooperative)

协程代码必须配合才能取消。所有在kotlinx.coroutines的挂起函数都是可取消的，它会检查去取消协程，并取消时会抛出 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)， 但是如果协程正在计算并没有检查是否取消，则无法取消该协程。比如下面例子：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) { // computation loop, just wastes CPU
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    job.cancel()
    println("main: I'm tired of waiting!")
    job.join() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

 可以看到在调用cancel()方法后，“..I'm sleeping..” 仍然在打印，直到5次打印才停止； 

#### 使计算中的协程可取消(Making computation code cancellable)

- 使用 yield 函数，该函数会让运行当前协程的线程或线程池尽可能调度其它协程来运行；(测试过，不可靠) 
- 根据协程的标志位 `isActive` 来显式检查取消状态；

看看后面一种方法的例子：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (isActive) { // cancellable computation loop
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

看输出结果， 循环能取消了。 

isActive是一个扩展属性，可以通过CoroutineScope对象在协程中使用。

#### 在`Finally`中释放资源 (Closing resources with `finally`)

可取消的挂起函数在取消时会抛出 CancellationException， 可使用`try {...} finally {...}` 表达式和 kotlin 的 `use` 函数都可用于在取消协程时执行回收操作。

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            println("job: I'm running finally")
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

#### 运行不可取消的代码块(Run non-cancellable block)

一般来说，一个性能良好的关闭操作（关闭文件、取消作业、关闭任何类型的通信通道等）通常都是非阻塞的，且不涉及任何挂起函数。但是，在极少数情况下，当需要在取消的协程中调用挂起函数时，可以使用 withContext 函数和 NonCancellable 上下文将相应的代码包装在 `withContext(NonCancellable) {...}` 代码块中； 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            withContext(NonCancellable) {
                println("job: I'm running finally")
                delay(1000L)
                println("job: And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

#### 超时(Timeout)

取消协程执行的最明显的实际原因是，它的执行时间已经超过了某个超时。虽然您可以手动跟踪对相应作业的引用，并在延迟之后启动一个单独的协程来取消跟踪的协程，但是有一个现成的withTimeout函数可以执行此操作。

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    withTimeout(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
//sampleEnd
}
```

输出：

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1300 ms
 at kotlinx.coroutines.TimeoutKt.TimeoutCancellationException (Timeout.kt:149) 
 at kotlinx.coroutines.TimeoutCoroutine.run (Timeout.kt:119) 
 at kotlinx.coroutines.EventLoopImplBase$DelayedRunnableTask.run (EventLoop.common.kt:493) 
```

withTimeout主动抛出TimeoutCancellationException是CancellationException的子类； 那前面的例子为什么没抛出CancellationException，那是因为对于一个已取消的协程来说，CancellationException 被认为是触发协程结束的正常原因。

TimeoutCancellationException异常抛出时，需要做一些特殊操作时，可使用 `try {...} catch (e: TimeoutCancellationException) {...}` ， 或者直接使用 [withTimeoutOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout-or-null.html)函数，该函数在异常抛出时，会直接返回null； 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val result = withTimeoutOrNull(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
        "Done" // will get cancelled before it produces this result
    }
    println("Result is $result")
//sampleEnd
}
```

输出结果：

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Result is null
```

#### 异步超时和资源(Asynchronous timeout and resources)

withTimeout中的超时事件相对于在其块中运行的代码是异步的，并且可能随时发生，甚至就在从超时块内部返回之前。如果在块内打开或获取某些资源，而这些资源需要在块外关闭或释放，请记住这一点。

```kotlin
import kotlinx.coroutines.*

//sampleStart
var acquired = 0

class Resource {
    init { acquired++ } // Acquire the resource
    fun close() { acquired-- } // Release the resource
}

fun main() {
    runBlocking {
        repeat(100_000) { // Launch 100K coroutines
            launch { 
                val resource = withTimeout(60) { // Timeout of 60 ms
                    delay(50) // Delay for 50 ms
                    Resource() // Acquire a resource and return it from withTimeout block
                }
                resource.close() // Release the resource
            }
        }
    }
    // Outside of runBlocking all coroutines have completed
    println(acquired) // Print the number of resources still acquired
}
//sampleEnd
```

上面输出有可能不是0，也就是resource.close()有可能没运行； 
注：acquired增加和减少是线程安全的； 

要解决此问题，可以在变量中存储对资源的引用，而不是从withTimeout块返回它。

```kotlin
import kotlinx.coroutines.*

var acquired = 0

class Resource {
    init { acquired++ } // Acquire the resource
    fun close() { acquired-- } // Release the resource
}

fun main() {
//sampleStart
    runBlocking {
        repeat(100_000) { // Launch 100K coroutines
            launch { 
                var resource: Resource? = null // Not acquired yet
                try {
                    withTimeout(60) { // Timeout of 60 ms
                        delay(50) // Delay for 50 ms
                        resource = Resource() // Store a resource to the variable if acquired      
                    }
                    // We can do something else with the resource here
                } finally {  
                    resource?.close() // Release the resource if it was acquired
                }
            }
        }
    }
    // Outside of runBlocking all coroutines have completed
    println(acquired) // Print the number of resources still acquired
//sampleEnd
}
```

### 三、组合挂起函数

本节介绍组成挂起函数的各种方法。

#### 默认顺序(Sequential by default)

假设我们在别处定义了两个挂起函数，它们执行一些有用的操作，比如远程服务调用或计算。我们只是假装它们是有用的，但实际上为了这个例子，每一个都会延迟一秒钟：

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

我们需要两个挂起函数进行顺序调用的话，则直接在协程代码块中，顺序调用2个方法即可；

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
//sampleStart
    //measureTimeMillis函数是返回当前代码块的运行时间(单位毫秒)
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")
//sampleEnd    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

输出结果：

```
The answer is 42
Completed in 2017 ms
```

从结果和运行时间来看，2个挂起方法是按顺序先后运行的。

#### 异步并发(Concurrent using async)

如果doSomethingUsefulOne和doSomethingUsefulTwo方法并没有依赖关系，我们想更快的返回结果(同时运行这2个挂起函数)，这时就需要使用到并发； 

`async`类似于 `launch`，会启动一个独立的协程来和其它协程并行运行；区别是`launch`会返回`job`对象并且不会返回结果；而`async`会返回结果并且返回`Deferred`对象(是轻量非阻塞功能，并且确保能返回结果)；你可以使用` .await()`来获取它的最终结果， `Deferred`继承于 `Job`， 因此你也可以通过`Deferred`来取消该协程；

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

输出结果：

```
The answer is 42
Completed in 1017 ms
```

整个运行时间减少了一半；

#### 懒式启动Async(Lazily stared async)

`async`协程可通过 `CoroutineStart.LAZY`来懒式启动： 先使用`CoroutineStart.LAZY`创建`async`协程，等需要时再启动该协程来获取结果；

```kotlin
val time = measureTimeMillis {
    val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
    val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
    // some computation
    one.start() // start the first one
    two.start() // start the second one
    println("The answer is ${one.await() + two.await()}")
}
println("Completed in $time ms")
```

输出结果：

```
The answer is 42
Completed in 1017 ms
```

这样就可以把定义async工作的地方，和启动运行的地方(.await())进行分开， 这样对于一些使用模式设计的代码会更好进行解耦；

注： 如果使用`CoroutineStart.LAZY`创建`async`协程，没启动`start()`就直接使用`await()`来获取结果的话，会造成这些协程会顺序运行的；

```kotlin
val time = measureTimeMillis {
    val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
    val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
    // some computation
    //one.start() // start the first one
    //two.start() // start the second one
    println("The answer is ${one.await() + two.await()}")
}
println("Completed in $time ms")
```

输出结果：

```
The answer is 42
Completed in 2040 ms
```

#### Async函数

在协程域上使用`async`协程生成器抽取函数，让挂起函数可异步并从结构化并发中退出；

一般这类函数以"...Async"的风格来命名， 来告知使用者该函数只启动异步运行和需要使用Deferred来获取结果；

```kotlin
// The result type of somethingUsefulOneAsync is Deferred<Int>
@OptIn(DelicateCoroutinesApi::class)
fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

// The result type of somethingUsefulTwoAsync is Deferred<Int>
@OptIn(DelicateCoroutinesApi::class)
fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}
```

需要注意的是这些`xxxAsync`函数并没有挂起函数， 它能使用于任何地方；然而当调用这些函数时，是异步(并发)的；

```kotlin
// note that we don't have `runBlocking` to the right of `main` in this example
fun main() {
    val time = measureTimeMillis {
        // we can initiate async actions outside of a coroutine
        val one = somethingUsefulOneAsync()
        val two = somethingUsefulTwoAsync()
        // but waiting for a result must involve either suspending or blocking.
        // here we use `runBlocking { ... }` to block the main thread while waiting for the result
        runBlocking {
            println("The answer is ${one.await() + two.await()}")
        }
    }
    println("Completed in $time ms")
}
```

输出结果：

```
The answer is 42
Completed in 1169 ms
```

上面`async`函数仅仅作为演示而已，一般为其它语言流行的风格， 但这风格对于Kotlin协程来说，是强烈不支持使用的，原因如下：
	以上面为例，如果`val one = somethingUsefulOneAsync()`和`one.await()`两行之间的代码有错误并抛出异常时，程序会中止；通常来说，全局错误处理会Catch到这个异常并报告给开发者，但程序可以继续执行其它操作；然而，尽管启动它的操作被中止了，但`somethingUsefulOneAsync()`仍然在后台运行；这种情况在结构化并发里不会出现；

#### 结构化并发Async(Structured concurrency with async)

如下代码： 把doSomethingUsefulOne 和 doSomethingUsefulTwo 方法抽取到一个挂起函数里，并且返回最终的共同结果；因为`async`的协程生成器是使用 `CoroutineScope`(协程域)创建，也就是这2个`async`协程都在同一个协程域中运行：

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```

按这样的方式， concurrentSum挂起函数就算出现错误， 那么抛出异常时，所有的协程域里协程都会被取消；

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        println("The answer is ${concurrentSum()}")
    }
    println("Completed in $time ms")    
}

suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

输出结果：

```
The answer is 42
Completed in 1017 ms
```

> 某个协程出了异常，同个协程域会通过协程层次结构来取消该协程域里的其它协程；
>
> ```kotlin
> import kotlinx.coroutines.*
> 
> fun main() = runBlocking<Unit> {
>     try {
>         failedConcurrentSum()
>     } catch(e: ArithmeticException) {
>         println("Computation failed with ArithmeticException")
>     }
> }
> 
> suspend fun failedConcurrentSum(): Int = coroutineScope {
>     val one = async<Int> { 
>         try {
>             delay(Long.MAX_VALUE) // Emulates very long computation
>             42
>         } finally {
>             println("First child was cancelled")
>         }
>     }
>     val two = async<Int> { 
>         println("Second child throws an exception")
>         throw ArithmeticException()
>     }
>     one.await() + two.await()
> }
> ```
>
> 输出结果：
>
> ```
> Second child throws an exception
> First child was cancelled
> Computation failed with ArithmeticException
> ```
>
> 可以看到`one`的`async`协程也取消了；

### 四、协程上下文和调度器(Coroutine context and dispatchers)

协程一直运行在由Kotlin库定义的`CoroutineContext`的上下文中；协程Context包含一系列的成员，主要成员是 前文介绍的`Job`和本节的调度者`dispatcher`; 

#### 调度器和线程(Dispatcher and threads)

协程Context包括其调度器，而调度器决定哪个线程或哪些线程们来运行协程。调度器可限定协程在某个线程内运行，也可让其在线程池中或不受限制地的运行；

所有协程创建器(`launch, async`等)都接受一个 [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/)参数， 而这个参数显式指定其调度器和Context; 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch { // context of the parent, main runBlocking coroutine
        println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
        println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher 
        println("Default               : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
        println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
    }    
}
```

输出结果：

```
Unconfined            : I'm working in thread main @coroutine#3
Default               : I'm working in thread DefaultDispatcher-worker-1 @coroutine#4
main runBlocking      : I'm working in thread main @coroutine#2
newSingleThreadContext: I'm working in thread MyOwnThread @coroutine#5
```

- 当使用launch { ... }`时， 其Context和调度器都继承于当前协程域的，runBlocking是Main线程的，所以会运行在Main线程上；`
- 而使用`launch(Dispatchers.Unconfined)`时，虽然log打印在Main线程，但实际是不限制其协程域的，后面会说明；
- 一般没显式指定其调度器(Unconfined)或使用Dispatchers.Default时， 都会运行在默认协程中，该默认协程是在后台共享线程池的协程域中；
- `newSingleThreadContext`会创建一个新线程来专门跑运行该协程；一般来说，创建一个专门使用的线程是浪费资源的，因此实际使用中，必须显式进行释放它(在使用完成后)；或者放到全局顶层变量，来进行复用；

#### Unconfined vs confied dispatcher

`Dispatchers.Unconfined`调度器会在当前线程开启协程的运行，直到遇到第一个挂起函数；在挂起后，在恢复协程时，其恢复的线程将由其调用该挂起函数来决定；`Dispatchers.Unconfined`适用于即不消耗CPU时间，也不需要在特定线程中共享(比如更新UI)的操作。

另一方面，默认情况下，调度器从外部CoroutineScope继承。尤其是runBlocking协程的默认调度程序仅限于调用程序线程，因此继承它的效果是通过可预测的FIFO调度将执行限制到该线程。

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
        println("Unconfined      : I'm working in thread ${Thread.currentThread().name}")
        delay(500)
        println("Unconfined      : After delay in thread ${Thread.currentThread().name}")
    }
    launch { // context of the parent, main runBlocking coroutine
        println("main runBlocking: I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("main runBlocking: After delay in thread ${Thread.currentThread().name}")
    }    
}
```

输出结果：

```
Unconfined      : I'm working in thread main
main runBlocking: I'm working in thread main
Unconfined      : After delay in thread kotlinx.coroutines.DefaultExecutor
main runBlocking: After delay in thread main
```

因此，协程的Context是继承于`runBlocking{…}`的Main线程，然而在unconfined 中恢复delay挂起函数时是运行`default`线程的；

>`unconfined `调度器是一种高级机制，在某个情况下可能有所帮忙，因为这不需要为其协程指定运行的线程；但这样可能会导致意外的错误，因为有些操作是必须指定线程的；因此在通用代码上，一般不会使用`unconfined `调度器。

#### 调试协程和线程(Debugging coroutines and threads)

协程可以在一个线程中挂起，又在另外一个线程上恢复；如果没有特殊的工具，即使只有一个线程的调度器也很难知道当前协程正在做什么。

##### 在IDEA上调试

Kotlin插件的协同程序调试器简化了IntelliJ IDEA中协同程序的调试(需要kotlinx-coroutines-core的版本1.3.8及以上)。

目前在AndroidStudio上不行，暂略；

##### 打log调试

在每个log中都打印出线程名称即可；

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking<Unit> {
    val a = async {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() * b.await()}")    
}
```

输出结果：

```
[main @coroutine#2] I'm computing a piece of the answer
[main @coroutine#3] I'm computing another piece of the answer
[main @coroutine#1] The answer is 42
```

#### 线程之间的跳转(Jumping between threads)

运行下面程序（添加JVM参数：-Dkotlinx.coroutines.debug）

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }    
}
```

输出结果：

```
[Ctx1 @coroutine#1] Started in ctx1
[Ctx2 @coroutine#1] Working in ctx2
[Ctx1 @coroutine#1] Back to ctx1
```

使用`runBlocking`显式指定Context, 另外一个使用`withContext`来更改协程的Context但仍然保持同一个协程中；

>上面例子中使用use方法，是kotlin标准库的， 它能确保不再需要时能释放掉由`newSingleThreadContext`创建的线程。

#### context里的Job(Job in the context)

协程的`Job`是context的一部分，可使用`coroutineContext[Job]`表达式来读取；

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    println("My job is ${coroutineContext[Job]}")    
}
```

输出结果： 

```
My job is "coroutine#1":BlockingCoroutine{Active}@21a06946
```

#### 子协程（Children of a coroutine）

在某个协程域下的协程A下创建一个协程B时， 新协程B会自动继承于该协程域的context，并且新协程B的Job是属于协程A的子Job；意味着协程A取消时， 其协程A下所有子协程（包含协程B）都会递归进行取消；

但是，可通过下面2种方式来显式重写协程的父子关系: 

1. 创建新协程时，进行显式指定其协程域，那么新协程不会继承其父域；比如 GlobalScope.launch.
2. 创建新协程时，传入一个`Job`对象给新协程， 那么新协程会继承于传入的Job对象的context，并且其子协程；

在上面这2种方式中， 新启动的协程和其启动范围无关，都能独立运行；或者说其启动范围的协程取消时，并不会影响新启动的协程；

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        // it spawns two other jobs
        launch(Job()) { 
            println("job1: I run in my own Job and execute independently!")
            delay(1000)
            println("job1: I am not affected by cancellation of the request")
        }
        // and the other inherits the parent context
        launch {
            delay(100)
            println("job2: I am a child of the request coroutine")
            delay(1000)
            println("job2: I will not execute this line if my parent request is cancelled")
        }
    }
    delay(500)
    request.cancel() // cancel processing of the request
    delay(1000) // delay a second to see what happens
    println("main: Who has survived request cancellation?")
}
```

输出结果：

```
job1: I run in my own Job and execute independently!
job2: I am a child of the request coroutine
job1: I am not affected by cancellation of the request
main: Who has survived request cancellation?
```

可以看到job1里并没有随着其启动范围的协程的取消而取消；

#### 父责任(Parental responsibilities)

一个协程会一直等待其所有子协程完成后，才能完成；父级不必显式跟踪它启动的所有子级，也不必使用`Job.join`在最后等待它们；

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        repeat(3) { i -> // launch a few children jobs
            launch  {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children that are still active")
    }
    request.join() // wait for completion of the request, including all its children
    println("Now processing of the request is complete")
}
```

输出结果：

```
request: I'm done and I don't explicitly join my children that are still active
Coroutine 0 is done
Coroutine 1 is done
Coroutine 2 is done
Now processing of the request is complete
```

从结果来看， request只需要等待自身完成即可，会等到所有子协程都完成后，才完成；并不需要显式来等待子协程的完成；

#### 协程的命名（Naming coroutines for debugging）

一般开发中， 协程的的自动分配的名字是足够我们来查看相关信息和关联的；但对于一些特定的协程(做一些特殊操作的后台任务)， 显式命名其协程的名字对我们调试起来更便捷；协程的`CoroutimeName`和线程的名字的作用是一样的；当调试模式开，它会显示在线程的名称里；

命名方式： 在创建新协程里，传入`CoroutineName`对象即可；

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking(CoroutineName("main")) {
    log("Started main coroutine")
    // run two background value computations
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(1000)
        log("Computing v2")
        6
    }
    log("The answer for v1 / v2 = ${v1.await() / v2.await()}")    
}
```

输出结果：

```
[main @main#1] Started main coroutine
[main @v1coroutine#2] Computing v1
[main @v2coroutine#3] Computing v2
[main @main#1] The answer for v1 / v2 = 42
```

#### 组合`context`元素(Combining context elements)

有时我们需要定义多个元素定义其协程的context. 我们可使用 `+` 来操作它；

同时定义其调度器和名称：

```kotlin
launch(Dispatchers.Default + CoroutineName("test")) {
    println("I'm working in thread ${Thread.currentThread().name}")
}
```

#### 协程域

我们正在编写一个Android应用程序，并在Android的Activity中启动各种协程，以执行异步操作来获取和更新数据、制作动画等。当Activity销毁时，必须取消所有这些协程，以避免内存泄漏。比较笨的方法，我们持有所有协程的Job，来手动进行取消。但`Kotlinx.coroutines`提供一个抽象封装： `CoroutineScope`（协程域）；

我们可创建一个与Activity的生命周期绑定的协程域来管理所有协程的生命周期；协程域可用`CoroutineScope()`或`MainScope工厂函数来创建， 前者创建一个通用域， 而后续为UI创建域并且默认使用`Dispatcher.Main`为默认调度器；

```kotlin
class Activity {
    private val mainScope = MainScope()

    fun destroy() {
        mainScope.cancel()
    }
    // to be continued ...
```

现在就可以使用`mainScope`来创建协程了：

```kotlin
import kotlinx.coroutines.*

class Activity {
    private val mainScope = CoroutineScope(Dispatchers.Default) // use Default for test purposes
    
    fun destroy() {
        mainScope.cancel()
    }

    fun doSomething() {
        // launch ten coroutines for a demo, each working for a different time
        repeat(10) { i ->
            mainScope.launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
                println("Coroutine $i is done")
            }
        }
    }
} // class Activity ends

fun main() = runBlocking<Unit> {
    val activity = Activity()
    activity.doSomething() // run test function
    println("Launched coroutines")
    delay(500L) // delay for half a second
    println("Destroying activity!")
    activity.destroy() // cancels all coroutines
    delay(1000) // visually confirm that they don't work    
}
```

输出结果：

```
Launched coroutines
Coroutine 0 is done
Coroutine 1 is done
Destroying activity!
```

可以看到，Activity调用destroy时(最终是调用`mainScope.cancel()`)，其协程域下的协程都取消了。

#### 线程本地数据(Thread-local data)

有时能把线程本地的数据能传给协程，或协程之间进行传递是很方便； 但进行手动传递的话会导致样板代码，因为协程并没绑定特定的线程；

使用ThreadLocal的asContextElement扩展方法可解决该问题：

```kotlin
import kotlinx.coroutines.*

val threadLocal = ThreadLocal<String?>() // declare thread-local variable

fun main() = runBlocking<Unit> {
    threadLocal.set("main")
    println("Pre-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    val job = launch(Dispatchers.Default + threadLocal.asContextElement(value = "launch")) {
        println("Launch start, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        yield()
        println("After yield, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    }
    val job1 = launch(Dispatchers.Default) {
        println("Launch start, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        yield()
        println("After yield, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    }
    job.join()
    job1.join()
    println("Post-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")    
}
```

输出结果：

```
Pre-main, current thread: Thread[main @coroutine#1,5,main], thread local value: 'main'
Launch start, current thread: Thread[DefaultDispatcher-worker-2 @coroutine#3,5,main], thread local value: 'null'
Launch start, current thread: Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main], thread local value: 'launch'
After yield, current thread: Thread[DefaultDispatcher-worker-2 @coroutine#3,5,main], thread local value: 'null'
After yield, current thread: Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main], thread local value: 'launch'
Post-main, current thread: Thread[main @coroutine#1,5,main], thread local value: 'main'
```

在这个例子里， 我们在Default调度器里启动一个新协程， 该协程会运行在和main不同的线程, 但它仍然可访问 thread local里的值(指定的threadLocal.asContextElement(value = "launch")；而job1是没设置，则为null；

由于编程时，很容易忘记去设置相应的Context元素， 如果运行协程的线程不同，则从协程访问的线程局部变量可能会有意外值。为了避免这种情况，建议使用[ensuRepresentation](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-present.html)方法，并在使用不当时快速失效。

`ThreadLocal`对于Kotlin的协程提供一流的支持；它有一个关键的限制： 当 thread-local更换时，新值并不会传递给协程的调用者（因为context elembent并不会跟踪所有threadlocal对象），并且在下一次挂起时，其更新的值会丢失；在协程中，使用`withContext`可更新`thread-local`的值。

> 注：yield 是挂起协程，让协程放弃本次 cpu 执行机会让给别的协程，当线程空闲时再次运行协程。

### 五、异步流(Asynchronous Flow)

`async`挂起方法只能返回一个值，但我们怎么能在异步中才返回多个值？就需要用到Kotlin Flow.

#### 多个值的表示(Representing multiple values)

Kotlin里使用`collections`来表示多个值， 比如List：

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}
```

##### 序列(Sequences﻿)

如果我们使用一些CPU消耗阻塞代码计算数字（每次计算需要100毫秒），那么我们可以使用`Sequence`表示数字：

```kotlin
fun simple(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it
        yield(i) // yield next value
    }
}

fun main() {
    simple().forEach { value -> println(value) } 
}
```

输出：

```
1
2
3
```

但上面每隔100ms输出一个数字，但会阻塞Main线程；

##### 挂起函数(Suspending functions)

这时可使用挂起函数来防止阻塞线程并返回一个数组；

```kotlin
suspend fun simple(): List<Int> {
    delay(1000) // pretend we are doing something asynchronous here
    return listOf(1, 2, 3)
}

fun main() = runBlocking<Unit> {
    simple().forEach { value -> println(value) } 
}
```

该程序在会等待1秒后输出结果；

##### Flows

在上面使用`List<Int>`结果类型，意味着一次返回所有值。为了能看到在异步运行，我们使用`Flow<Int>`类型，该类型类似于`Sequence<Int>`;

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
    println("end")
}
```

输出结果：

```
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
end
```

该程序每隔100ms输出数字并且没阻塞Main线程；

下面为使用`Flow`后的不同地方：

- Flow的构建函数是`flow`；
- 在`flow {...}`代码块里是可以挂起其它挂起函数的；
- `simple` 函数不再标记为suspend的。
- 从flow中发射数据使用[emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html)函数。
- 从flow从收集数据使用[collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 函数。

#### Flows是冷启动（Flows are cold）

`Flows`类似`Sequences`一样，直到flow 调用 `collect`方法时，才会运行该flow构建方法；

```kotlin
     
fun simple(): Flow<Int> = flow { 
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = simple()
    println("Calling collect...")
    flow.collect { value -> println(value) } 
    println("Calling collect again...")
    flow.collect { value -> println(value) } 
}
```

输出结果：

```
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3
```

这是`simple`函数(返回一个`flow`)没有用suspend修饰的原因，simple()本身很快就会返回而不用等待任何东西；每次在flow上调用`collect`的时候`flow`才会开始执行，这就是为什么当我们再次调用collect的时候会看到输出 "Flow started"。

#### `Flow`取消的基础(Flow cancellation basics)

Flow的取消是跟随协程的取消的， 通常情况下， 当Flow被挂起在一个可取消的挂起函数(比如`delay`)中时，其Flow的collect动作可被取消; 

```kotlin
fun simple(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)          
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) { // Timeout after 250ms 
        simple().collect { value -> println(value) } 
    }
    println("Done")
}
```

输出结果：

```
Emitting 1
1
Emitting 2
2
Done
```

数字3由于提前取消了，并没有打印出来；

#### 创建 `Flow`（Flow builders）

`flow {...}`是基本创建方式，其它方式如下：

- `flowOf`创建定义输出固定的一组值：flowOf(1, 2, 3)

- 不同的集合（collection）和序列（sequence）可以调用扩展函数`.asFlow`转换成`flow`。

  ```kotlin
  // Convert an integer range to a flow
  (1..3).asFlow().collect { value -> println(value) }
  ```

#### 中间流的操作符(Intermediate flow operators﻿)

flow可以通过操作符来转换，就像你将要使用的`collection`和`sequence`一样。操作符会适用于上游（upstream）的flow和返回一个下游（downstream）的flow; 这些操作符都冷启动的，并且不是挂起函数，因此能快速返回最新定义的flow。

基础操作符是 `map` 和`filter`；与`sequences`最重要的区别在于这个操作符里代码块是可挂起函数的；

比如：可以使用map操作符将传入请求流映射到结果，即使执行请求是由挂起函数实现的长时间运行的操作：

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
```

输出：

```
response 1
response 2
response 3
```

#### 变换操作符(Transform opterator)

变换操作符中，最常用是`transform`；它可用于模仿像map和filter的简单变换，也可用于复杂的转换；使用`transform`操作符可发出任意次数的任意值；

例如，使用transform，我们可以在执行长时间运行的异步请求之前发出一个字符串，并在其之后发出一个响应：

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .transform { request ->
            emit("Making request $request") 
            emit(performRequest(request)) 
            emit(1.8F) 
        }
        .collect { response -> println(response) }
}
```

输出结果：

```
Making request 1
response 1
1.8
Making request 2
response 2
1.8
Making request 3
response 3
1.8
```

#### 个数限制操作符(Size-limiting operators)

当个数限制达到时，`take`操作符会取消执行。协程里的取消会抛出一个异常，需要处理都应该添加`try {...} finally {}`代码块来处理；

```kotlin
fun numbers(): Flow<Int> = flow {
    try {                          
        emit(1)
        emit(2) 
        println("This line will not execute")
        emit(3)    
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers() 
        .take(2) // take only the first two
        .collect { value -> println(value) }
}    
```

可以看到输出1，2后，该flow方法不再执行了；

输出：

```
1
2
Finally in numbers
```

#### 终结操作符(Terminal flow operators)

终结操作符在flow是一个挂起函数（不会马上返回），而 最基本的就是`collect`操作符；还有一些更便捷终结操作符：

- 转化为其它集合： [toList](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/to-list.html) 和 [toSet](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/to-set.html)；
- 获取第一个结果的`first`操作符以及确保发出只能一个值的`single`操作符；
- 缩减flow为一个值的： `reduce` 和 `fold`; 

```kotlin
val sum = (1..5).asFlow()
    .map { it * it } // squares of numbers from 1 to 5                           
    .reduce { a, b -> 
        println("a=$a")
        println("b=$b")
        a + b } // sum them (terminal operator)
println(sum)
```

输出：

```
a=1
b=4
a=5
b=9
a=14
b=16
a=30
b=25
55
```

可以看到a第一次为第1和第2个元素，后面a为前一轮的结果；而fold其实只是比reduce多一个初始值而已；

#### Flows是顺序的(Flows are sequential)

除了操作多个flow操作符外，每个flow的collection运行都是顺序的；在调用终结操作符时，会直接进行收集结果。默认情况下，不会启动新的协程。每个值都是由上游到下游的中间操作符一步一步往下传，最终分发给终结操作符；

```kotlin
fun main() = runBlocking<Unit> {

    (1..5).asFlow()
        .filter {
            println("Filter $it")
            it % 2 == 0              
        }              
        .map { 
            println("Map $it")
            "string $it"
        }.collect { 
            println("Collect $it")
        }                      
}
```

输出结果：

```
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
```

#### Flow上下文(Flow Context)

Flow的Collecttion会一直运行在调用它的协程的Context中。比如下面的例子，不管simple()是怎么实现的， 整个flow运行都会在`withContext(Context)`指定的协程下运行；

```kotlin
withContext(context) {
    simple().collect { value ->
        println(value) // run in the specified context
    }
}
```

Flow这个属性称为上下文保持；

所以，默认上

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    log("Started simple flow")
    for (i in 1..3) {
        emit(i)
    }
}  

fun main() = runBlocking<Unit> {
    simple().collect { value -> log("Collected $value") } 
}   
```

输出：

```
[main @coroutine#1] Started simple flow
[main @coroutine#1] Collected 1
[main @coroutine#1] Collected 2
[main @coroutine#1] Collected 3
```

可以看到整个flow的操作都main的@coroutine#1协程里。 这对于那些不关心运行上下文且不阻塞调用者的快速运行代码和异步代码来说，这种默认设置非常不错。

##### `withContext`的错误使用(Wrong emission `withContext`)

然而有些需要长时间运行的可能需要运行在`Dispatcher.Default`，或者需要切换到UI线程里更新UI。 一般来说，我们会**使用`withContext`来切换`context`让其运行不同的线程**， 但在`flow {...}`语句中又遵守“上下文保持”的原则，因此不允许发射(`emit`)到其它`context`中，会直接抛`IllegalStateException`；

```kotlin
fun simple(): Flow<Int> = flow {
    // The WRONG way to change context for CPU-consuming code in flow builder
    kotlinx.coroutines.withContext(Dispatchers.Default) {
        for (i in 1..3) {
            Thread.sleep(100) // pretend we are computing it in CPU-consuming way
            emit(i) // emit next value
        }
    }
}

fun main() = runBlocking<Unit> {
    simple().collect { value -> println(value) } 
}   
```

运行时，会报以下异常：

```
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@5511c7f8, BlockingEventLoop@2eac3323],
		but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@2dae0000, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
	at ...
```

##### `flowOn`操作符（flowOn operator）

使用`flowOn`操作符能改变发射flow的Context; 

```kotlin
fun simple(): Flow<Int> = flow {
    log("flow")
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it in CPU-consuming way
        log("Emitting $i")
        emit(i) // emit next value
    }
}.flowOn(Dispatchers.Default) // RIGHT way to change context for CPU-consuming code in flow builder

fun main() = runBlocking<Unit> {
    simple().map { 
            log("Map $it")
            "string $it"
        }.flowOn(Dispatchers.IO)
    .collect { value ->
        log("Collected $value") 
    } 
}           
```

输出结果：

```
[DefaultDispatcher-worker-2 @coroutine#3] flow
[DefaultDispatcher-worker-2 @coroutine#3] Emitting 1
[DefaultDispatcher-worker-1 @coroutine#2] Map 1
[main @coroutine#1] Collected string 1
[DefaultDispatcher-worker-2 @coroutine#3] Emitting 2
[DefaultDispatcher-worker-1 @coroutine#2] Map 2
[main @coroutine#1] Collected string 2
[DefaultDispatcher-worker-2 @coroutine#3] Emitting 3
[DefaultDispatcher-worker-1 @coroutine#2] Map 3
[main @coroutine#1] Collected string 3
```

可以看到开始的flow{}，map 和 collection都运行在不同的协程； 

需要注意的是： 不管`flow{...}`跑在什么线程，其collect方法块都会运行在Main线程的； 

#### 缓冲(Buffering)

在不同的协程里运行不同部分的flow是很有帮助的，特别是长时间运行的任务； 

在下面的任务中，需要100ms来发射一个数据，而Colloection则需要300ms来运行一个数据； 

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}
```

输出运行时间：

```
1
2
3
Collected in 1229 ms
```

##### `Buffer`操作符

接下来使用`buffer`操作符将其flow流不按原来的顺序运行，而是让其并发运行； 

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
    simple()
        .buffer() // buffer emissions, don't wait
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
	}  
    println("Collected in $time ms")
}
```

输出结果：

```
1
2
3
Collected in 1071 ms
```

可以看到，除了发射第一个数据需要100ms外，后面2个数据的发射是和collection并发的； 

##### `conflate`操作符

当flow的部分结果或状态更新时，可能不需要去收集每个结果，只需要收集最近的结果即可； 

比如下面的例子里，由于collector部分是很慢的，导致有一些值在更新时，它还没空去处理； 使用`conflate`操作可跳过collector的收集(collector无法马上处理时)； 

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple()
            .conflate() // conflate emissions, don't process each one
            .collect { value -> 
                delay(300) // pretend we are processing it for 300 ms
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}
```

输出结果：

```
1
3
Collected in 758 ms
```

我们可以看到第一个结果还在collector处理时，第三个结果已经出来了，因此collector直接只收集了最后的结果； 

##### 只处理最后一个结果

在发射数据和收集数据处理都很慢的情况下， `conflate`操作符是其中一种提高运行效率的方法，因为它可忽略其中一些值； 

而另外一种提高运行速度的方式是当新结果收集到时，取消当前collector的运行并重启它； 做这种操作的为 `xxxLatest`系列操作符；

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple()
            .collectLatest { value -> // cancel & restart on the latest value
                println("Collecting $value") 
                delay(300) // pretend we are processing it for 300 ms
                println("Done $value") 
            } 
    }   
    println("Collected in $time ms")
}
```

输出结果：

```
Collecting 1
Collecting 2
Collecting 3
Done 3
Collected in 741 ms
```

#### Flow的组合(Composing multiple flows)

##### `zip`操作符

`zip`操作符可组合2个flows的值；

```kotlin
fun main() = runBlocking<Unit> { 

    val nums = (1..3).asFlow() // numbers 1..3
    val strs = flowOf("one", "two", "three") // strings 
    nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string
        .collect { println(it) } // collect and print
}
```

输出结果：

```
1 -> one
2 -> two
3 -> three
```

可以看到nums和strs的2个flows进行组合结果完，再发射给collector; 

##### `combile`操作符

当flow表示变量或操作的最新值时，其中某个值变化时，可能需要进行重新计算； 简单就是上游任何一个flow发射一个结果时，都需要重新算； 

```kotlin
fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
    val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms          
    val startTime = System.currentTimeMillis() // remember the start time 
    nums.combine(strs) { a, b -> "$a -> $b" } // compose a single string with "combine"
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果：

```
1 -> one at 452 ms from start
2 -> one at 646 ms from start
2 -> two at 854 ms from start
3 -> two at 947 ms from start
3 -> three at 1255 ms from start
```

需要注意的是：combine的第一次计算时，需要2个flow都要有结果，有可能运行快的flow已经是发射不只一个结果，以最后结果以准：

```kotlin
fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
    val strs = flowOf("one", "two", "three").onEach { delay(700) } // 更改时间为700ms         
    val startTime = System.currentTimeMillis() // remember the start time 
    nums.combine(strs) { a, b -> "$a -> $b" } // compose a single string with "combine"
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果：

```
2 -> one at 754 ms from start
3 -> one at 949 ms from start
3 -> two at 1456 ms from start
3 -> three at 2157 ms from start
```

可以看到，第一次打印时，nums的flow已经发射2出来了； 

#### 展平流(Flatterning flows)

Flow代表一系列异步收到的值，所以你很可能遇到一个值又触发产生另一个Flow的情况,比如我们可以使用下面的函数每隔500ms产生一个字符串：

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500) // wait 500 ms
    emit("$i: Second")
}
```

现在有一个flow会发射三个整数出来，并且调用`requestFlow`

```kotlin
(1..3).asFlow().map { requestFlow(it) }
```

这里我们得到一个嵌套的flow(`Flow<Flow<String>>`), 为了进一步处理这个嵌套的flow，我们需要将改成单个flow来运行；flatten及flatMap就是应对这种情况的， 然而由于flow的异步性，将会存在不同形式的flatten，同样对于flow也有一系列的flatten操作符。

##### `flatMapConcat﻿`

连接模式由`flatMapConcat` 与`flattenConcat`操作符实现。它们是相应序列操作符最相近的类似物。它们在等待内部流完成之后才开始收集下一个值，如下面的示例所示：

```kotlin
fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapConcat { requestFlow(it) }                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果：

```
1: First at 121 ms from start
1: Second at 622 ms from start
2: First at 727 ms from start
2: Second at 1227 ms from start
3: First at 1328 ms from start
3: Second at 1829 ms from start
```

可以看到`flatMapConcat`可以让2个嵌套的flow按顺序输出；

而`flattenConcat`效果是一样，但用法稍稍不同：

```kotlin
fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .map { requestFlow(it) } 
        .flattenConcat()
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

输出结果是上面一致；

`flatMapMerge﻿`

另一种连接模式就是并发收集所有传入的值，并合并这些值到单独的flow中，以便尽快发射数据；  它由 `flatMapMerge` 与 `flattenMerge`操作符实现。他们都接收可选的用于限制并发收集的流的个数的 `concurrency` 参数（默认情况下，它等于 [DEFAULT_CONCURRENCY](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-d-e-f-a-u-l-t_-c-o-n-c-u-r-r-e-n-c-y.html)）

```kotlin
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapMerge { requestFlow(it) }                                                            
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
```

输出结果：

```
1: First at 136 ms from start
2: First at 231 ms from start
3: First at 333 ms from start
1: Second at 639 ms from start
2: Second at 732 ms from start
3: Second at 833 ms from start
```

`flatMapMerge` 的并发性质很明显：

> 注意，flatMapMerge 会顺序调用代码块（本示例中的 { requestFlow(it) }），但是并发收集结果流，相当于执行顺序是首先执行 map { requestFlow(it) } 然后在其返回结果上调用 flattenMerge。

##### `flatMapLatest`

与 `collectLatest`操作符类似，`flatMapLatest`在新值到来时， 会立即取消当前`flatMapLatest`块的代码运行； 

```kotlin
val startTime = System.currentTimeMillis() // remember the start time 
(1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
    .flatMapLatest { requestFlow(it) }                                                               
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
```

输出结果：

```
1: First at 142 ms from start
2: First at 322 ms from start
3: First at 425 ms from start
3: Second at 931 ms from start
```

> 注意，flatMapLatest 在一个新值到来时取消了块中的所有代码 (本示例中的 { requestFlow(it) }）。 这在该特定示例中不会有什么区别，由于调用 requestFlow 自身的速度是很快的，不会发生挂起， 所以不会被取消。然而，如果我们要在块中调用诸如 delay 之类的挂起函数，这将会被表现出来。

#### 流异常(`Flow exception`)

当操作符的发射器或代码块发生异常时，流的收集可以带有异常的完成；

##### 收集器的Try catch

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->         
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}
```

输出结果:

```kotlin
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

这段代码成功的在末端操作符 [collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 中捕获了异常，并且， 如我们所见，在这之后不再发出任何值.

##### 捕获一切异常

前面的示例实际上捕获了在发射器或任何过渡或末端操作符中发生的任何异常。

```kotlin
fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "[map]Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}            
```

输出：

```
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: [map]Crashed on 2
```

#### 异常透明性(Exception transparency﻿)

流必须*对异常透明*，即在 `flow { ... }` 构建器内部的 `try/catch` 块中[发射](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html)值是违反异常透明性的。这才能保证收集器抛出来的异常能被捕获到； 

发射器可以使用`catch`操作符来保留此异常的透明性并允许封装它的异常处理。catch 操作符的代码块可以分析异常并根据捕获到的异常以不同的方式对其做出反应：

- 可以使用`throw`重新抛出异常。
- 可以使用`catch`代码块中的`emit`将异常转换为值发射出去。
- 可以将异常忽略，或用日志打印，或使用一些其他代码处理它。

```kotlin
simple()
    .catch { e -> emit("Caught $e") } // emit on exception
    .collect { value -> println(value) }
```

输出结果：

```
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: [map]Crashed on 2
```

可以看到我们没try-catch,但可以通过 `catch`操作符来捕获它；

##### 透明捕获(Transparent catch)

`catch`过渡操作符遵循异常透明性，因此它仅能捕获上游异常(在`catch`操作符上游的异常)，如果是下游抛出一个异常，那么该异常会逃逸：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}   
```

输出结果：

```
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
 at FileKt$main$1$2.emit (File.kt:15) 
 at FileKt$main$1$2.emit (File.kt:14) 
 at kotlinx.coroutines.flow.FlowKt__ErrorsKt$catchImpl$2.emit (Errors.kt:158) 
```

可以看到`collect{...}`里的异常无法捕获； 

##### 声明式捕获

可将 catch 操作符的声明性与处理所有异常的期望相结合，将 collect 操作符的代码块移动到 onEach 中，并将其放到 catch 操作符之前。收集该流必须由调用无参的 collect() 来触发：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .onEach { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
        .catch { e -> println("Caught $e") }
        .collect()
}
```

输出结果：

```
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Collected 2
```

#### 流完成

当流收集完成时（普通情况或异常情况），它可能需要执行一个动作， 可通过两种方式完成：命令式或声明式；

##### 命令式`finally`块

除了 `try`/`catch` 之外，收集器还能使用 `finally` 块在 `collect` 完成时执行一个动作

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
} 
```

##### 声明式处理

对于声明式，流拥有 [onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html) 过渡操作符，它在流完全收集时调用。

改写上面命令式例子：

```kotlin
simple()
    .onCompletion { println("Done") }
    .collect { value -> println(value) }
```

输出：

```
1
2
3
Done
```

`onCompletion `的主要优点是其 lambda 表达式的可空参数 Throwable 可以用于确定流收集是正常完成还是有异常发生。在下面的示例中 simple 流在发射数字 1 之后抛出了一个异常：

```kotlin
fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}  
```

输出：

```
1
Flow completed exceptionally
Caught exception
```

onCompletion 操作符与 catch 不同，它不处理异常。我们可以看到前面的示例代码，异常仍然流向下游。它将被提供给后面的 onCompletion 操作符，并可以由 catch 操作符处理。

##### 命令式还是声明式

现在我们知道如何收集流，并以命令式与声明式的方式处理其完成及异常情况。 这里有一个很自然的问题是，哪种方式应该是首选的？为什么？ 作为一个库，我们不主张采用任何特定的方式，并且相信这两种选择都是有效的， 应该根据自己的喜好与代码风格进行选择。

##### 成功完成

与 [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) 操作符的另一个不同点是 [onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html) 能观察到所有异常并且仅在上游流成功完成（没有取消或失败）的情况下接收一个 `null` 异常。

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> println("Flow completed with $cause") }
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}
```

我们可以看到完成时 cause 不为空，因为流由于下游异常而中止：

```
1
Flow completed with java.lang.IllegalStateException: Collected 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
```

##### 启动流

使用流表示来自一些源的异步事件是很简单的。

```kotlin
// 模仿事件流
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect() // <--- 等待流收集
    println("Done")
}     
```

输出结果：

```
Event: 1
Event: 2
Event: 3
Done
```

由于我们使用`collect`末端操作符来收集数据， 所以collect会一直等待流被收集完才会往下运行；

这里可使用 `launchIn` 替换 `collect` 我们可以在单独的协程中启动流的收集，这样就可以立即继续进一步执行代码：

```kotlin
fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this) // <--- 在单独的协程中执行流
    println("Done")
}        
```

输出结果：

```
Done
Event: 1
Event: 2
Event: 3
```

`launchIn` 必要的参数 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) 指定了用哪一个协程来启动流的收集。在先前的示例中这个作用域来自 runBlocking 协程构建器，在这个流运行的时候，runBlocking 作用域等待它的子协程执行完毕并防止 main 函数返回并终止此示例。

在实际的应用中，作用域来自于一个寿命有限的实体。在该实体的寿命终止后，相应的作用域就会被取消，即取消相应流的收集。这种成对的 `onEach { ... }.launchIn(scope)` 工作方式就像 `addEventListener` 一样。而且，这不需要相应的 `removeEventListener` 函数， 因为取消与结构化并发可以达成这个目的。

> 注: launchIn 也会返回一个Job

#### 流取消检测

为方便起见，[流](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow.html)构建器对每个发射值执行附加的 [ensureActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/ensure-active.html) 检测以进行取消。 这意味着从 `flow { ... }` 发出的繁忙循环是可以取消的：

```kotlin
fun foo(): Flow<Int> = flow { 
    for (i in 1..5) {
        println("Emitting $i") 
        emit(i) 
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value -> 
        if (value == 3) cancel()  
        println(value)
    } 
}
```

仅得到不超过 3 的数字，在尝试发出 4 之后抛出 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)；

```
Emitting 1
1
Emitting 2
2
Emitting 3
3
Emitting 4
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job="coroutine#1":BlockingCoroutine{Cancelled}@6d7b4f4c
```

但是，出于性能原因，大多数其他流操作不会自行执行其他取消检测。 例如，如果使用 [IntRange.asFlow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/kotlin.ranges.-int-range/as-flow.html) 扩展来编写相同的繁忙循环， 并且没有在任何地方暂停，那么就没有取消的检测；

##### 让繁忙的流可取消

在协程处于繁忙循环的情况下，必须明确检测是否取消。 可以添加 `.onEach { currentCoroutineContext().ensureActive() }`， 但是这里提供了一个现成的 [cancellable](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/cancellable.html) 操作符来执行此操作：

```kotlin
          
fun main() = runBlocking<Unit> {
    (1..5).asFlow().cancellable().collect { value -> 
        if (value == 3) cancel()  
        println(value)
    } 
}
```



#### Flow的常用方法

| 方法名                         | 方法使用说明                                                 |
| ------------------------------ | ------------------------------------------------------------ |
| flow {...}                     | 用于构建flow；                                               |
| emit()                         | 用于从flow中发射数据;                                        |
| collect()                      | 用于从flow从收集数据， 属于终结操作符；                      |
| asFlow()                       | 用于不同的集合（collection）和序列（sequence）转为Flow；     |
| asFlow()                       | 用于把上游的结果进行一对一的转换;                            |
| map {...}                      | 把上游的结果进行一定的转换                                   |
| filter()                       | 用于把上游的结果进行一定的条件进行过滤，来决定是否发射给下游； |
| transform()                    | 可用于模仿像map和filter的简单变换，也可用于复杂的转换；使用`transform`操作符可发出任意次数的任意值； |
| take()                         | 用于限制发送给下游的个数；                                   |
| reduce()和flod()               | 用于缩减flow为一个值， fold其实只是比reduce多一个初始值而已；两者属于终结操作符； |
| flowOn(Diapatch)               | 用于切换发射数据的context的                                  |
| buffer()                       | 用于发射的协程与collector协程并发运行                        |
| conflate()                     | 用于发射的结果可跳过collector的收集(如果collector无法马上处理时)；但最后一个会收集到 |
| xxxLatest {...}                | mapLatest， collectLatest，flatMapLatest： 用于发射结果时，不管{...}里是否已处理完之前的值，就直接重启{...}方法块的代码； |
| zip(flow){..}                  | 组合2个flow的结果进行整合:每次需要2个flow都发射结果时，会触发zip |
| combile(flow){..}              | 组合2个flow的结果: 第一次合并时，需要2个flow都有结果(某个flow有多个时，取最新)，后面只要有flow有更新结果，combile都会触发 |
| flatMapConcat{...}             | 组合2个flow的结果: 可等待内部流({...})完成之后才收集下一个上游传入的值； |
| flatMapMerge﻿{...}              | 组合2个flow的结果: 并发收集结果流，相当于执行顺序是首先执行 map { ... } 然后在其返回结果上调用 flattenMerge。 |
| flatMapLatest{...}             | 组合2个flow的结果: 与 `collectLatest`操作符类似，`flatMapLatest`在新值到来时， 会立即取消当前`flatMapLatest`块的代码运行； |
| onEach{}                       | 进行声明式捕获，以便捕获上下游的异常；                       |
| onCompletion {}                | 进行流完成时的最终操作；相比try..finally{}, `onCompletion `的主要优点是其 lambda 表达式的可空参数 Throwable 可以用于确定流收集是正常完成还是有异常发生。 |
| catch{}                        | `catch`操作符来保留此异常的透明性并允许封装它的异常处理。catch 操作符的代码块可以分析异常并根据捕获到的异常以不同的方式对其做出反应， 但其下游的异常会逸出； |
| onEach { ... }.launchIn(scope) | 工作方式就像 `addEventListener` 一样。而且，这不需要相应的 `removeEventListener` 函数， 因为取消与结构化并发可以达成这个目的。 |



[Flow的相关方法文档](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/index.html)



### 相关函数

| 函数                           | 描述                                                         |      |
| :----------------------------- | :----------------------------------------------------------- | ---- |
| withContext(NonCancellable) {} | 启动不可取消的代码块                                         |      |
| withTimeout/withTimeoutOrNull  | 超时会进行取消其挂起方法                                     |      |
| measureTimeMillis{...}         | 返回该方法块所运行的总时间                                   |      |
| check(condition) {}            | 检测是否满足condition，不满足则直接抛出以运行其代码块{}的结果为错误信息的IllegalStateException异常 |      |
|                                |                                                              |      |



### 通道(Channel)

延期的值提供了一种便捷的方法使单个值在多个协程之间进行相互传输。 通道提供了一种在流中传输值的方法。

#### 通道基础

一个 [Channel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/index.html) 是一个和 `BlockingQueue` 非常相似的概念。其中一个不同是它代替了阻塞的 `put` 操作并提供了挂起的 [send](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/send.html)，还替代了阻塞的 `take` 操作并提供了挂起的 [receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html)。

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
        for (x in 1..5) channel.send(x * x)
    }
    // 这里我们打印了 5 次被接收的整数：
    repeat(5) { println(channel.receive()) }
    println("Done!")
}
```

输出结果：

```
1
4
9
16
25
Done!
```

#### 关闭与迭代通道

和队列不同，一个通道可以通过被关闭来表明没有更多的元素将会进入通道。 在接收者中可以定期的使用 `for` 循环来从通道中接收元素。

从概念上来说，一个 [close](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/close.html) 操作就像向通道发送了一个特殊的关闭指令。 这个迭代停止就说明关闭指令已经被接收了。所以这里保证所有先前发送出去的元素都在通道关闭前被接收到。

```kotlin
val channel = Channel<Int>()
launch {
    for (x in 1..5) channel.send(x * x)
    channel.close() // 我们结束发送
}
// 这里我们使用 `for` 循环来打印所有被接收到的元素（直到通道被关闭）
for (y in channel) println(y)
println("Done!")
```

#### 构建通道生产者

协程生成一系列元素的模式很常见。 这是 *生产者—消费者* 模式的一部分，并且经常能在并发的代码中看到它。 你可以将生产者抽象成一个函数，并且使通道作为它的参数，但这与必须从函数中返回结果的常识相违悖。

这里有一个名为 [produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html) 的便捷的协程构建器，可以很容易的在生产者端正确工作， 并且我们使用扩展函数 [consumeEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/consume-each.html) 在消费者端替代 `for` 循环：

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

fun main() = runBlocking {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
}
```











https://kotlinlang.org/docs/coroutines-guide.html

中文：https://www.kotlincn.net/docs/reference/coroutines/flow.html

https://cloud.tencent.com/developer/article/1603354

https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md

[协程框架概述](https://www.sukaidev.top/2021/02/28/c18f109d/)

