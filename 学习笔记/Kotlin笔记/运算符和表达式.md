## 运算符和表达式
Kotlin 基本支持 JAVA 的全部运算符，但 Kotlin 不支持三目运算符(可使用if表达式代替)


### 与 Java 相同的运算符
Kotlin 的运算符都是以方法形式来实现的，这些运算符都具有特定的符号和固定的优化级。 

#### 单目运行算符
单目前缀运行算符有 +， -， ！这三个。

| 运算符 | 对应的方法 |
| ------ | -------------- |
| +a     | a.unaryPlus()  |
| -a     | a.unaryMinus() |
| !a     | a.not()        |

#### 自加和自减

| 运算符 | 对应的方法 |
| ------ | ---------- |
| a++    | a.inc()    |
| a--    | a.dec()    |

但++， -- 放在变量前面是有区别的， 因此对应的inc(), dec() 方法还不完全等同的，只符合放在后面的情况；

对于一些对象来说， 它也可以实现 inc(), dec() 方法，从而支持 object++/object-- 的操作;

#### 双目运算符

| 运算符 | 对应的方法 |
| ------ | ------------ |
| a + b  | a.plus(b)    |
| a - b  | a.minus(b)   |
| a * b  | a.times(b)   |
| a / b  | a.div(b)     |
| a % b  | a.rem(b)     |
| a..b   | a.rangeTo(b) |

同样其它类也可以实现这些方法，从而对这些实现也可使用这些运算符；

#### in 和 !in 运算符
| 运算符 | 对应的方法 |
| ------ | ---------- |
| a in b    | b.contains(b)    |
| a !in b    | !b.minus(a)    |


同样其它类也可以实现这些方法，从而对这些实现也可使用这些运算符；


#### 索引访问运算符

| 运算符          | 对应的方法       |
| ------------------ | --------------------- |
| a[i]               | a.get(i)              |
| a[i,j]             | a.get(i, j)           |
| a[i_1,...,i_n]     | a.get(i_1,...,i_n)    |
| a[i] = b           | a.set(i, b)           |
| a[i,j] = b         | a.set(i,j,b)          |
| a[i_1,...,i_n] = b | a.set(i_1,...,i_n, b) |

同样其它类也可以实现这些方法，从而对这些实现也可使用这些运算符；
比如Kotlin的String及Java的ArrayList都提供get(index) 和 set(index, val)的方法，因此他们都可以使用这些运算符；

#### 调用运算符

| 运算符 | 对应的方法 |
| ------ | -------------- |
| a()     | a.invoke()  |
| a(b)     | a.invoke(b) |
| a(b1, b2)     | a.invoke(b1, b2)        |
| a(b1, b2,b3...)     | a.invoke(b1, b2,b3...)        |

这个运算符是针对我们通过反射机制获取Method对象时进行调用其方法；

```
fun main(args: ArrayList<String>) {
    val s = "java.lang.String"
    //使用反射获取String类的length方法
    val mtd = Class.forName(s).getMethos("length")
    
    //两个调用方法相同
    print(mtd("java"))
    print(mtd.invoke("java"))
}
```

#### 广义赋值运算符

| 运算符 | 对应的方法  |
| ------ | ---------------- |
| a += b | a.plusAssign(b)  |
| a -= b | a.minusAssign(b) |
| a *= b | a.timesAssign(b) |
| a /= b | a.divAssign(b)   |
| a %= b | a.remAssign(b)   |

这种运算符有些特殊， 比如 a += b，实际上相当于 a = a + b, 因此在程序运算过程，往往不需要a有 plusAssign() 方法；

对于这种广义赋值操作，例如 a += b： 
1， 判断 plusAssign() 方法是否存在？ 
   存在  ----> 1) 如果 plus() 也存在，将会报告错误 (调用的目标方法不确定)；
               2) 确保 plusAssign() 没有返回值， 否则报告错误；
               3) 如果能通过前两步的检查，则转换成执行 a.plusAssign(b)；
   不存在 ---> 那么将转换成 a = a + b; 
   

#### 相等与不等运算符

| 运算符 | 对应的方法               |
| ------ | ----------------------------- |
| a == b | a?.equal(b) ?:(b === null)    |
| a != b | !(a?.equal(b) ?:(b === null)) |

从这里看， Kotlin的 “==” 不再比较两个变量是否引用同一个对象。而是与equal()比较基本上是等义的， 只不过 "==" 比较是空指针安全的。 

Java 提供的 "==" 和 "!=" 运算符，在Kotlin中， 则由 "===" 和 "!==" 代替；


#### 比较运算符

| 运算符 | 对应的方法     |
| ------ | ------------------- |
| a > b  | a.compareTo(b) > 0  |
| a < b  | a.compareTo(b) < 0  |
| a >= b | a.compareTo(b) >= 0 |
| a <= b | a.compareTo(b) <= 0 |

从上面看， 比较运算符其实是由 compareTo() 方法来实现的， 而这个这个方法是在 Comparable 接口中定义的方法；
因此原来Java中支持使用 compareTo()方法比较大小的对象，都可以使用比较运算符来计算；比如Kotlint的 String, Java中的 Date; 



### 位运算符

Kotlin 不再像 java 以特殊符给出运算符的，而是 infix 函数的形式给出；

- add(bits): 按位与，对应java的 &
- or(bits): 按位或，对应java的 |
- inv(bits): 按位非，对应java的 ~
- xor(bits): 按位异或，对应java的 ^ 
- shl(bits): 左移运算符，对应java的 << 
- shr(bits): 右移运算符，对应java的 >>
- ushr: 无符号右移运算符，对应java的 >>>

```
fun main(args: ArrayList<String>) {
    print(5 and 9) //输出1
    print(5 or 9)  //输出13
    print(5 shl 2)  //输出20
}
```

Kotlin的位运算符只能对 Int 和 Long 的两种数据类型起作用。

另外移位还需要新遵循如下的规则: 当 a shr b 时， 
- a 是 Int 类型， b > 32时， 需要先用 b 对32求余，得到的结果还是真正移位数；
- a 是 Long 类型的，b > 64, 则用b 对64求余， 得到的结果还是真正移位数；

### 区间运算符

Kotling 提供闭区间运算符和半开区间运算符，它可以非常方便构建一种数据结构，这种数据结构可包含特定区间的所有值；

#### 闭区间运算符

闭区间运算符 a .. b 用来定义一个从a~b（包括a,b边界值）的所有值区间，并且 a 不能大于 b; 

```
fun main(args: ArrayList<String>) {
    var range = 2 .. 9
    for( r in range) {
        print("${r} * 5 = ${r *5}\n")
    }
}
```

#### 半开区间运算符

使用 a until b 来定义一个从从a~b（包括a边界值，但不包括 b的边界值）的所有值区间，并且 a 不能大于 b; 

如果a与b相同，那么该半开区间就是一个空区间；

可利用半开区间遍历数组等列表时，非常方便：
```
fun main(args: ArrayList<String>) {
    var books = arrayOf("java", "Kotlin", "C", "C++")
    
    for (index in 0 until books.size){
        var book = books[index]
    }
}
```

#### 反向区间

有时候，我们希望区间是从大到小的， 那么就可以使用 downTo 运算符了；

```
fun main(args: ArrayList<String>) {
    var range = 9 downTo 2
    for( r in range) {
        print("${r} * 5 = ${r *5}\n")
    }
}
```

输出结果为：

```
9 * 5 = 45
8 * 5 = 40
7 * 5 = 35
6 * 5 = 30
5 * 5 = 25
4 * 5 = 20
3 * 5 = 15
2 * 5 = 10
```

#### 区间步长

前面的区间默认是步长是1；如果需要调整的话， 就要使用 step 运算符了；比如

```
fun main(args: ArrayList<String>) {
    var range = 9 downTo 2 step 2
    for( r in range) {
        print("${r} * 5 = ${r *5}\n")
    }
}
```

输出结果为：
```
9 * 5 = 45
7 * 5 = 35
5 * 5 = 25
3 * 5 = 15
```

### 运算符重载

在我们类的， 只需要实现各运算符号对应的方法，那么就可以使用相应的运算符了；这里不写栗子了^^。
