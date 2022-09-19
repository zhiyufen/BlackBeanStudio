### [Kotlin笔记] 泛型

概念： 是程序设计语言的一种风格或范式，泛型允许程序员在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型；

泛型是对类型的抽象，属于编译时多态，即“参数化类型”，为类型安全提供保证，消除类型强转的烦恼；

比如：

```kotlin
fun <T> go(worker: T) {
    //do something
} 

class Worker<T> {
    fun work(value: T) {
        //do something
    }
}
```



##### 泛型约束

```kotlin
fun <T: Something> go (a: T) {
	a.doSomething()
}
```

为一个类型参数指定多个约束：

```kotlin
fun <T> go (a: T) 
	where T: CharSequence, T: Appendable{
	a.doSomething()
}
```

注： 没有指定上界的类型形参将会使用Any?(可为空)这个默认的上界；



##### 协变/逆变

型变： 型变是为了解决类型安全问题，是防止更加泛化的实例调用某些存在危险操作的方法。

###### 协变

基本定义： Kotlin 中规定一个泛型协变类，在泛型形参前面加上out修饰后，那么修饰这个泛型形参在函数内部使用范围将受到限制只能作为函数的返回值或者修饰只读权限的属性。

标准形式：

```kotlin
interface Producer<out T> {//在泛型类型形参前面指定out修饰符
   val something: T//T作为只读属性的类型，这里T的位置也是out协变点
   fun produce(): T//T作为函数的返回值输出给外部,这里T的位置就是out协变点
}
```

变种形式：

```kotlin
interface Producer<out T> {
   val something: List<T>//即使T不是单个的类型，但是它作为一个泛型类型修饰只读属性，所以它所处位置还是out协变点
   
   fun produce(): List<Map<String,T>>//即使T不是单个的类型，但是它作为泛型类型的类型实参修饰返回值，所以它所处位置还是out协变点
}
```



###### 逆变

如果一个泛型类声明成逆变的，用in修饰泛型类的类型形参，在函数内部出现的位置只能是作为可变属性的类型或者函数的形参类型。相对于外部而言逆变是消费泛型参数的角色，消费者请求外部输入in。

```kotlin
interface Consumer<in T>{//在泛型类型形参前面指定in修饰符
   fun consume(value: T)
}

interface Consumer<in T>{//在泛型类型形参前面指定in修饰符
   var something: T //T作为可变属性的类型，这里T的位置也是in逆变点
   fun consume(value: T)//T作为函数形参类型，这里T的位置也就是in逆变点
}

```

###### 不变

不变看起来就是我们常用的普通泛型，它既没有in关键字修饰，也没有out关键字修饰。它就是普通的泛型，所以很明显它没有像协变、逆变那样那么多的条条框框，它很自由既可读又可写，既可以作为函数的返回值类型也可以作为函数形参类型，既可以声明成只读属性的类型又可以声明可变属性。但是注意了:不变型就是没有子类型化关系，所以它会有一个局限性就是如果以它作为函数形参类型，外部传入只能是和它相同的类型，因为它根本就不存在子类型化关系说法，那也就是没有任何类型值能够替换它，除了它自己本身的类型 例如MutableList<String>和MutableList<String?>是完全两种不一样的类型。

结论：

协变宗旨就是定义的泛型类内部不能存在写操作的行为，对于逆变根本宗旨一般都是只写的。

**协变点out和逆变点in的位置的规则是一般大体情况下要遵守的，但是需要具体情况具体分析，针对设计的泛型类具体情况，适当地在不违背根本宗旨以及满足需求情况下变下协变点和逆变点的位置规则**

关于这块，更详细请参考：

> [教你如何攻克Kotlin中泛型型变的难点(上篇)](https://blog.csdn.net/u013064109/article/details/83869716)
>
> [教你如何攻克Kotlin中泛型型变的难点(下篇)](https://blog.csdn.net/u013064109/article/details/84060626)
>
> [教你如何攻克Kotlin中泛型型变的难点(应用篇)](https://blog.csdn.net/u013064109/article/details/84504847)



##### 类型擦除

泛型信息只存在于代码编译阶段，在进入 JVM 之前，与泛型相关的信息会被擦除掉。

```kotlin
fun main() {
    val list1 = ArrayList<String>()
    
    println(list1.javaClass) //class java.util.ArrayList
}
```





##### reified

`reified`关键字是用于Kotlin内联函数的,修饰内联函数的泛型,泛型被修饰后,在方法体里,能从泛型拿到泛型的Class对象

reified与inline结合使用，在kotlin中一个内联函数（inline）可以被具体化（reified），这意味着我们可以得到使用泛型类型的Class。所以具体实际类型在调用地方是已经知道的；

比如定义拓展函数来启动Activity:

```kotlin
private fun <T : Activity> Activity.startActivity(context: Context, clazz: Class<T>) {
    startActivity(Intent(context, clazz))
}
//调用上面函数：
startActivity(context, NewActivity::class.java)
```

使用reified+inline后：

```kotlin
inline fun <reified T : Activity> Activity.startActivity(context: Context) {
    startActivity(Intent(context, T::class.java))
}
//调用上面函数：
startActivity<NewActivity>(context)
```

从上面，一般涉及传入Class类型时，可使用该关键字

具体实用例子：

```kotlin
data class User(val first: String, val last: String)

val json = "{"first":"Zhi", "last":"Yufen"}"
//一般使用方式：
val user: User = Gson().fromJson(getJson(), User.class)

//使用 Reifed
inline fun <reified T> Gson.fromJson(json: String) = fromJson(json, T::class.java)

//使用方式
val user: User = Gson().fromJson(json)//会自动推导T为User
//or
val user = Gson().fromJson<User>(json)
```

更多reified的用法，可参考

[Kotlin的独门秘籍Reified实化类型参数](https://youkmi.blog.csdn.net/article/details/83507076?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-83507076-blog-114261938.pc_relevant_multi_platform_whitelistv1_mlttest2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-83507076-blog-114261938.pc_relevant_multi_platform_whitelistv1_mlttest2&utm_relevant_index=1) 

[推荐使用 Kotlin 关键字 Reified](https://www.wandouip.com/t5i245905/) 