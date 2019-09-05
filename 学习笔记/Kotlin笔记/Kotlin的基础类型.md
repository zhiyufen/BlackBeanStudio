## Kotlin 的基础类型 

### 注释
 
可使用单行注释 // 和多行注释 /* */， 另外多行注释支持嵌套注释；

```java
/*
我是注释1
我是注释2
*/
fun testing() {
    //单行注释
    /**
     * 我是中国人
     * /* 我是广东人 */
     * 
     * */
    print("Hello World!")
}
```

Kotlin可通过使用 Dokka 工具来生成 API 文档，可自行到 ![Dokka](https://github.com/Kotlin/dokka/releases/download/0.9.10/dokka-fatjar.jar) 下载该工具，下载后得到一个dokka-fatjar.jar文件；

Dokka 工具的基础语法：
```xml
    java -jar dokka-fatjar.jar <source directories> <arguments>
```
关于 Dokka 更详细及其用法，请前往 Dokka 的github地址查看： https://github.com/Kotlin/dokka


### 变量

#### 关键字

可分成3类：

硬关键字： 这些关键字无论在什么情况下都不可以用作标识符： as, as?, break, class, continue, do, else, false, for, fun, if, in, !in, is, !is, null, object, package, return, super, this, throw, true, try, typealise, val , var ,when, while; 

软关键字： 这些关键字可以在它们不起作用的上下文中用作标识符：by, catch, constructor, delegate, field, file, init, finally, get, import, init, param, property; 

修饰符关键字： 这些关键字也可以在代码中用作标识符： abstract, annotation, companion, const, crossinline, data, enum, external, final, infix, inline, inner, internal, lateinit, noinline, open, override, private, protected, public, reified, sealed, suspend, tailrec, vararg; 

注： 标识符就是变量的命名

#### 声明变量

Kotlin 是强类型语言，所有变量必须先声明，后使用；

使用 var(可多次被赋值) 或 val(值不变，只能被赋值一次) ：

```
    var|val 变量名 [:类型] [= 初始值]
```

比如：
```
var b : Int
var name = "Hello World!"

var bookId = 25

//错误，定义的类型与赋值不一致
var book : String = 500 

val book = "Hello"
//错误， val不可以被重复赋值
book = "World!"
```

### 整型

有四种：
1，Byte: 占8位，兼容 Java 的 byte 和 Byte 类型；

2，Short: 占16位，兼容 Java 的 short 和 Short 类型；

3，Int: 占32位， 兼容 Java 的 int, Integer 类型；

4，Long: 占64位，兼容 Java 的 long, Long 类型；

Kotlin 的整型不是基本类型，而是引用类型，四大整型都继承于 Number 类型， 因此它们可调用方法，访问属性；比如可调用 MIN_VALUE, MAX_VALUE 属性来获取其的最小值和最大值；
```
 Int.MIN_VALUE
 Long.MAX_VALUE
```

Kotlin 整数支持3种进制的表示方式： 十进制， 二进制(以 0b 或 0B 开头的整数数值)， 十六进制(以 0x 或 0X 开头的整数数值)；


### 浮点型

有两种：

1，Float: 表示32位的浮点型；

2，Double: 表示64位的双精度浮点型；

有两种表示方式：十进制数形式(5.12, 512.0, 0.512) 及 科学计数形式(5.12e2或5.12E2(即 5.12 * 10^2))

### 字符型

字符型通常用于表示单个的字符，字符型值必须使用单引号(')括起来；Char

字符型有3种表示形式：

1， 直接通过单个字符来指定字符型值： 'A', '9'等；

2， 通过转义字符表示特殊字符型值： '\n', '\t'等；

3， 直接使用 Unicode 值来表示字符型值，格式是 '\uXXXX', 其中XXXX代表一个十六进制的整数；

相关常用转义字符： \b, \n, \r, \t, \", \', \\

注： Kotlin的 Char 型变量不能当成整数值使用，Char的值也不能赋值给整型变量；

```
    val a :Char = 'a'
    
    var book : Char = '植'

```

### 数值型之间的类型转换

#### 整形之间的转换

Kotlin为所有数值类型都提供如下的方法进行转换：toByte(), toShort(), toInt(), toLong(), toChar(), toFloat(), toDouble(); 

Kotlin 要求不同整型的变量或值之间必须进行显式转换；另外需要注意一个取值范围大的转取值范围小的类型时，会有可能造成数据溢出；

虽然Kotlin不支持隐式转换，但在表达式中可以自动转换，

```
    var bookPrice :Byte = 79
    var itemPrice : Short = 120; 
    
    //两个都会自动提升到Int 类型，最后total的结果是 Int类型的。
    var total = bookPrice + itemPrice

    //bookPrice强制转换成Long，这样整个表达式最高等级的操作数是Long类型，因此最后的结果也是Long类型的
    val tot  = bookPrice.toLong() + itemPrice.toByte();
```

虽然 Char  不能当成整数进行算术计数，但Char 提供了加，减运算支持： 

1，Char 型值加，减一个整形值，最后结果是Char 型， 比如： 'a' + 1  ==> 'b'; 

2, 两个Char 型值相等相减(不能相加)，最后返回一个 Int型；

看起来，与JAVA的运算没有多大区别；

#### 浮点与整型之间的转换

与整型之间的转换一样；

##### 表达式类型的自动提升

当一个算术表达式中包含多个数值型的值时， 整个算术表达式的数据类型发生自动提升，Kotlin 定义了与 java 基本相似的自动提升规则：

1，所有的 Byte, Short 类型将被提升到 Int 类型；

2，整个算术表达式的数据类型自动提升到与表达式中最高等级操作数同样的类型；操作数等级排列如下(右边为最高)：
```
    Byte --> Short --> Int --> Long  --> Float --> Double
```
    
注： 当把加号(+) 放在字符串和数值之间时， 这个加号是一个字符串连接运算符，而不是加法运算；

### Boolean 类型
Boolean 类型的值只能是 true, false,不能用 0 或者非 0 来代表，其它数据类型的值也不能转换成 Boolean 类型；

Boolean类型的变量不能接受 null 值，Boolean? 类型的变量才能接受 null 的值；Boolean 类型将直接映射为 Java 的 boolean基本类型， 
但 Boolean? 类型将映射成 Java 的 boolean 的包装类： Boolean；


### null 安全

#### 非空类型和可空类型

Byte, Short, Int, Long 等类型变量不能接受 null 值， 叫做非空类型；在已有数据类型后添加 '?'， 添加后相当于对原有的类型进行扩展， 可支持被赋值 null 的值， 为可空类型；

Kotlin 对可空类型进行了限制， 如果不加任何处理，可空类型不允许直接调用方法或访问属性；因此需要对可空类型要先判断后使用；

```
var a : String? = "hello"

var len = if (a != null) a.length else -1

a = null
if (a != null && a.length > 0) {
    print(a.length)
} else {
    print("空字符串“)
}
```

#### 安全调用

安全调用的用法：
```
   变量名?.[属性]|[函数]|[方法]
```

当变量为空时， 会直接返回 null, 不会调用后面的属性或方法, 这安全调用是支持链式调用： user?.dog?.name

安全调用还可以与 let 全局函数结合使用；

#### Elvis 运算
Elvis 运算其实就是 if else 的简化写法。比如：

```java
    var b: String? = "Hello"

    //正常写法
    var len1 = if (b != null) b.length else -1

    //Elvis 写法
    var len2 = b?.length ?: -1
```

"?:" 运算符就是Elvis ---- 它的含义是， 如果 ”?:“ 左边的表达式不为null时， 则返回左边表达式的值，否则返回 ”?:“ 右边表达式的值；

#### 强制调用

强制调用是”NPE“ 爱好者使用的，就是不管变量是不是 null, 都直接调用该变量的属性或方法；

```java
    var b: String? = "Hello"
    print(b!!.length)
```


### 字符串

Kotlin 使用 String 代表字符串， 字符串表示一个有序的字符集合。

#### 字符串类型

String 允许通过形如 s[i] 的格式来访问字符串指定索引处的字符，也可以通过 for 循环来遍历字符串中的每一个字符。

```java
fun main(args: Array<String>) {
    var str: String = "Hello"
    println(str[2])
    
    for (c in str) {
        println(c)
    }
}

```

Kotlin 字符串有两种字面值（Literal）：

1，转义字符串： 转义字符串可以有转义字符，该很像 Java 的字符串；

2，原始字符串： 原始字符串可以包含换行和任意文本。但原始字符串需要用3个引号引起来；
```java
fun main(args: Array<String>) {
    //普通字符串
    var str: String = "Hello"
    
    //定义原始字符串
    val txt = """
        |鹅,鹅,鹅,
        |曲项向天歌,
        |白毛浮绿水,
        |红掌拨清波
        """.trimMargin()
     print(txt + '\n')
}
```
由于代码编码格式， 在原始字符串中进行一些缩进，此时可使用 trimMargin() 方法 来去掉原始字符串前面的缩进；默认情况下， Kotlin 使用'|' 作为边界符， 开发者也可以使用其它符号作为边界符，但在需要在trimMargin() 方法中传入自定义的边界符；

##### 字符串模板

Kotlin 允许在字符串中嵌入变量或表达式，只要将变量或表达式放入 ${} 中即可；

``` 
fun main(args: Array<String>) {
    //普通字符串
    var str: String = "Hello"
    
    val s1 = " ${str} World!"
    
    //定义原始字符串
    val txt = """
        | ${s1}
        |鹅,鹅,鹅,
        |曲项向天歌,
        |白毛浮绿水,
        |红掌拨清波
        """.trimMargin()
     print(txt + '\n')
}
```

打印出来就是：
```
Hello World!
鹅,鹅,鹅,
曲项向天歌,
白毛浮绿水,
红掌拨清波
```

##### String 的方法

Kotlin的 String 与 Java 的不是同一个类，功能差不多；

1， 提供一系列的 toXXX() 方法将字符串转换成数值；

2， 提供将字符串首字母大写(capitalize())，小写(decapitalize())的方法， 以及 返回两个字符串相同的前缀，后缀的方法: str.commonPrefixWith/commonSufffixWith("XXX"), 

3， 其contains()方法 支持正则表达式(Java的不支持)；

更多 String 详细方法，请参考官网： https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/index.html


### 类型别名

类似于 C 语言中的 typedef 的功能： 可以为已有的类型指定另一个可读性更强的名字。 Kotlin 提供了 typealias 来定义类型别名的；

语法格式如下：
```
 typealias 类型别名 = 已有类型
```

一般类型名太长，我们就可以使用较短的别名来替代原类型名, 比如用于缩短集合类型就很有用：
```
typealias NodeSet = Set<Network.Node>
typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

另外也可以给内部类定义别名 和 Kotlin的 Lambda表达式定义别名；


