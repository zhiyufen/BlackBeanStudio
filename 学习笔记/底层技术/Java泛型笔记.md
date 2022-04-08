## Java泛型笔记

[TOC]

### 概念

Java中的泛型在JavaSE5中引入，所谓泛型，即参数化类型。就是说，类型是以参数的方式传入泛型类；

泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

泛型设计意味着编写的代码可以被很多不同类型的对象所重用。

### 类型擦除

泛型只在编译阶段有效；在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。

```
泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。
```

虚拟机没有泛型类型对象，所有类型都属于普通类。当定义一个泛型类型时，都自动提供了一个相应的原始类型。原始类型就是删去类型参数后的泛型名。擦除类型变量，并替换为限定类型。
原始类型用第一个限定的类型变量来替换，如果没有给定限定就用Object替换。

```java
public class ResultSecond<T extends Comparable> {
    private T first;
    private T second;

    public ResultSecond(T first, T second) {
        if (first.compareTo(second) > 0) {
            this.first = first;
            this.second = second;
        } else {
            this.first = second;
            this.second = first;
        }
    }
}
```

上面的原始类型为：

```java
public class ResultSecond {
    private Comparable first;
    private Comparable second;

    public ResultSecond(Comparable first, Comparable second) {
        if (first.compareTo(second) > 0) {
            this.first = first;
            this.second = second;
        } else {
            this.first = second;
            this.second = first;
        }
    }
}
```

从上面得出几个点：

- 不能用基本类型实例化类型参数；
  因为类型擦除后，只有Object类型的类，而Object不能存储int的值；
- 运行时类型查询只适用于原始类型；
- 

### 类型边界

#### 普通泛型：

类型会擦除到Object

```java
class Normal<T>
{
    T item;
    public Normal(T item)
    {
        this.item = item;
    }
 
    void test()
    {
        //此处，item只能调用属于Obeject类的方法
    }
}
```

#### 上边界限定

实现类型的限定：

```java
class Animal{
    void talk(){}
}
 
class B<T extends Animal>
{
    T item;
    public A(T item)
    {
        this.item = item;
    }
 
    void test()
    {
        //此处，item可以调用属于Animal类的方法
        item.talk();
    }
}
```

上面例子类型会擦除到Animal，也就是可调用Animal类的方法；

也可以限定多个边界：

```java
interface Eatable {void eat();}
interface Drinkable {void drink();}
class B<T extends Dog & Cat>
{
    T item;
    void test()
    {
        //此处，item可以调用属于Eatable和Drinkable的方法
        //不过，前提是，item同时是Eatable和Drinkable
        //（这里只能通过多个接口实现的方式，或者一个基类和多个接口的方式，因为你无法使一个类继承自多个父类）
    }
```

限定类型用“&”分隔，而逗号用来分隔类型变量。注意点：Java的继承中，可以根据需要拥有多个接口超类型，但限定至多有一个类。如果一个类作为限定，它必须是限定列表中的第一个。

### 通配符

通配符，作用就是匹配多种类型； 一般使用是泛型实例化时，进行定义的；

#### 上界通配符:  < ? extends B >

也就是说可以匹配类型A及其所有子类，即类型的上界是B，无下界。

有几个这样类：

```java
    class A
    {
        public void print()
        {
            System.out.println("A");
        }
    }
    class B extends A
    {
        @Override
        public void print()
        {
            System.out.println("B");
        }
    }
    class C extends B
    {
        @Override
        public void print()
        {
            System.out.println("C");
        }
    }
```

正常来说，C是B的子类，而存放C的数组是可以认为是B的数组，但实际上使用是不行的：

```java
ArrayList<B> myArrayList1 = new ArrayList<C>(); //Error
```

这时就要使用到上界通配符：

```java
ArrayList<? extends B> myArrayList2 = new ArrayList<C>(); //OK

ArrayList<C> myArrayList3 = new ArrayList<C>(); //OK
ArrayList<B> myArrayList4 = new ArrayList<C>(myArrayList3); //Erro
ArrayList<? extends B> myArrayList5 = new ArrayList<>(myArrayList3); //OK
```

上界通配符的副作用： 不能往里存，只能往外取；

```java
ArrayList<? extends B> myArrayList2 = new ArrayList<C>(); //OK
myArrayList2.add(new B()) // Error
myArrayList2.add(new C()) // Error
myArrayList2.get(0).print();//OK
```

ArrayList<? extends B>在原始类型上 表示存放的元素可能为B和B的任何子类，那么为了确保类型安全，只有继承B和所有B的子类的类型才能Add，而实际上这个类型是不可能 存在的。

#### 下界通配符 ? super B

可以匹配类型B及其所有基类，即类型的下界是B，上界是Object

```java
 ArrayList<? super B> myArrayList = new ArrayList<A>(); //OK
 ArrayList<? super B> myArrayList2 = new ArrayList<B>(); //OK
 ArrayList<? super B> myArrayList3 = new ArrayList<C>(); //ERROR
 ArrayList<? super B> myArrayList4 = new ArrayList<Object>(); //OK
```

ArrayList<? super B>在原始类型上表示存放的元素确保为B及B其所有基类， 也就是B和B子类都可以，所以我们能操作add方法

```java
ArrayList<? super B> myArrayList = new ArrayList<A>(); //OK
myArrayList.add(new A()); // Error
myArrayList.add(new C());
myArrayList.add(new B());
Object object = myArrayList.get(0); // 取出来为Object类型，需要开发者自己判断后再转类型；
```

下界通配符的副作用： 不影响往里存，但往外取只能放在Object对象里；

### \<T> VS  <?>

- \<T>：泛型标识符，用于泛型定义（类、接口、方法等）时，可以想象成形参。
- <?>：通配符，用于泛型实例化时，可以想象成实参。

通配符\<?>和类型参数\<T>的区别就在于，对编译器来说 所有的T都代表同一种类型，但通配符\<?>没有这种约束， 所以编译器无法知道放进来的对象是哪一种，所以什么都放不进去；

#### PECS（Producer Extends Consumer Super）原则

1. 频繁往外读取内容的，适合用上界Extends。
2. 经常往里插入的，适合用下界Super。



本文笔记于：

> https://blog.csdn.net/StruggleShu/article/details/73384796
> https://www.cnblogs.com/xinxinBlog/p/10264653.html
> https://www.cnblogs.com/coprince/p/8603492.html