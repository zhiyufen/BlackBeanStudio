## [kotlin] 异常处理

与其它语言的异常处理基本上相同的， 都是 try{} catch..，但不同点在于该语句可以作为 表达式处理,；

```kotlin
val result = try {
    Api.call() //返回一个值
} catch (e : Exception) {
    null
} finally {
    
}
```



其它使用方法例子：

```kotlin
val s = person.name ? : throw IllegalArgumentException("Name required") //名字为空，则抛出异常
```

