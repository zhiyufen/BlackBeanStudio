## 流程控制
Kotlin 提供了 if 和 when 两种分支语句，when 语句可以代替 java 的 switch 语句，并且功能更强大；
Kotlin 提供了while, do while, for-in 循环，抛弃了 Java 的原有的普通 for 循环。同样也提供了 break, continue 来控制程序的循环结构。

### 顺序结构

#### if 分支
与Java的一样；形式：
```
    if (expression) {
        statment...
    } else if (expression) {
        statment...
    } else if (expression) {
        statment...
    }  else {
        statment...
    }
```

#### if 表达式
Kotlin的 if 分支还可作为 表达式使用：

```
fun main(args: ArrayList<String>) {
    var age = 20
    var str = if (age > 20) "age 大于20" else if (age < 20) "age 小于20" else "age 等于20"
    print(str)
}
```

#### when 分支语句

Kotlin 的 when 语句代替 java 的 switch 语句；

比如：
```
fun main(args: ArrayList<String>) {
    var scroe = 'B'
    when(scroe) {
        'A' -> print("优秀")
        'B' -> print("良好")
        'C' -> print("合格")
        else -> print("不合格")
    }
}
```
分支里，有多选语句的话，必须可以加 {}， 单名可加，可不加；

when 分支的3个改进：

- when 分支可以分配多个值；
- when 分支后的值不要求是常量 ，可以是任意表达式；
- when 分支对条件表达式的类型没有任意要求， 只要when 的条件与某个分支的值通过 "==" 比较返回 true，程序就可以进入该分支的代码；

```
fun main(args: ArrayList<String>) {
    var scroe = 'B'
    var str = "EFGH"
    when(scroe) {
        str[0] - 4, str[1] - 4 -> {
            print("优秀")
        }
        str[2] -4, str[3] - 4 -> {
            print("良好")
        }
        else -> {
            print("不合格")
        }
    }
}
```

#### when 表达式
如果 when 分支被当成表达式， 那么符合条件的分支的代码块的值就是整个表达式的值；因此需要有一个返回值，因此when表达式通常必须有else分支；

```
fun main(args: ArrayList<String>) {
    var scroe = 'B'
    var result = when(scroe) {
        'A', 'B' -> {
            "优秀"
        }
        'C', 'D' -> {
            "良好"
        }
        else -> {
            "不合格"
        }
    }
    print(result)
}
```

#### when 分支处理范围

通过使用 in, !in 运算符，我们可以使用 when 分支检查表达式是否位于指定区间或集合中；

```
fun main(args: ArrayList<String>) {
    var scroe = 90
    var result = when(scroe) {
        in 90 .. 100 -> {
            "优秀"
        }
        in 60 .. 89 -> {
            "合格"
        }
        else -> {
            "不合格"
        }
    }
    print(result)
}
```

#### when 分支处理类型

通过使用 is, !is 运算符，我们可以使用 when 分支检查表达式是否位于指定区间的类型；

```
fun toDouble(value: Any) = when (value) {
    is Int -> value.toDouble()
    is String -> value.toDouble()
    is Double -> value
    else -> 0.0
}
```

#### when 条件分支

when 分支还可以用来取代 if..else if 链， 此时不需要为 when 分支提供任何条件表达式，每个分支条件都是一个布尔表达式,当指定分支的布尔表达式为 true时，执行该分支；

```
fun main(args: ArrayList<String>) {
    var ln = readLine()
    if (ln != null) {
        when {
            ln.matches(Regex("\\d+")) -> print("输入的全是数字")
            ln.matches(Regex("[a-zA-Z]+")) -> print("输入的全是字母")
            ln.matches(Regex("[a-zA-Z0-9]+")) -> print("输入的是字母,数字")
            else -> print("输入的是特殊字符")
        }
    }
}
```


### 循环结构

#### while 循环
#### do while 循环
与 Java的一样， 略过

#### for-in 循环

for-in循环专门用于遍历范围、序列和集合等包含的元素。

语法格式如下：
```
for (常量名 in 字符串|范围|集合) {
    statements...
}
```
需要说明的是：
- for-in 里的常量无须声明。
- for-in循环可用于遍历任何可迭代对象；
- 不允许在for-in循环中对循环计数器进行赋值；

```
for (num in 1 .. 10) {
    statements...
}
```

#### 嵌套循环

Kotlin 同样支持嵌套循环；

### 控制循环结构

#### 使用 break 结束循环

``` 
for (i in 2 .. 4) {
    if (i == 3) 
        break
}

```

另外 kotlin 可使用 break 后面跟@的标识符(用于标识某个循环层)， 该标签只有放在循环语句或when语句前才起作用；

```
fun main(args: ArrayList<String>) {
    myOuter@ for (i in 2 .. 9) {
        for(a in 3 .. 8) {
            print("${i} , ${a}\n")
            if (i  == 3 && a == 3){
                break@myOuter
            }
        }
    }
}
```

#### 使用 continue 忽略本次循环的剩下语句；

``` 
for (i in 2 .. 4) {
    if (i > 3) 
        continue
    print("${i} \n")
}

```

continue 后面也可以跟 '@' 标识符； 

```
fun main(args: ArrayList<String>) {
    myOuter@ for (i in 2 .. 9) {
        for(a in 3 .. 8) {
            print("${i} , ${a}\n")
            if (a == 4){
                continue@myOuter
            }
        }
    }
}
```

输出结果是：

```
2 , 3
2 , 4
```

当a == 4时， 可见myOuter@后面的外层循环也停止了。

```
fun testing() {
    for (i in 2 .. 4) {
      if (i == 3) 
          return
      print("${i} \n")
  }
} 

```
