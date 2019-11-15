### [kotlin笔记] 委托

#### Interface

概念略，看看一般使用例子：

```kotlin
interface A {
    fun funA()
    
    //可实现默认的接口方法
    fun demoA() {
        println("Demo A")
    }
}

interface B {
    fun funB()
    
    //可实现默认的接口方法
    fun demoB() {
        println("Demo B")
    }
}

//多重继承/实现
class MyClass : A, B {
    override fun funA() {
        println("Hello A")
    }

    override fun funB() {
        println("Hello b")
    }

    //可重写接口的默认方法
    override fun demoA() {
        super.demoA()
        println("Hello Demo b")
    }
}

fun interfaceMemo() {
    val obj = MyClass();
    obj.funA()
    obj.funB()
    obj.demoA()
    obj.demoB()
}
```



#### 委托模式

该模式目的是管理对象的使用，一般解决问题是在直接访问对象时带来的问题， 那么想在访问一个类时做一些控制时， 就可以使用该模式；

##### VS 代理模式

代理模式： 代理模式下，目标对象从头到尾不会有任何的改变；代理方法中不会有任何业务相关的逻辑存在，更不会改变真正的逻辑实现。

委托模式： 委托模式不仅可以自由切换被委托者，甚至可以自己实现一段逻辑; 



##### 类委托

```kotlin
//创建接口
interface Base {
    fun print()
}
//实现此接口被委托的类
class BaseImpl1(val x: Int) : Base {
    override fun print() {
        print(x)
    }
}

//实现此接口被委托的类
class BaseImpl2(val x: String) : Base {
    override fun print() {
        print(x)
    }
}

//通过关键字 "by" 建立委托类
//Derived虽然继承了Base接口，但它并没直接实现接口方法，而是把直接实现委托b；
class Derived(b: Base) : Base by b

fun deriveDemo() {
    val b1 = BaseImpl1(100)
    val b2 = BaseImpl2("Hello Derived.")
    Derived(b1).print()
    Derived(b2).print()
}
```

##### 属性委托

```kotlin
class MyClass {
    //把属性委托给Delegate
    var p: String by Delegate()
}

//委托的类
class Delegate {
    //重载操作符
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String{
        return "$thisRef, There will delegate to ${property.name}."
    }
    //重载操作符
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        print("$thisRef, There will delegate to ${property.name}.")
    }
}

fun DelegateDemo() {
    val e = MyClass()
    print(e.p) //调用 getValue() 方法
    
    e.p = "Hello" ////调用 setValue() 方法
    print(e.p)
}
```

##### 懒初始化

对于一些对象，使用次数比较少，所以不需要一开始就初始化它，只要等真正使用它时，再进行初始化，这就是懒初始化；

这个用法对于我们来说，是非常好用的；

```kotlin
val lazyValue: String by lazy {
    //这里只能调用一次，
    print("init the value")
    "Hello"
}

fun lazyDemo() {
    print(lazyValue) //会打印"init the value"
    print(lazyValue) //不会打印"init the value"， 因为该值已经初始化好， 不会再进行laz方法里
}
```

###### vs lateinit

- lateinit针对var；bylazy 针对val（但不是所有by ... 都是针对val）。

- lateinit仅可用于nullable的类型，如String，如果是Int，是不行的。

- lateinit标示的值可被多次赋值；而by lazy标示的值仅在第一次被调用时赋值，且有且仅有一次；

- lateinit仅用于类中的成员属性；by lazy 可用于类中与函数中；

- lateinit支持Backing Fields机制(**set()/get()** )

  那么，如果想在**函数和类中**使用类似的lateinit，要怎么做呢？可以使用下面介绍 **Delegates.notNull<\*>()**

##### Observable

这时我们需要用一个Kotlin的内部类Delegaters, 通过它，我们可以观察一个值的变化；

其实这个对数据驱动来说，是非常有用的，类似jetpack提供的LiveData<>, 比如我们一个TextView里面的内容对应的字符串是mTextValue, 那边我们就可以通过为观察mTextValue的变化，同时设置给TextView进行显示；



```kotlin
class Data {
    var mTextValue: String by Delegates.observable("初始值") {
        property, oldValue, newValue ->  
        print("")
        //mTextView.setText(newValue)
    }
}

fun ObservableDemo() {
    val d = Data()
    d.mTextValue = "第一次赋值"
    d.mTextValue = "第二次赋值"
}
```

###### vs Delegates.vetoable()

与与observable不同的是这个回调会返回一个Boolean值，来决定此次属性值是否执行修改。

```kotlin
class Person{
    var address: String by Delegates.vetoable(initialValue = "NanJing", onChange = {property, oldValue, newValue ->
        println("property: ${property.name}  oldValue: $oldValue  newValue: $newValue")
        return@vetoable newValue == "BeiJing"
    })
}
```



##### 从map取值

```kotlin
class Website(val map: MutableMap<String, Any?>) {
    val name: String by map //以"name"为key来从map中取对应的值
    val url: String by map  //以"url"为key来从map中取对应的值
}

fun mapDemo() {
    var map: MutableMap<String, Any?> = mutableMapOf(
        "name" to "My Homepage",
        "url" to "wwww.homepage.com"
    )

    val site = Website(map)
    print(site.name) //输出：My Homepage
    print(site.url)  //输出：wwww.homepage.com
    
    map.put("name", "Baidu")
    map.put("url", "www.baidu.com")

    print(site.name) //输出：Baidu
    print(site.url)  //输出：www.baidu.com
}
```

##### notNull

这种委托保证提供一个非空的对象，但当前无法初始化；因此需要确保在使用前必须初始化；

```kotlin
class Go {
    var notNullValue: String by Delegates.notNull<String>()//只不知道不能为空
}

fun notNullDemo() {
    val go = Go()
	....
    go.notNullValue = "Hello" //使用前需要赋值， 否则会抛出IllegalStateException异常
    print(go.notNullValue)
}
```

可能有的人并没有看到notNull()有什么大的用处，先说下大背景吧就会明白它的用处在哪了？

**大背景:** 在Kotlin开发中与Java不同的是在定义和声明属性时必须要做好初始化工作，否则编译器会提示报错的，不像Java只要定义就OK了，管你是否初始化呢。我解释下这也是Kotlin优于Java地方之一，没错就是空类型安全，就是Kotlin在写代码时就让你明确一个属性是否初始化，不至于把这样的不明确定义抛到后面运行时。如果在Java你忘记了初始化，那么恭喜你在运行时你就会拿到空指针异常。

问题来了:** 大背景说完了那么问题也就来了，相比Java，Kotlin属性定义时多出了额外的属性初始化的工作。但是可能某个属性的值在开始定义的时候你并不知道，而是需要执行到后面的逻辑才能拿到。这时候解决方式大概有这么几种：

**方式A: 开始初始化的时给属性赋值个默认值**

**方式B: 使用Delegates.notNull()属性代理**

**方式C: 使用lateinit修饰属性**

以上三种方式有局限性，方式A就是很暴力直接赋默认值，对于基本类型还可以，但是对于引用类型的属性，赋值一个默认引用类型对象就感觉不太合适了。方式B适用于基本数据类型和引用类型，但是存在属性初始化必须在属性使用之前为前提条件。方式C仅仅适用于引用类型，但是也存在属性初始化必须在属性使用之前为前提条件。

**优缺点分析:**

| 属性使用方式                       | 优点                                               | 缺点                                                         |
| :--------------------------------- | :------------------------------------------------- | :----------------------------------------------------------- |
| 方式A(初始化赋默认值)              | 使用简单，不存在属性初始化必须在属性使用之前的问题 | 仅仅适用于基本数据类型                                       |
| 方式B(Delegates.notNull()属性代理) | 适用于基本数据类型和引用类型                       | 1、存在属性初始化必须在属性使用之前的问题； 2、不支持外部注入工具将它直接注入到Java字段中 |
| 方式C(lateinit修饰属性)            | 仅适用于引用类型                                   | 1、存在属性初始化必须在属性使用之前的问题； 2、不支持基本数据类型 |

**使用建议:** 如果能对属性生命周期做很好把控的话，且不存在注入到外部字段需求，建议使用方式B；此外还有一个不错建议就是方式A+方式C组合，或者方式A+方式B组合。具体看实际场景需求。
