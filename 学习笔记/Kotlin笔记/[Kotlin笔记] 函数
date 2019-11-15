### [Kotlin笔记] 函数

#### 函数式编程

定义： 函数式编程更加强调程序执行的结果而非执行的过程；

特点：

- 函数是第一等公民；
- 只用表达式，不用“语句”；//只想要结果
- 没有“副作用”， 不得修改外部变量的值；
- 不修改状态；
- 引用透明；//只要参数相同，返回结果总是相同的；

好处：

- 代码可以写得很精简；
- 抽象程序高，可以更方便的利用；
- 接近自然语言，易于理解；
- 易于并发编程；
- 更方便的代码管理；

#### kotlin Lambda 库函数

Kotlin 的每个函数都有特定的类型，函数类型由函数 的形参列表， ->, 返回值类型组成 ;  

栗子：

```kotlin
//定义一个乘方的函数
fun pow1(a: Int, b: Int) : Int {
    var result = 1
    for (i in 1..b) {
        result *= a
    }
    return result
}

fun funDemo() {
    //定义变量为函数类型
    var myFun: (Int, Int) -> Int
    myFun = ::pow1 //将pow函数赋值给myFun变量；"::"表示访问函数引用；
    print(myFun(2, 4)) //输出16
    //另外我们还可以把其它也是(Int, Int) -> Int的函数赋值给myFun
}

```



Lambda与局部函数：

由于函数也是一种类型，因此我们可以使用函数类型来作为形参类型传递；

```kotlin
val myFun: () -> Unit = {
    
}

val myGet: (String) -> Unit = {
    print("Hello... $it") //只有一个参数时，使用it来表达该参数变量
}

val tellWang: (String, Any) -> Boolean = {
        s, _ -> //"_"表示该变量不使用
    "Wang" == s
}

val tellLi: (String, Any) -> Boolean = {
        s, _ ->
    "Li" == s
}

fun lambdaDemo(param: (String, Any) -> Boolean) {
    val result = param("Li", "")
    print(result)
}

fun testLambda() {
    lambdaDemo(tellLi) //将局部函数作为参数传入
    lambdaDemo(tellWang) //将局部函数作为参数传入
}
```

Kotlin lambda的库函数

```kotlin
data class Person(val name: String, val age: Int)
var peoples = arrayListOf(Person("zhiyufe", 18), Person("Liyu", 30))

fun test() {
    print(peoples.filter { it.age > 25 })
    print(peoples.map { it.name})
}
```

###### 高阶 Lambda

高阶函数就是使用函数对象作为参数或返回值的函数；

```kotlin
fun jia(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    print(result)
}

fun testDemo() {
    //对于最后一个实参是Lambda表达式的，可把它挪到小括号外面来使用 {}形式使用；
    jia { a, b -> a + b }
    jia { a, b -> a * b }
    
    fun temp(a: Int, b: Int) :Int {
        return a - b /2
    }
    jia(::temp)
}
```

当 lambda是函数的唯一实参时， 就可以去掉空的小括号：

```kotlin
mButton.setOnClickListener { view: View ->
	view.sisibility = View.GONE
}
```

如果lambda的参数的类型可以被编译器推导出来，那么就可以省略它： 比如：

```kotlin
mButton.setOnClickListener { view ->
	view.sisibility = View.GONE
}
```

只有有一个参数，view 可去掉，使用it来表示：

```kotlin
mButton.setOnClickListener {
	it.sisibility = View.GONE
}
```

#### 库函数进阶

###### 返回函数的函数

```kotlin
fun getShippingCost(fruits: Fruits): (Order) -> Double {
    if (fruits == Fruits.APPLE) {
        return {order -> 2.3 * order.itemCount}
    } else if (fruits == Fruits.BANANA) {
        return {order -> 1.3 * order.itemCount}
    }

    return {order -> 1.0 * order.itemCount}
}

fun demo() {
    //返回计算苹果的函数
    val appCalculator = getShippingCost(Fruits.APPLE)
    //算出3个苹果的价格
    print(appCalculator(Order(3)))
}
```



###### 优化lambda-使用内联

每个非内联的lambda都会被编译成一个匿名类；但如果把lambda传给了标记为 inline 的 Kotlin 函数，那么就不会创建任何匿名类，说白了，该lambda的代码就直接复制粘贴到内联函数时， 而不是把Lambda当成一个函数调用；

例子可自行查看synchronized的源码；

```kotlin
@kotlin.internal.InlineOnly
public actual inline fun <R> synchronized(lock: Any, block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }

    @Suppress("NON_PUBLIC_CALL_FROM_PUBLIC_INLINE", "INVISIBLE_MEMBER")
    monitorEnter(lock)
    try {
        return block()
    }
    finally {
        @Suppress("NON_PUBLIC_CALL_FROM_PUBLIC_INLINE", "INVISIBLE_MEMBER")
        monitorExit(lock)
    }
}
```



###### 优化lambda-sequence

一般来说，链式操作可能会生成多个中间集合，从而导致性能问题；比如：

```kotlin
people.map{it.name}.filter{it.startsWith("A")}
```

map和filter都会创建一个列表，但实际上我们只需要最后一个列表；

可以调用 asSequence 把任何集合转为序列，进行相关操作后，再调用 toList 把序列转换回List：

```kotlin
perple.asSequence()
		.map{it.name}
		.filter{it.startsWith("A")}
		.toList()
```

集合 VS 序列

Collections和Sequences（集合与序列）是Kotlin提供的两种使用集合的方式；

集合是每次transform执行时立即计算的，并且计算结果会存储在一个新的集合里面。集合的每个transform是内联方法，例如集合的map方法：

而序列是延迟计算的，分为两种操作：中间转换及最终操作。中间转换不会立即执行，而是会存储操作本身，只有当最终操作执行时，才会按序列在每个item上执行中间操作，然后调用最终操作。中间操作（如map，distinct，groupBy等等）返回另一个序列而终端操作（如first，toList，count等等）没有。

从而也就是解释了为什么上面使用asSequence 可以优化性能；

集合与序列的对比，更详细请参考 [kotlin集合和序列](https://blog.csdn.net/tanwei4199/article/details/98052685)

###### 从lambda中返回-标签

对于return的解释： 表示它会从最近 enclosing function中退出；

如果要指定哪里退出，则需要使用标签：

```kotlin
fun foo(ints: List<Int>) {
    ints.forEach inner@ {
        //从inner@表达式中退出，下面"Hello there."也会打印出来；
        //如果没有@inner，则会直接退出foo方法；
        if (it == 0) return@inner 
       
        println(it)
    }
    println("Hello there.")
}
//如果上面 List里是 1, 0, 3, 4
//则输出： 1, 3, 4,Hello there.
```

