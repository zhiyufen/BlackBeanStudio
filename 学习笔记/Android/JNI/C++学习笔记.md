### C++学习笔记

[TOC]

#### 零碎的知识点

##### 引用

引用引入了对象的一个同义词。定义引用的表示方法与定义指针相似，只是用&代替了*。引用（reference）是c++对c语言的重要扩充。引用就是某一变量（目标）的一个别名，对引用的操作与对变量直接操作完全一样。

　　其格式为：类型 &引用变量名 = 已定义过的变量名。

普通引用在声明时必须用其它的变量进行初始化，引用作为函数参数声明时不进行初始化。

C++编译器在编译过程中使用常指针作为引用的内部实现，因此引用所占用的空间大小与指针相同。

###### 一般用法：

1. 用于形参；如果形参为引用类型，则形参是实参的别名。这样不需要复制值传递，性能更好，另外可通过其引用更改其内值；
2. 用于函数的返回值；当我们希望修改某个函数的返回值时，通常我们会返回这个值的引用（因为函数返回值其实是返回那个值得一份拷贝而已，所以想要修改必须使用引用）：

##### 对象的大小计算

1. 对象的大小与结构体计算方式一致 
2. static 静态变量和方法没有计算到类的大小里面
3. C++类对象计算需要考虑成员变量大小，内存对齐、是否有虚函数（因为虚函数表指针带来的影响）、是否有虚继承（因为虚基表指针带来的影响）等。
4. 类对象大小没有因为增加了静态成员而变化。因为静态成员是属于类成员共有的，不单独属于任何一个对象，对静态成员的存储不会选择在某个对象空间，而是存在于堆当中，因此不会对对象的大小造成影响。
5. 空类(不带任何数据，即类中没有非静态(non-static)数据成员变量，没有虚函数(virtual function)，也没有虚基类(virtual base class)。)的大小是一个特殊情况,空类的大小为1; 

对于有虚函数的对象，多了一个指向虚函数表的指针， 而指针的sizeof是8，因此含有虚函数的类或实例最后的sizeof是实际的数据成员的sizeof加8。

更多详细可参考：[c++类的大小计算](https://blog.csdn.net/fengxinlinux/article/details/72836199)

##### Const:修饰类的函数

void test() const{}
主要是来修饰this指针不能被修改

##### Typedef: 类型替换

```c++
typedef unsigned int u_int;   

u_int tt = 266;
```

##### 迁移c的代码到C++

需要定义下面的宏来包含C的代码：

```c++
#ifdef __cplusplus
extern "C"{
#endif
void func(int x,int y);
#ifdef __cplusplus    
}
#endif
//__cplusplus 是由c++编译器定义的宏，用于表示当前处于c++环境

```

**原因**： C++语言在编译的时候为了解决函数的多态问题，会将函数名和参数联合起来生成一个中间的函数名称，而C语言则不会，因此会造成链接时找不到对应函数的情况， 因此要告诉编译器C的代码要用C语言来编译；

##### 类的四大函数

**类的构造函数**： 类的构造函数是类的一种特殊的成员函数，它会在每次创建类的新对象时执行。构造函数的名称与类的名称是完全相同的，并且不会返回任何类型，也不会返回 void。构造函数可用于为某些成员变量设置初始值。构造函数一般是被c++编译器自动调用，也可以被手动调用。

二个特殊的构造函数：

1. 默认无参构造函数： 当类中没有定义构造函数时，编译器默认提供一个无参构造函数，并且其函数体为空；
2. 默认拷贝构造函数(对象)
3. 当类中没有定义拷贝构造函数时，编译器默认提供一个默认拷贝构造函数，简单的进行成员变量的值复制。

**类的析构函数**：C++中的类可以定义一个特殊的成员函数清理对象，这个特殊的成员函数叫做析构函数。析构函数没有参数也没有任何返回 类型的声明。析构函数在对象销毁时自动被调用。注意这个调用的时机。语法：～classname；

析构函数调用时机：

1. 栈对象离开其作用域时；
2. 堆对象被手动 delete时；

**拷贝构造函数**：拷贝构造函数是一种特殊的构造函数，它在创建对象时，是使用同一类中之前创建的对象来初始化新创建的对象.

拷贝构造函数通常用于：

1. 通过使用另一个同类型的对象来初始化新创建的对象。Teacher t = t2; 
2. 复制对象把它作为参数传递给函数;
3. 复制对象，并从函数返回这个对象;

注： 如果在类中没有定义拷贝构造函数，编译器会自行定义一个(浅拷贝)。如果类带有指针变量，并有动态内存分配，则它必须有一个拷贝构造函数，也就是据说的深拷贝。

拷贝构造函数什么时候会被调用:

- 用 = 定义变量时
- 将一个对象作为实参传递给一个非引用类型的形参
- 从一个返回类型为非引用类型的函数返回一个对象
- 用花括号列表初始化一个数组中的元素或一个聚合类中的成员
- 某些类类型还会对它们所分配的对象使用拷贝初始化。例如，当我们初始化标准库容器或是调用insert或push成员，容器会对其元素进行拷贝初始化。与之相反，用emplace成员创建的元素都进行直接初始化。

例子：

```c++
#include <iostream>
#include <string.h>

using namespace std;

class Student
{
public:
    Student(int age,char *name)
    {
        printf("构造函数执行\n");
        //this->name = name;
        this->name = (char*)(malloc(sizeof(char) * 100));
        strcpy(this->name,name);
        this->age = age;
    }

    /**注意这里的写法 有点像kotlin  赋值参数**/
    Student(char *name) : sex("男") {
        this->name = name;
    }

    Student() {
        printf("空参构造函数执行\n");
    }

    ~Student() //如果在对象内部堆内存上开辟了内存，一定要记得回收
    {
        printf("析构函数执行\n");
        free(this->name); //free 对应malloc
        this->name = NULL;

    }

    Student(const Student& student)
    {
        printf("对象的拷贝构造函数执行\n");
        this->name = student.name;
        this->age = student.age-1; //方便测试
    }

public:
    int getAge()
    {
        return this->age;
    }

    char* getName()
    {
        return this->name;
    }

    void setAge(int age) {
        Student::age = age;
    }

    void setName(char *name) {
        Student::name = name;
    }

    void printStudent(Student stu){// stu 是该方法栈中一个新的对象，拷贝构造函数赋值，方法执行完会调用析构函数
        cout << stu.getName() << " , " << stu.getAge() << endl;
    }


private:
    int age;
    char* name;
    char* sex;
};

```

**友元函数**：类的友元函数是定义在类外部，但有权访问类的所有私有（private）成员和保护（protected）成员。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。友元可以是一个函数，该函数被称为友元函数；友元也可以是一个类，该类被称为友元类，在这种情况下，整个类及其所有成员都是友元。

一般来说，将数据和处理数据的函数封装在一起，构成类，实现了数据的隐藏，无疑是面向对象程序设计的一大优点。但是有时候封装不是绝对的。友元函数提供了不同类或对象的成员函数之间、类的成员函数和一般函数之间进行数据共享的机制。通俗的说，友元关系就是一个类主动声明哪些类或函数是它的朋友，进而给它们提供对本类的访问特性。也就是说，通过友元关系，一个普通函数或者类的成员函数可以访问封装于另外一个类中的数据。

友元函数例子：

```c++
class Point {

public:
    Point(int xx = 0, int yy = 0) {
        X = xx;
        Y = yy;
    }

    int GetX() { return X; }

    int GetY() { return Y; }

    friend float fDist(Point &a, Point &b); //友元函数

private:
    int X, Y;
};


float fDist(Point &p1, Point &p2) {
    double x = double(p1.X - p2.X);//通过对象访问私有数据成员，而不是必须使用Getx()函数
    double y = double(p1.Y - p2.Y);
    return float(sqrt(x * x + y * y));
}

int main() {
    Point p1(1, 1), p2(4, 5);
    cout << "the distance is:";
    cout << fDist(p1, p2) << endl;//计算两点之间的距离
    return 0;
}
```

友元类例子：

```c++
//友元类和友元函数主要解决的问题就是 提供私有的变量给外界访问
class A {
public:
    int GetX() { return x; }

    friend class B;//B类是A类的友元类
    //其它成员略
private:
    int x;
};

class B {
public:
    void set(int i);
    //其他成员略
private:
    A a;
};

void B::set(int i) {
    a.x = i;//由于B类是A类的友元类，所以在B的成员函数中可以访问A类对象的私有成员
}

class C : private A{

};
//友元关系不能传递
//友元关系是单向的
//友元关系不能被继承
```

##### 操作符重载

C++ 运算符重载，函数重载（Function Overloading）可以让一个函数名有多种功能，在不同情况下进行不同的操作；运算符重载（Operator Overloading）也是一个道理，同一个运算符可以有不同的功能；实际上，我们已经在不知不觉中使用了运算符重载；例如，+号可以对不同类型（int、float 等）的数据进行加法操作；<<既是位移运算符，又可以配合 cout 向控制台输出数据；C++ 本身已经对这些运算符进行了重载。

```c++
bool ArrayList::operator==(ArrayList a)  {
    if (a.size != size) return false;

    for (int i = 0; i < (a.size); i++) {
        if (a[i] != array[i]) {
            cout << "false" << endl;
            return false;
        } else{
            cout << "true" << endl;
            return true;
        }
    }
}
```

#### 封装

None

#### 多态

多态就是程序运行时，父类指针可以根据具体指向的子类对象，来执行不同的函数，表现为多态。

看一下下面程序：

```c++
class People {
public:
    People(const char *name, int age); 
    void print() const;
protected:
    const char *m_name;
    int m_age;
};

People::People(const char *name, int age) : m_name(name), m_age(age) {
}
void People::print() const {
    printf("name: %s, age: %d\n", m_name, m_age);
}

class Student : public People {
public:
    Student(const char *name, int age, float score);
    void print() const;
private:
    float m_score;
};

Student::Student(const char *name, int age, float score) : People(name, age), m_score(score) {}
void Student::print() const {
    printf("name: %s, age: %d, score: %g\n", m_name, m_age, m_score);
}

int main() {
    People *p = nullptr;
    p = new People("Jesson", 25);
    p -> print();
    //delete p;

    p = new Student("Tom", 14, 87.5f);
    p -> print(); //p指针指向派生类的时候，只能调用派生类的成员变量，不能调用派生类的成员函数
    delete p;
    return 0;
}
```

运行的结果是：

```
name: Jesson, age: 25
name: Tom, age: 14
```

可以知道实际上Student对象调用print()并不是我们需要能调用本身的成员函数结果；

是因为C++规定：p指针指向派生类的时候，只能调用派生类的成员变量，不能调用派生类的成员函数,；

注： p这里是指针不是实例对象，如果是对象，是可以调用调用派生类的成员函数；

这时候就要引入虚函数了；

##### 虚函数

让基类指针能够访问派生类的成员函数，C++ 增加了虚函数（Virtual Function）；

指针调用普通的成员函数时会根据指针的类型（通过哪个类定义的指针）来判断调用哪个类的成员函数；而虚函数是根据指针的指向来调用的，指针指向哪个类的对象就调用哪个类的虚函数；

因此上面的代码只需要更改一下 print()函数为虚函数即可达到我们要想的结果：

```c++
class Student : public People {
public:
    Student(const char *name, int age, float score); //构造函数不能是虚函数 虚函数在c++ 只有1个作用 ： 多态
    virtual void print() const;//定义虚函数
private:
    float m_score;
};
```

##### **原理**：

当类中存在虚函数时，编译器会在类中自动生成一个虚函数表 Virtual table

- 虚函数表是一个存储类成员函数指针的数据结构
- 虚函数表由编译器自动生成和维护
- virtual 修饰的成员函数会被编译器放入虚函数表中 
- 存在虚函数时，编译器会为对象自动生成一个指向虚函数表的指针（通常称之为 vptr 指针）

虚函数对于多态具有决定性的作用，有虚函数才能构成多态，这节我们来重点说一下虚函数的注意事项：

​	1) 只需要在虚函数的声明处加上 virtual 关键字，函数定义处不加
​	2) 可以只将基类中的函数声明为虚函数，当派生类中出现参数列表相同的同名函数时，自动成为虚函数
​	3) 当在基类中定义了虚函数时，如果派生类没有定义新的函数来override此函数，那么将使用基类的虚函数
​	4) 构造函数不能是虚函数；对于基类的构造函数，它仅仅是在派生类构造函数中被调用，这种机制不同于继承；也就是说，派生类不继承基类的构造函数，将构造函数声明为虚函数没有什么意义
​	5) 析构函数有必要声明为虚函数

##### 为什么析构函数要用virtual--理解Android 时间传递

```c++
#include <cstdio>

using namespace std;

class A {
public:
    A() { printf("A()\n"); }
    virtual ~A() { printf("~A()\n"); }
};

class B : public A {
public:
    B() { printf("B()\n"); }
     ~B() { printf("~B()\n"); }
};

int main() {
    A *p = nullptr;
    p = new B; //子类进行了一些耗时的操作 内存泄漏
    delete p;
    return 0;
}
```



如果析构函数A不加 virtual 的话， 那边在delete p时，会无法调用到B的析构函数，从而导致内存泄漏；因此析构函数需要定义为虚析构函数；

##### 纯虚函数和抽象类

含有纯虚函数的类就是抽象类，之所以说它抽象，是因为它无法实例化，也就是无法创建对象；抽象类通常是作为基类，让派生类去实现纯虚函数；派生类必须实现纯虚函数才能被实例化

可定义如下：

```
virtual Type area() const = 0; 
```

```c++
class Line {
public:
    Line(float len);
    virtual ~Line() {}
    virtual float area() const = 0; //需要子类来实现
    virtual float volume() const = 0; //需要子类来实现
protected:
    float m_len;
};

Line::Line(float len) : m_len(len) {}

class Rectangle : public Line {
public:
    Rectangle(float len, float width);
    virtual float area() const override;
protected:
    float m_width;
};


Rectangle::Rectangle(float len, float width) : Line(len), m_width(width) {
	//Do somthing
}
float Rectangle::area() const {
	return m_len * m_width;
}


int main(){
    //todo 回去补齐
    //Rectangle line;
}
```



总结：

当通过指针访问类的成员函数时：

如果该函数是非虚函数，那么编译器会根据指针的类型找到该函数；也就是说，指针是哪个类的类型就调用哪个类的函数；

如果该函数是虚函数，并且派生类有同名的函数override它，那么编译器会根据指针的指向找到该函数；也就是说，指针指向的对象属于哪个类就调用哪个类的函数，这就是多态；



#### 继承

1) public继承方式
    基类中所有 public 成员在派生类中为 public 属性；
    基类中所有 protected 成员在派生类中为 protected 属性；
    基类中所有 private 成员在派生类中不能使用。

2) protected继承方式
	基类中的所有 public 成员在派生类中为 protected 属性；
	基类中的所有 protected 成员在派生类中为 protected 属性；
	基类中的所有 private 成员在派生类中不能使用。

3) private继承方式
	基类中的所有 public 成员在派生类中均为 private 属性；
	基类中的所有 protected 成员在派生类中均为 private 属性；
	基类中的所有 private 成员在派生类中不能使用。

```
继承语法 class Type ： 修饰符 Type
```

例子:

```c++
#include <printf.h>
#include <iostream>
using namespace std;

class People {
public:
    People(){}
    People(const char *m_name, int m_age) : m_name(m_name), m_age(m_age) {}

public:
    void setname(const char *name) { m_name = name; }
    void setage(int age) { m_age = age; }
    const char *getname() const { return m_name; }
    int getage() const { return m_age; }
private:
    const char *m_name;
    int m_age;
protected:

};


//继承语法 class Type ： 修饰符 Type
class Student : public People {
public:
    Student() : People() {}

    Student(const char *name, int age):People(name,age){
        cout<<"Student有参构造"<<endl;
    };
    void setscore(float score) { m_score = score; }
    float getscore() const { return m_score; }
private:
    float m_score;

};

int main() {
    //指针对象 ->
    Student* s = new Student("jesson",99);;
    s->setscore(111);
    Student s1 ;
    s1.setname("jesson");
    s1.setage(18);
    s1.setscore(111);
    printf("name: %s, age: %d, score: %g\n", s->getname(), s->getage(), s->getscore());
    printf("name: %s, age: %d, score: %g\n", s1.getname(), s1.getage(), s1.getscore());
    return 0;
}

```



##### 使用 using 来改变权限；

```c++
class A {
protected:
    int m_a;
    char m_b;
    float m_c;
private:
    char *m_d;
};

class B : public A {
public:
    using A::m_a;
protected:
    using A::m_b;
    //using A::m_d; // 基类的private成员在派生类中不可见
private:
    using A::m_c;
};

int main() {
    B b;
    b.m_a = 1;
    return 0;
}
```

##### 派生类会覆盖父类的方法：

```c++
#include <cstdio>

using namespace std;

class A {
public:
    void func() {
        printf("class A: func()\n");
    }
};

class B : public A {
public:
    void func() {
        printf("class B: func()\n");
    }
};

int main() {
    B b;
    b.func();
    b.A::func();
    b.A::func(); //子类调用父类
    return 0;
}
```

输出结果为：

```
class B: func()
class A: func()
class A: func()
```

由此可见： 派生类调用父类的方法： childClass.parentClass::method()

##### c++内存模型

没有继承关系时：成员变量和成员函数会分开存储，成员变量存储在栈、堆、全局数据区，而成员函数存储在代码区；

有继承关系时：派生类的内存模型可以看成是基类成员变量和新增成员变量的总和，而所有成员函数仍然存储在代码区，由所有对象共享；

基类的成员变量排在前面，派生类的排在后面； c++中类的内存大小和函数无关，只和成员变量有关系;

```c++
#include <cstdio>

using namespace std;

class A {
public:
    A() : m_a(1) {} //这种写法很常见
protected:
    int m_a;
};

class B : public A {
public:
    B() : A(), m_a(2) {}
protected:
    int m_a;
};

class C : public B {
public:
    C() : B(), m_b(11) {}
protected:
    int m_b;
};

class D : public C {
public:
    D() : C(), m_b(22) {}
protected:
    int m_b;
};

//父类的成员变量是在前面的
int main() {
    D d;
    printf("sizeof(d) = %lu\n", sizeof(d));
    printf("A::m_a = %d, B::m_a = %d, C::m_b = %d, D::m_b = %d\n",
           *(int *)&d, //1
           *(int *)((long)&d + 4), //2
           *(int *)((long)&d + 8), //11
           *(int *)((long)&d + 12)); //22
    return 0;
}
```

输出结果：

```
sizeof(d) = 16
A::m_a = 1, B::m_a = 2, C::m_b = 11, D::m_b = 22
```

##### 构造函数

1.类的构造函数不能被继承；(即使继承了，它的名字和派生类的名字也不一样，不能成为派生类的构造函数，当然更不能成为普通的成员函数）

2.在派生类的构造函数中调用基类的构造函数；派生类构造函数中只能调用直接基类的构造函数，不能调用间接基类的；通过派生类创建对象时必须要调用基类的构造函数，这是语法规定.

```c++
class A {
public:
    A() {
        printf("class_A A()\n");
    }
    A(int) {
        printf("class_A A(int)\n");
    }
    A(char) {
        printf("class_A A(char)\n");
    }
};

class B : public A {
public:
    B() {
        printf("class_B B()\n");
    }
    B(char) {
        printf("class_B B(char)\n");
    }
};

class C : public B {
public:
    C() : B('A') {
        printf("class_C C()\n");
    }
};

int main() {
    C c;
    return 0;
}
```

输出结果：

```
class_A A()
class_B B(char)
class_C C()
```

##### 析构函数

析构函数也不能被继承；在派生类的析构函数中不用显式地调用基类的析构函数，因为每个类只有一个析构函数，编译器知道如何选择，无需程序员干涉；

析构函数的执行顺序和构造函数的执行顺序也刚好相反：

1, 创建派生类对象时，构造函数的执行顺序和继承顺序相同，即先执行基类构造函数，再执行派生类构造函数；

2, 而销毁派生类对象时，析构函数的执行顺序和继承顺序相反，即先执行派生类析构函数，再执行基类析构函数；

```

class A {
public:
    A() {
        printf("A()\n");
    }
    ~A() {
        printf("~A()\n");
    }
};

class B : public A {
public:
    B() {
        printf("B()\n");
    }
    ~B() {
        printf("~B()\n");
    }
};

class C : public B {
public:
    C() {
        printf("C()\n");
    }
    ~C() {
        printf("~C()\n");
    }
};

int main() {
    C c;
    return 0;
}
```

输出结果：

```
A()
B()
C()
~C()
~B()
~A()
```

##### 类的多继承----调用顺序

基类构造函数的调用顺序和它们在派生类构造函数中出现的顺序无关，而是和声明派生类时基类出现的顺序相同; 

```c++
class A {
public:
    A() {
        printf("A()\n");
    }
    ~A() {
        printf("~A()\n");
    }
protected:
    void print() {
        printf("A::print()\n");
    }
};

class B {
public:
    B() {
        printf("B()\n");
    }
    ~B() {
        printf("~B()\n");
    }
protected:
    void print() {
        printf("B::print()\n");
    }
};

class C : public A, public B {
public:
    C() : B(), A() {
        printf("C()\n");
    }
    ~C() {
        printf("~C()\n");
    }
public:
    void print() {
        A::print();
        B::print();
    }
};

int main() {
    C c;
    c.print();
    return 0;
}
```

输出结果：

```
A()
B()
C()
A::print()
B::print()
```

##### 缺省函数

当一个类只写一个命名，那么编译器会需要的时候生成6个成员函数： 一个缺省的构造函数，一个拷贝函数，一个析构函数，一个赋值运算符重载函数，一对取走运算符和一个this指针；

```c++
class A {
    public:
    	A(){}
    	A(const A&){}
    	~A(){}
    
    	A&operator=(const A&){}
    	
    	A*operator&(){}
    	const A*operator&()const {}
}
```



#### C++ 模板

C++是一门强类型语言，所以无法做到像动态语言（python javascript）那样子，编写一段通用的逻辑，可以把任意类型的变量传进去处理。泛型编程弥补了这个缺点，通过把通用逻辑设计为模板，摆脱了类型的限制，提供了继承机制以外的另一种抽象机制，极大地提升了代码的可重用性。

模板定义本身不参与编译，而是编译器根据模板的用户使用模板时提供的类型参数生成代码，再进行编译，这一过程被称为模板实例化。用户提供不同的类型参数，就会实例化出不同的代码。

例子：

```c++
//函数 重载 合并方法
//函数模板
template<typename T>
T sum(T a,T b){
    return a+b;
}
//类模板
template<class T>
class CallBack{
public:
    void onError(){
    }

    int onSuccess(T result){
        cout<<result<<endl;
        //printf("%d",result);
    }
};

template<class T>
class HttpCallBack : public CallBack<T>{

};

int main(){
    std::cout<<sum(1,2)<<std::endl;
    std::cout<<sum(1.0,2.0)<<std::endl;
    printf("%.1f",sum(1.0,2.3));

    CallBack<string >* callBack = new CallBack<string >();
    callBack->onSuccess("sss");

    HttpCallBack<string>* httpCallBack =new HttpCallBack<string>();
    httpCallBack->onSuccess("hello");
}

```

#### 异常 

与java的类似，可抛出任何类型的异常，catch (...) 表示捕获所有的异常；

例子：

```c++
#include <iostream>
#include <stdexcept>

using namespace std;

class Exception{
public:
    string message;
    Exception(char* msg){
        this->message = msg;
    }

    virtual const char* what() const{  //这里要注意 需要在Androidstudio下试一试
        return message.c_str();
    }
};

void yewu() throw (int,Exception)  {
    ///
}

void MyFunc(int c)
{
    if (c > numeric_limits< char> ::max())
        throw invalid_argument("MyFunc argument too large.");
    //...
}

//jni
int main(){
    try {
        int i =0;
        if(i == 0){
            throw 10.0;
            //throw new Exception("error");
        }
        //throw new Exception("error");
        //MyFunc(256);

    }catch (int num){
        cout<<"捕获到了异常"<<endl;
    }catch (Exception* ex){ //注意这里的* 另外这里的异常要交给Java处理  看bitmap的代码
        cout<<ex->what()<<endl;
    }catch (invalid_argument& e){
        cerr << e.what() << endl;
    }catch  (...){
        //捕获所有的异常 IOException RunTimeException Exception
        cout<<"捕获到了其他的异常信息"<<endl;
    }
}
```



#### C++ STL

C++ STL（标准模板库）是一套功能强大的 C++ 模板类，提供了通用的模板类和函数，这些模板类和函数可以实现多种流行和常用的算法和数据结构，如向量、链表、队列、栈。

| **组件**            | **描述**                                                     |
| ------------------- | ------------------------------------------------------------ |
| 容器（Containers）  | 容器是用来管理某一类对象的集合。C++ 提供了各种不同类型的容器，比如 deque、list、vector、map 等。 |
| 算法（Algorithms）  | 算法作用于容器。它们提供了执行各种操作的方式，包括对容器内容执行初始化、排序、搜索和转换等操作。 |
| 迭代器（iterators） | 迭代器用于遍历对象集合的元素。这些集合可能是容器，也可能是容器的子集。 |

##### 迭代器

迭代器是一种检查容器内元素并遍历元素的数据类型。它是一种行为类似指针的对象，它提供类似指针的功能，对容器的内容进行走访，而指针的各种行为中最常见也最重要的是“成员访问”。 因此，**迭代器最重要的编程工作就是对operator*和operator->进行重载工作**。

迭代器类型:

- 普通iterator
- 元素的只读迭代器类型: const_iterator
- 按逆序寻址元素的迭代器: reverse_iterator
- 元素的只读逆序迭代器: const_reverse_iterator

迭代器常用方法：

| 方法名           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| begin()          | 返回的迭代器指向容器内第一个元素                             |
| end()            | 返回的迭代器指向容器内“最后一个元素的下一个位置；end()操作返回的迭代器并不指向vector中任何实际的元素，它只是起到一个哨兵的作用，表示我们已经处理完vector中的所有元素。 |
| rbegin()         | 返回一个逆序迭代器，它指向容器的最后一个元素，rend()返回一个逆序的迭代器，它指向容器的第一个元素前面的位置。 |
| *iter2; iter2++; | 用于迭代器的寻址                                             |

例子：

```c++
#include<iterator>
#include <vector>
#include <utility>
#include <iostream>

using namespace std;

int main(){
    vector<int>::iterator iter;  // 将iter声明为int类型的向量迭代器
    vector<int> vec;
    vector<int>::const_iterator iter2;
    vector<int>::reverse_iterator iter3;
    vector<int>::const_reverse_iterator iter4;
    vector<int>::iterator iter5 = vec.begin();
    *iter2; iter2++; iter2+1; iter2--; //用于迭代器的寻址
```

##### pair

pair 本质上是key-value， pair 是一个模板类，

```c++
    template <class _T1, class _T2>
    struct _LIBCPP_TEMPLATE_VIS pair
#if defined(_LIBCPP_DEPRECATED_ABI_DISABLE_PAIR_TRIVIAL_COPY_CTOR)
    : private __non_trivially_copyable_base
#endif
    {
        typedef _T1 first_type;
        typedef _T2 second_type;

        _T1 first;
        _T2 second;

#if !defined(_LIBCPP_CXX03_LANG)
        pair(pair const&) = default;
        pair(pair&&) = default;
```

使用例子：

```c++
#include<iterator>
#include <vector>
#include <utility>
#include <iostream>
#include <cstring>
#include <string>
#include <list>

using namespace std;
typedef pair<string,string> author;

int main() {
    author aa("name","jesson");
    author bb("age","33");

    cout<<aa.second<<bb.first<<endl;

    int a = 8;
    string m = "James";
    pair<int, string> newone;

    newone = make_pair(a, m);
    int num = newone.first;
    string str = newone.second;
    cout<<num<<str<<endl;
}
```

##### && 

non-const reference (非静态引用)，VS const int&

##### Auto关键字

auto的原理就是根据后面的值，来自己推测前面的类型是什么。auto的作用就是为了简化变量初始化，如果这个变量有一个很长很长的初始化类型，就可以用auto代替；仅需要知道就行，一般不使用；

```c++
auto a = 1 + 2;            // type of a is int
```

##### Vector 容器

在数据存储上，有一种对象类型，它可以持有其它对象或指向其它对像的指针，这种对象类型就叫做容器。很简单，容器就是保存其它对象的对象。

容器的分类：

1. 顺序容器：是一种各元素之间有顺序关系的线性表，是一种线性结构的可序群集。顺序性容器中的每个元素均有固定的位置，除非用删除或插入的操作改变这个位置。顺序容器的元素排列次序与元素值无关，而是由元素添加到容器里的次序决定。顺序容器包括：vector(向量)、list（列表）、deque（队列）。
2. 关联容器：关联式容器是非线性的树结构，更准确的说是二叉树结构。各元素之间没有严格的物理上的顺序关系，也就是说元素在容器中并没有保存元素置入容器时的逻辑顺序。但是关联式容器提供了另一种根据元素特点排序的功能，这样迭代器就能根据元素的特点“顺序地”获取元素。元素是有序的集合，默认在插入的时候按升序排列。关联容器包括：map（集合）、set（映射）、multimap（多重集合）、multiset（多重映射）。
3. 容器适配器：本质上，适配器是使一种不同的行为类似于另一事物的行为的一种机制。容器适配器让一种已存在的容器类型采用另一种不同的抽象类型的工作方式实现。适配器是容器的接口，它本身不能直接保存元素，它保存元素的机制是调用另一种顺序容器去实现，即可以把适配器看作“它保存一个容器，这个容器再保存所有元素”。STL 中包含三种适配器：栈stack 、队列queue 和优先级队列priority_queue 。

注： 容器类自动申请和释放内存，因此无需new和delete操作。

| **标准容器类** |                           **特点**                           |
| :------------: | :----------------------------------------------------------: |
|   顺序性容器   |                                                              |
|     vector     |  从后面快速的插入与删除，直接访问任何元素（支持[ ] 操作符）  |
|     deque      | 从前面或后面快速的插入与删除，直接访问任何元素（支持[ ] 操作符） |
|      list      |     双链表，从任何地方快速插入与删除（不支持[ ] 操作符）     |
|    关联容器    |                                                              |
|      set       |                    快速查找，不允许重复值                    |
|    multiset    |                     快速查找，允许重复值                     |
|      map       |         一对多映射，基于关键字快速查找，不允许重复值         |
|    multimap    |          一对多映射，基于关键字快速查找，允许重复值          |
|   容器适配器   |                                                              |
|     stack      |                           后进先出                           |
|     queue      |                           先进先出                           |
| priority_queue |                 最高优先级元素总是第一个出列                 |

详细说明：

① vector 

　　（连续的空间存储,可以使用[]操作符）快速的访问随机的元素，快速的在末尾插入元素，但是在序列中间岁间的插入，删除元素要慢，而且如果一开始分配的空间不够的话，有一个重新分配更大空间，然后拷贝的性能开销。

② deque 

　　（小片的连续，小片间用链表相连，实际上内部有一个map的指针，因为知道类型，所以还是可以使用[]，只是速度没有vector快）快速的访问随机的元素，快速的在开始和末尾插入元素，随机的插入，删除元素要慢，空间的重新分配要比vector快,重新分配空间后，原有的元素不需要拷贝。对deque的排序操作，可将deque先复制到vector，排序后在复制回deque。

③ list 

　　（每个元素间用链表相连）访问随机元素不如vector快，随机的插入元素比vector快，对每个元素分配空间，所以不存在空间不够，重新分配的情况

④ set 

　　内部元素唯一，用一棵平衡树结构来存储，因此遍历的时候就排序了，查找也比较快的哦。 

⑤ map 

　　一对一的映射的结合，key不能重复。 

⑥ stack 

　　适配器，必须结合其他的容器使用，stl中默认的内部容器是deque。先进后出，只有一个出口，不允许遍历。 

⑦ queue 

　　是受限制的deque，内部容器一般使用list较简单。先进先出，不允许遍历。

###### API 及例子

- vector 的使用例子

  ```c++
  
  #include <vector>
  #include <queue>
  #include <string.h>
  #include <iostream>
  #include <stack>
  #include <algorithm>
  #include <list>
  #include <map>
  
  
  using namespace std;
  /**
   *  打印vetor的数据内容
   * @param v
   */
  void printvector(vector<int> v){
      vector<int>::iterator it=v.begin();
      for(it;it!=v.end();++it){
          cout<<*it<<" ";
      }
      cout<<endl;
  }
  
  
  int main(){
      //实例化
      vector<int> vec1;//默认初始化，vec1为空
      vector<int> vec2(vec1);//使用vec1初始化vec2
      vector<int> vec3(vec1.begin(),vec1.end());//使用vec1初始化vec2
      vector<int> vec4(10);//10个值为0的元素
      vector<int> vec5(10,4);//10个值为4的元素
      vector<string> vec6(10);//10个值为null的元素
      cout<<"获取容器容量大小"<<vec5.capacity()<<endl;
      cout<<"获取容器容量大小"<<vec5.size()<<endl;
  
      vector<int>::iterator iterator = vec5.begin(); //编译器要实现c++11
  	//插入
      vec5.insert(iterator,8);
  	//这里再次直接插入会错误，因为原来的iterator已经无效了，需要重新begin()来获取新的iterator.
  	//vec5.insert(iterator,5);
  
  	//把元素复制值到容器的最尾部，会扩大长度，会调用拷贝函数；
      vec5.push_back(100);
      vec5.push_back(101);
  
      cout<<"获取容器容量大小"<<vec5.capacity()<<endl;
      cout<<"获取下标元素"<<vec5[0]<<endl;
      cout<<"获取下标元素"<<vec5[11]<<endl;
  
      //注意C++在语法层面没有规定vector内部缓冲区提前增加容量的大小 这取决于STL的实现
  
      printvector(vec5);
  	//返回第一个元素的引用
      cout<<vec5.front()<<endl;
      vec5.end();
      vec5.clear(); //清空所有元素，size为0，但 capacity()不变；
      vec5.erase(vec5.begin(),vec5.end()); //清空
  
      vec5.size();//元素的个数
  
      cout<<"获取容器容量大小"<<vec5.capacity()<<endl;//思考这个数据 怎么让容量是0
  
      vector<int>().swap(vec5); //交换函数
      cout<<"获取容器容量大小"<<vec5.capacity()<<endl;
  
      return -1;
  }
  ```

  **capacity() VS size():**

  size()  -- 返回目前存在的元素数。即： 元素个数
  capacity() -- 返回容器能存储 数据的个数。 即：容器容量
  reserve() --设置 capacity 大小
  resize()  --设置 size ，重新指定有效元素的个数 ，区别与reserve()指定 容量的大小

  ![Image](https://images0.cnblogs.com/blog2015/744826/201506/101701375356214.png)

- List API 使用例子：

  ```c++
  #include <numeric>
  #include <algorithm>
  #include <list>
  #include <iostream>
  
  using namespace std;
  
  //创建一个list容器的实例LISTINT
  typedef list<int> LISTINT;
  //创建一个list容器的实例LISTCHAR
  typedef list<char> LISTCHAR;
  
  int main(){
      LISTINT listOne;
      LISTINT::iterator i;
  	//插入前面
      listOne.push_front (2);
      listOne.push_front (1);
  	//插入后面
      listOne.push_back (3);
      listOne.push_back (4);
  
      //从前向后显示listOne中的数据
      cout<<"listOne.begin()--- listOne.end():"<<endl;
      for (i = listOne.begin(); i != listOne.end(); ++i)
          cout << *i << " ";
      cout << endl;
  
      //从后向后显示listOne中的数据
      LISTINT::reverse_iterator ir;
      cout<<"listOne.rbegin()---listOne.rend():"<<endl;
      for (ir =listOne.rbegin(); ir!=listOne.rend();ir++) {
          cout << *ir << " ";
      }
      cout << endl;
  
      //使用STL的accumulate(累加)算法: #include <numeric>
      int result = accumulate(listOne.begin(), listOne.end(),0);
      cout<<"Sum="<<result<<endl;
      cout<<"------------------"<<endl;
  
      //--------------------------
      //用list容器处理字符型数据
      //--------------------------
  
      //用LISTCHAR创建一个名为listOne的list对象
      LISTCHAR listTwo;
      //声明i为迭代器
      LISTCHAR::iterator j;
  
      //从前面向listTwo容器中添加数据
      listTwo.push_front ('A');
      listTwo.push_front ('B');
  
      //从后面向listTwo容器中添加数据
      listTwo.push_back ('x');
      listTwo.push_back ('y');
  
      //从前向后显示listTwo中的数据
      cout<<"listTwo.begin()---listTwo.end():"<<endl;
      for (j = listTwo.begin(); j != listTwo.end(); ++j)
          cout << char(*j) << " ";
      cout << endl;
  
      //使用STL的max_element算法求listTwo中的最大元素并显示: #include <algorithm>
      j=max_element(listTwo.begin(),listTwo.end());
      cout << "The maximum element in listTwo is: "<<char(*j)<<endl;
  }
  ```

  

- 容器适配器例子

  ```c++
  using namespace std;
  
  int main(){
      // stack: 后进先出
      // stack实例化
      stack<int> stack1;
      stack<vector<int>> stack2;
      stack<int> stack3(stack1);
  
      stack1.push(1);
      stack1.push(11);
      stack1.push(111);
      stack1.push(1111);
  
      while (stack1.size()>0){
          cout<<"top-"<<stack1.top()<<endl;
          stack1.pop();
      }
  
      cout<<stack1.empty()<<endl; //1 true 0 false
  
  	//队列： 先进先出
      queue<int> queue1;
      queue1.push(100);
      queue1.push(200);
      queue1.push(300);
      while (queue1.size()>0){
          //返回第一个元素(队顶元素); back() 返回最后被压入的元素(队尾元素)
  		cout<<queue1.front()<<endl;
          queue1.pop();
      }
  
      cout<<queue1.size()<<endl;
  
      //优先级队列：当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出的行为特征；
      //定义：priority_queue<Type, Container, Functional>:
      //Type 就是数据类型，Container 就是容器类型（Container必须是用数组实现的容器，比如vector,deque等等，但不能用 list。STL里面默认用的是vector），Functional 就是比较的方式; 
      // less<T>: 降序队列; greater<T>: 升序队列
      priority_queue<string,vector<string>,less<string>> priority_queue1;
      ////对于基础类型 默认是大顶堆
      //priority_queue<int> a; 
      //等同于 priority_queue<int, vector<int>, less<int> > a;
      priority_queue1.push("abs");
      priority_queue1.push("ccc");
      priority_queue1.push("ddd");
      priority_queue1.push("wqwqw");
      while(!priority_queue1.empty()){
  		cout<< "top-" << priority_queue1.top() <<endl;
          priority_queue1.pop();
      }
  }
  ```

  对于优先队列，我们进行自定义排序：

  ```c++
  //-----------优先级队列----实现自定义排序------
  struct Node
  {
      char szName[20];
      int  priority;
      Node(int nri, char *pszName)
      {
          strcpy(szName, pszName);
          priority = nri;
      }
  };
  
  struct NodeCmp
  {
  	//重写仿函数；可参考: https://blog.csdn.net/codedoctor/article/details/79654690
  	//返回false,表示 nb 需要和na交换，排到前面去；
      bool operator()(const Node &na, const Node &nb)
      {
          if (na.priority != nb.priority)
              return na.priority <= nb.priority;
          else
              return strcmp(na.szName, nb.szName) > 0;
      }
  };
  
  void printfNode(const Node &na)
  {
      printf("%s %d\n", na.szName, na.priority);
  }
  
  
  int main(){
  
      priority_queue<Node, vector<Node>, NodeCmp> a;
  
      a.push(Node(5, "小谭"));
      a.push(Node(3, "小刘"));
      a.push(Node(1, "小涛"));
      a.push(Node(5, "小王"));
      //队头的2个人出队
      printfNode(a.top());
      a.pop();
      a.pop();
      printf("--------------------\n");
  
      //再进入3个人
      a.push(Node(2, "小白"));
      a.push(Node(2, "小强"));
      a.push(Node(3, "小新"));
      //所有人都依次出队
      while (!a.empty()) {
          printfNode(a.top());
          a.pop();
      }
  }
  ```

  

- 迭代器的多重用法

  ```c++
  #include <vector>
  #include <iostream>
  #include <string>
  #include <algorithm>
  using namespace std;
  
  //迭代器的多重用法
  int main(){
      //HEAP
      int nArr[10] = {0,2,8,9,1,4,3,7,5,6};
      vector<int> vec(nArr,nArr+10);    //构建载体vector
      make_heap(vec.begin(),vec.end()); //围绕vec构建一个堆
      sort_heap(vec.begin(),vec.end()); //堆排序
      for (int i = 0; i<vec.size(); ++i) {
          cout << vec[i]  << endl;
      }
  
      //第二种遍历方式，迭代器
      cout << "第二种遍历方式，迭代器访问" << endl;
      for (auto iter = vec.begin(); iter != vec.end(); iter++)
      {
          cout << (*iter)<< endl;
      }
      cout << "第3种遍历方式，auto" << endl;
      //第二种遍历方式，auto
      for(auto i : vec){
          cout << i<< endl;
      }
  }
  ```

  #### 输入输出流

  | 头文件     | 函数与描述                                                   |
  | ---------- | ------------------------------------------------------------ |
  | <iostream> | 标准输入输出流： cin， cout, cerr, clog对象；                |
  | <iomanip>  | 通过所谓的参数化为的流操纵器，来声明对执行标准化I/O有用的服务 |
  | <fstream>  | 为用户控制的文件处理声明服务。                               |

  

#### 相关文章网址

[C++](https://en.cppreference.com/w/)

[C语言相关方法](https://en.cppreference.com/w/c/language)

[C++编译过程](https://www.cnblogs.com/mickole/articles/3659112.html)

#### 作业：

1. 面向对象三大特征？

   封装，继承，多态

2. 虚析构函数是什么？

   在析构函数定义的加多一个 virtual 关键字； 主要用于多态时，进行释放对象时，才会调用的其子类的析构函数。

3. 指针和引用的区别？

   1. 指针有自己的一块空间，而引用只是一个别名；

   2. 使用sizeof看一个指针的大小是4，而引用则是被引用对象的大小；

   3. 指针可以被初始化为NULL，而引用必须被初始化且必须是一个已有对象的引用；

   4. 作为参数传递时，指针需要被解引用才可以对对象进行操作，而直接对引用的修改都会改变引用所指向的对象；

   5. 可以有const指针，但是没有const引用；

   6. 指针在使用中可以指向其它对象，但是引用只能是一个对象的引用，不能 被改变；

   7. 指针可以有多级指针（**p），而引用至于一级；

   8. 指针和引用使用++运算符的意义不一样；

   9. 如果返回动态内存分配的对象或者内存，必须使用指针，引用可能引起内存泄露。

      可参考： https://www.cnblogs.com/WindSun/p/11434417.html

4. new和malloc的区别？

   https://blog.csdn.net/nyist_zxp/article/details/80810742
   https://blog.csdn.net/foroverontheroad/article/details/85272309

5. 虚函数是怎样实现的？为什么还要区分虚函数和普通函数，都成虚函数不好么？

