## Kotlin 协程笔记

[TOC]

### 一、协程基础（Coroutine Basics）

#### 1. 第一个协程

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

> 开发者需要明白，协程是运行于线程上的，一个线程可以运行多个（可以是几千上万个）协程。线程的调度行为是由 OS 来操纵的，而协程的调度行为是可以由开发者来指定并由编译器来实现的。当协程 A 调用 delay(1000L) 函数来指定延迟1秒后再运行时，协程 A 所在的线程只是会转而去执行协程 B，等到1秒后再把协程 A 加入到可调度队列里。所以说，线程并不会因为协程的延时而阻塞，这样可以极大地提高线程的并发灵活度 

#### 2. 桥接阻塞与非阻塞

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

#### 3. 等待作业

上面程序runBlocking无法保证launch运行完，如果改这里delay时间为500L，只输出“Hello,”，而launch的生命周期早早结束。也就是延迟一段时间来等待另一个协程运行并不是一个好的选择，可以显式（非阻塞的方式）地等待协程执行完成 ：

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

#### 4. 结构化并发（Structured concurrency）

当使用 GlobalScope.launch 时，会创建一个顶层协程，是针对整个应用的生命周期的，尽管它是轻量级的，但仍然会造成一些内存资源消耗；

当我们忘记持有这个新顶层协程的引用，没显式关闭它，它一直在跑或delay中，这样就会造成泄漏，但如果每个都要去持有它，显式关闭它又是一个很麻烦的事；

这时一个更好的方案就是使用结构化并发来代替使用 GlobalScope.launch： 在我们正在执行的操作的特定范围内启动协同程序。 

**规则**：外部协程只有在其作用域中启动的所有协程完成后才会完成。

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



#### 5. 域的创建（Scope builder）

除了不同协程构建器提供的协程域，你也可以使用`coroutineScope`创建声明自己的域。它创建的一个协程作用域会直到所有开启的子协程完成后才结束。

[runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) VS [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)

共同点： 2个都会等子协程完成后才会完成； 

不同点：runBlocking会阻塞当前线程来等待， 是一个常规函数；而coroutineScope仅仅是挂起并会释放当前线程， 是一个挂起函数，也有一个含义就是coroutineScope只能在协程内或挂起函数里使用； 

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
Coroutine scope is over1
Task from coroutine scope
Task from runBlocking
Task from nested launch
Task from coroutine scope2
Coroutine scope is over
```

从结果来看，当运行到coroutineScope创建时， 当前线程会挂起，并不会往下执行，直到当前协程完成后才恢复往下执行； 

#### 6. 提取重构函数(Extract function refactoring)

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

But what if the extracted function contains a coroutine builder which is invoked on the current scope? In this case, the `suspend` modifier on the extracted function is not enough. Making `doWorld` an extension method on `CoroutineScope` is one of the solutions, but it may not always be applicable as it does not make the API clearer. The idiomatic solution is to have either an explicit `CoroutineScope` as a field in a class containing the target function or an implicit one when the outer class implements `CoroutineScope`. As a last resort, [CoroutineScope(coroutineContext)](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope.html) can be used, but such an approach is structurally unsafe because you no longer have control on the scope of execution of this method. Only private APIs can use this builder.  没看懂!!

#### 7. 协程是轻量级(Coroutines ARE light-weight)

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

#### 8. 全局协程类似于守护线程

GlobalScope 中启动了一个会长时间运行的协程:

```
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

输出结果?：

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```

launch 函数依附的协程作用域是 GlobalScope，而非 runBlocking 所隐含的作用域。在 GlobalScope 中启动的协程无法使进程保持活动状态，它们就像守护线程（当主线程消亡时，守护线程也将消亡）



### 二、取消和超时(Cancellation and Timeouts)

#### 1. 取消执行中的协程(Cancelling coroutine execution)

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

#### 2. 取消是协作的(Cancellation is cooperative)

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
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

 可以看到在调用cancelAndJoin()方法后，“..I'm sleeping..” 仍然在打印，直到5次打印才停止； 

#### 3. 使计算中的协程可取消(Making computation code cancellable)

- 使用 yield 函数，该函数会让运行当前协程的线程或线程池尽可能调度其它协程来运行；(测试过，不可靠) 
- 根据 isActive 来显式检查取消状态；

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

#### 4. 在`Finally`中释放资源 (Closing resources with `finally`)

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



#### 5. 运行不可取消的代码块(Run non-cancellable block)

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

#### 6. 超时(Timeout)

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

#### 7. 异步超时和资源(Asynchronous timeout and resources)

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

#### 1. 默认顺序(Sequential by default)

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

其中measureTimeMillis函数是返回当前代码块的运行时间(单位毫秒)； 

#### 2. 异步并发(Concurrent using async)

如果doSomethingUsefulOne和doSomethingUsefulTwo方法并没有依赖关系，我们想更快的返回结果，这时就需要使用到并发； 













































https://cloud.tencent.com/developer/article/1603354

https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md