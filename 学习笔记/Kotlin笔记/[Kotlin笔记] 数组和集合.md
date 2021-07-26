## 数组和集合

Kotlin 为数组增加了一个 Array 类，为元素是基本类型的数组数组增加了 XxxArray 类（其中 Xxx 可以是 Byte, Shot, Int, Long等基本类型）；

Kotlin集合体系抛弃了 Java 集合体系中的 Queue 集合，但增加了可变集合和不可变集合的概念；Kotlin 集合体系由 List, Set, Map 三种集合组成：

- List 代表有序、集合元素可重复的集合；
- Set 代表无序、集合元素不可重复的集合；
- Map 则采用 key-value 对形式的存储数据；

### 数组

Kotlin 的数组使用 Array<T> 类代表，因此数组就是一个 Array 类的实例；

#### 创建数组

Kotlin 创建数组大致有如下两种方式:

- 使用 arrayOf(), arrayOfNulls(), emptyArray() 工具函数；
- 使用 Array(size: Int, init: (Int) -> T) 构造器；

```
fun main(args:Array<String>) {
    //创建包含指定元素的数组
    var arr1 = arrayOf("Java", "Kotlin", "C++")
    var intArr1 = arrayOf(2, 4, 5)

    //创建指定长度，元素为 null, 的数组
    var arr2 = arrayOfNulls<String>(5)
    var intArr2 = arrayOfNulls<Int>(5)

    //创建长度为0的空数组
    var arr3 = emptyArray<String>()
    var intArr3 = emptyArray<Int>()

    //创建指定长度、使用 Lambda 表达式初始化数组元素的数组
    var arr4 = Array(5, { (it * 2 + 97).toChar()} )
    var intArr4 = Array(6, {"Hello"})
}
```

对于8种基本类型： Int, Short 等， 有专门提供的 XxxArray，因此可以这么创建其级数：

```
 var intArr = intArrayOf(2,4, 9,10)
 var intArr2 = IntArray(6, {it * it})
```
