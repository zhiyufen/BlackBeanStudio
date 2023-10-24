## [Android笔记] 单元测试学习

由于之前看过这方面，因此该笔记不会作概念性的记录，更多的相关方法和使用的笔记。

### 1. Android 单元 测试

一般分下面两种；

| Junit test                                                   | androidTest                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Java环境下的测试: 纯Java代码的测试, 只运行在本地电脑的JVM环境上, 不依赖于Android框架的任何api, 因此执行速度快，效率较高，但是无法测试Android相关的代码 | Android环境下的测试, 需要运行在真机设备或模拟器上，运行速度较慢，但是可以测试UI的交互以及对设备信息的访问，得到接近真实的测试结果。 |

一般创建Android app时， build.gradle 会默认添加下面测试依赖配置:

```groovy
    //Java test
	testImplementation 'junit:junit:4.12'
	//Android test
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
```



#### 1.1 Junit 相关方法和注解

Assert类中**主要断言方法**如下：

| 方法名            | 方法描述                               |
| ----------------- | -------------------------------------- |
| assertEquals      | 断言传入的预期值与实际值是相等的       |
| assertNotEquals   | 断言传入的预期值与实际值是不相等的     |
| assertArrayEquals | 断言传入的预期数组与实际数组是相等的   |
| assertNull        | 断言传入的对象是为空                   |
| assertNotNull     | 断言传入的对象是不为空                 |
| assertTrue        | 断言条件为真                           |
| assertFalse       | 断言条件为假                           |
| assertSame        | 断言两个对象引用同一个对象，相当于“==” |
| assertNotSame     | 断言两个对象引用不同的对象，相当于“!=” |
| assertThat        | 断言实际值是否满足指定的条件           |

注意：上面的每一个方法，都有对应的重载方法，可以在前面加一个String类型的参数，表示如果断言失败时的提示。



相关**Junit注解**如下：

| 注解名                             | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| @Test                              | 表示此方法为测试方法且为 public void                         |
| @Before                            | 在每个测试方法前执行，可做初始化操作。 <br />注： 每个@Test方法的执行都会触发一次调用 |
| @After                             | 在每个测试方法后执行，可做释放资源操作。<br />注：每个@Test方法的执行完都会触发一次调用。 |
| @Ignore                            | 忽略的测试方法                                               |
| @BeforeClass                       | 在类中所有方法前运行。此注解修饰的方法必须是public static void；用于做一些全局可用的初始化工作(如: 连接数据库) |
| @AfterClass                        | 在类中最后运行。此注解修饰的方法必须是public static void；用于清理数据(如: 断开数据连接) |
| @RunWith                           | 指定该测试类使用某个测试运行器                               |
| @Parameters                        | 指定测试类的测试数据集合                                     |
| @Rule                              | 重新制定测试类中方法的行为                                   |
| @FixMethodOrder                    | 指定测试类中方法的执行顺序；分别是DEFAULT、JVM、NAME_ASCENDING（字母顺序） |
| @Test (expected = Exception.class) | 该测试方法没有抛出Annotation中的Exception类型(子类也可以)，则测试失败 |
| @Test(timeout=100)                 | 如果该测试方法耗时超过100毫秒，则测试失败，用于性能测试      |

其中执行顺序：@BeforeClass –> @Before –> @Test –> @After -> @Before –> @Test –> @After -> ....–> @AfterClass

#### 1.2 相关使用

创建相关类的测试文件： 在要测试的类中，**右击鼠标/Go To/Test/Create New Test...**一步步创建即可；

##### 参数化测试

使用Parameterized可以直接连续用不同的值来测试一个方法；

首先在测试类上添加注解`@RunWith(Parameterized.class)`，在创建一个由 `@Parameters` 注解的public static方法，让返回一个对应的测试数据集合。最后创建构造方法，方法的参数顺序和类型与测试数据集合一一对应。

```kotlin
@RunWith(Parameterized::class)
class TimeUtilsTest(var time: Long) {

    companion object {
        @Parameterized.Parameters
        @JvmStatic
        fun getTimeStamp() : Collection<Long> {
            return arrayListOf(System.currentTimeMillis(), 1573894167000L, 1583893425000L)
        }
    }


    @Test
    fun getDateString() {
        assertEquals("2019-11-16", TimeUtils.getDateString(time))
    }
}
```

##### assertThat用法

使用assertThat可以设置在失败时输出相应的信息， 

```java
public static <T> void assertThat(String reason, T actual,
            Matcher<? super T> matcher)
```

其中`reason`为断言失败时的输出信息，`actual`为断言的值，`matcher`为断言的匹配器

常用的匹配器整理：

| 匹配器               | 说明                               | 例子                                              |
| -------------------- | ---------------------------------- | ------------------------------------------------- |
| is                   | 断言参数等于后面给出的匹配表达式   | assertThat(5, is (5));                            |
| not                  | 断言参数不等于后面给出的匹配表达式 | assertThat(5, not(6));                            |
| equalTo              | 断言参数相等                       | assertThat(30, equalTo(30));                      |
| equalToIgnoringCase  | 断言字符串相等忽略大小写           | assertThat(“Ab”, equalToIgnoringCase(“ab”));      |
| containsString       | 断言字符串包含某字符串             | assertThat(“abc”, containsString(“bc”));          |
| startsWith           | 断言字符串以某字符串开始           | assertThat(“abc”, startsWith(“a”));               |
| endsWith             | 断言字符串以某字符串结束           | assertThat(“abc”, endsWith(“c”));                 |
| nullValue            | 断言参数的值为null                 | assertThat(null, nullValue());                    |
| notNullValue         | 断言参数的值不为null               | assertThat(“abc”, notNullValue());                |
| greaterThan          | 断言参数大于                       | assertThat(4, greaterThan(3));                    |
| lessThan             | 断言参数小于                       | assertThat(4, lessThan(6));                       |
| greaterThanOrEqualTo | 断言参数大于等于                   | assertThat(4, greaterThanOrEqualTo(3));           |
| lessThanOrEqualTo    | 断言参数小于等于                   | assertThat(4, lessThanOrEqualTo(6));              |
| closeTo              | 断言浮点型数在某一范围内           | assertThat(4.0, closeTo(2.6, 4.3));               |
| allOf                | 断言符合所有条件，相当于&&         | assertThat(4,allOf(greaterThan(3), lessThan(6))); |
| anyOf                | 断言符合某一条件，相当于或         | assertThat(4,anyOf(greaterThan(9), lessThan(6))); |
| hasKey               | 断言Map集合含有此键                | assertThat(map, hasKey(“key”));                   |
| hasValue             | 断言Map集合含有此值                | assertThat(map, hasValue(value));                 |
| hasItem              | 断言迭代对象含有此元素             | assertThat(list, hasItem(element));               |

可实现继承`BaseMatcher`抽象类的自定义匹配器；



相关文章：

[测试 之Java单元测试、Android单元测试](https://blog.csdn.net/junhuahouse/article/details/80780216)

[Android单元测试](https://weilu.blog.csdn.net/category_9270906.html)

[Android单元测试学习总结](https://blog.csdn.net/lyabc123456/article/details/89363721)



