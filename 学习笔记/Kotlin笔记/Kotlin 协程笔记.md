## Kotlin 协程笔记

[TOC]

### 协程基础（Coroutine Basics）

#### 1． 第一个协程程序

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



####　２．桥接阻塞与非阻塞

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
    delay(500L)      // delaying for 2 seconds to keep JVM alive
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



https://cloud.tencent.com/developer/article/1603354

https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md