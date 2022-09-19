## Python基础笔记

[TOC]

本文笔记来自于python编程入门到实践第2版， 其中python3;

### 基础概念

#### 命名

- 变量名只能包含字母、数字和下划线。变量名能以字母或下划线打头，但不能以数字打头。
- 变量名不能包含空格，但能使用下划线来分隔其中的单词；
- 不要将Python关键字和函数名用作变量名，即不要使用Python保留用于特殊用途的单词；
- 变量名应既简短又具有描述性；
- 慎用小写字母l 和大写字母O ，因为它们可能被人错看成数字1 和0。

#### 字符串

在Python中，用引号括起的都是字符串，其中的引号可以是单引号，也可以是双引号；

```python
"This is a string."
'This is also a string.'
```

常用方法：

- title(): 把每个单词的首个字母进行大写；

- upper()/lower(): 全部字母改大写或小写；

- f字符串： python3.6加入，之前则需要使用format()方法； 

  ```python
  first_name = "ada"
  last_name = "lovelace"
  full_name = f"{first_name} {last_name}"
  message = f"Hello, {full_name.title()}!"
  print(message) #输出：Hello, Ada Lovelace!
  
  #旧版本
  full_name = "{} {}".format(first_name, last_name)
  ```

- rstrip()/lstrip()/rstrip()： 删除字符串开头和末尾多余的空白；

#### 数（Number）

##### 整数(int)

- 对整数执行加（+ ）减（- ）乘（* ）除（/ ）运算； 

- Python使用两个乘号表示乘方运算；

  ```python
  >>> 3 ** 2
  9
  ```

##### 浮点数(float)

Python将所有带小数点的数称为浮点数 。

从很大程度上说，使用浮点数时无须考虑其行为。你只需输入要使用的数，Python通常会按你期望的方式处理它们：

```python
>>> 0.1 + 0.1
0.2
>>> 0.2 + 0.2
0.4
>>> 2 * 0.1
0.2
>>> 2 * 0.2
0.4
```

但需要注意的是，结果包含的小数位数可能是不确定的：

```python
>>> 0.2 + 0.1
0.30000000000000004
>>> 3 * 0.1
0.30000000000000004
```

##### 整数VS浮点数

将任意两个数相除时，结果总是浮点数，即便这两个数都是整数且能整除； 无论是哪种运算，只要有操作数是浮点数，Python默认得到的总是浮点数，即便结果原本为整数也是如此。

```python
>>> 4/2
2.0
>>> 1 + 2.0
3.0
>>> 2 * 3.0
6.0
>>> 3.0 ** 2
9.0
```

##### Others

- 可使用下划线将其中的数字分组，使其更清晰易读;

  ```python
  >>> universe_age = 14_000_000_000
  >>> print(universe_age)
  14000000000
  ```

- 可在一行代码中给多个变量赋值;

  ```python
  >>> x, y, z = 0, 0, 0
  ```

- 使用全大写来指出应将某个变量视为常量，其值应始终不变:

  ```python
  MAX_CONNECTIONS = 5000
  ```

- 在Python中，注释用井号（# ）标识。井号后面的内容都会被Python解释器忽略；多行则使用输入''' '''或者""" """，将要注释的代码插在中间或使用Ctr+/（前提是选中需要注释的代码）

  ```python
  #注释
  '''
  这是
  多行注释
  '''
  """
  这是
  多行注释
  """
  ```

#### 列表(List)

在Python中，用方括号（[] ）表示列表，并用逗号分隔其中的元素。

##### 列表基础操作：

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
#访问列表元素
print(bicycles[0])
#修改元素
bicycles[0] = "yufen"
#添加元素:新元素添加到列表末尾，不影响其它元素
bicycles.append("new one")
#插入元素:可在列表的任何位置添加新元素，并将列表中既有的每个元素都右移一个位置；
bicycles.insert(3, 'new tow')
#删除某个元素：
del bicycles[2]
#删除列表末尾的元素,并返回其删除的元素
delete = bicycles.pop()
#删除列表中任意位置的元素
delete2 = bicycles.pop(3)
#根据元素值删除元素
bicycles.remove("redline")
```

##### 组织列表：

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']
#排序：按字母顺序
cars.sort()
#反向排序
cars.sort(reverse=True)
#返回排序后的，并不改变cars原来的数组
tmep = sorted(cars)

#根据student_tuples里元素第三个(age)来排序
student_tuples = [
        ('john', 'A', 15),
        ('jane', 'B', 12),
        ('dave', 'B', 10),
	]
sorted(student_tuples, key=lambda student: student[2])

#反转列表元素
cars.reverse()
#获取列表长度
len(cars)
```

##### 遍历整个列表：

```python
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
	print(magician)
    print(f"{magician.title()}, that was a great trick!")
```

- 在for 循环中，想包含多少行代码都可以。在代码行for magician in magicians 后面，每个缩进的代码行都是循环的一部分，将针对列表中的每个值都执行一次。
- for 语句末尾的冒号告诉Python，下一行是循环的第一行

Python根据缩进来判断代码行与前一个代码行的关系，所以需要注意每一行代码的缩进。

##### 创建数值列表

ython函数range() 让你能够轻松地生成一系列数。

```python
#输出： 1，2，3，4
for value in range(1, 5):
	print(value)
```

使用range() 创建数字列表：

```python
numbers = list(range(1, 6))
print(numbers)
```

使用函数range() 时，还可指定步长：range(2, 11, 2)

对数字列表执行简单的统计计算：

```python
>>> digits = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]
>>> min(digits)
0 
>>> max(digits)
9 
>>> sum(digits)
45
```

##### 列表解析

列表解析 将for 循环和创建新元素的代码合并成一行，并自动附加新元素。

```python
squares = [value**2 for value in range(1, 11)]
print(squares) #输出结果：[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

在这个示例中，表达式为value2 ，它计算平方值。接下来，编写一个for 循环，用于给表达式提供值，再加上右方括号。在这个示例中，for 循环为for value in range(1,11) ，它将值1～10提供给表达式value2 。请注意，这里的for 语句末尾没有冒号。

##### 使用列表的一部分

切片

```python
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[1:3])#输出列表中的前三个元素:['martina', 'michael']
print(players[:4])#输出列表中从头开始的4个元素：['charles', 'martina', 'michael', 'florence']
print(players[2:])#输出从第三个元素到列表末尾的所有元素
print(players[-3:])#输出列表末尾的3个元素
```

复杂列表

要复制列表，可创建一个包含整个列表的切片，方法是同时省略起始索引和终止索引（[:] ）。

```python
my_foods = ['pizza', 'falafel', 'carrot cake']
friend_foods = my_foods[:]
```

#### 元组

Python将不能修改的值称为不可变的 ，而不可变的列表被称为元组 。定义元组(使用圆括号)后，就可使用索引来访问其元素，就像访问列表元素一样。

```python
dimensions = (200, 50)
print(dimensions[0])
print(dimensions[1])
```

元组不可修改元素值，但可重新给存储元组的变量赋值：

```python
dimensions = (200, 50)
print("Original dimensions:")
for dimension in dimensions:
	print(dimension)
dimensions = (400, 100)
print("\nModified dimensions:")
for dimension in dimensions:
	print(dimension)
```

python代码格式推荐：

- 每级缩进都使用四个空格。这既可提高可读性，又留下了足够的多级缩进空间。
- 建议每行不超过80字符。
- 要将程序的不同部分分开，可使用空行。



#### `if`语句

每条if 语句的核心都是一个值为True 或False 的表达式，这种表达式称为条件测试。

```python
if conditional_test:
    do something
```

`if-else` 结构:

```python
age = 17
if age >= 18:
	print("You are old enough to vote!")
	print("Have you registered to vote yet?")
else:
	print("Sorry, you are too young to vote.")
	print("Please register to vote as soon as you turn 18!")
```

`if-elif-else `结构:

```python
age = 12
if age < 4:
	print("Your admission cost is $0.")
elif age < 18:
	print("Your admission cost is $25.")
elif age < 28:
	print("Your admission cost is $35.")
else:
	print("Your admission cost is $40.")
```

其中 else 可省略；

使用if处理列表:

- 确定列表不是空的

  ```python
  requested_toppings = []
  if requested_toppings:
  	for requested_topping in requested_toppings:
  		print(f"Adding {requested_topping}.")
  		print("\nFinished making your pizza!")
  else:
  	print("Are you sure you want a plain pizza?")
  ```

- 是否在某列表中

  ```python
  available_toppings = ['mushrooms', 'olives', 'green peppers','pepperoni', 'pineapple', 'extra cheese']
  if requested_topping in available_toppings:
  	print(f"Adding {requested_topping}.")
  ```

#### 字典

字典是另一种可变容器模型，且可存储任意类型对象。字典的每个键值 **key:value** 对用冒号 **:** 分割，每个键值对之间用逗号 **,** 分割，整个字典包括在花括号 **{}** 中 ,格式如下所示：

```python
d = {key1 : value1, key2 : value2 }
```

访问字典的值：

```python
alien_0 = {'color': 'green'}
print(alien_0['color'])

alien_0 = {'color': 'green', 'speed': 'slow'}
point_value = alien_0.get('points', 'No point value assigned.')
print(point_value)
```

字典是一种动态结构，可随时在其中添加键值对。要添加键值对，可依次指定字典名、用方括号括起的键和相关联的值。

```python
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)
alien_0['x_position'] = 0
alien_0['y_position'] = 25
print(alien_0)
```

创建字典：

```python
alien_0 = {}#创建空字典
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }
```

删除键值对:

```
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)
del alien_0['points']
print(alien_0)
```

遍历所有键值对:

```python
user_0 = {
    'username': 'efermi',
    'first': 'enrico',
    'last': 'fermi',
    }
#items()返回一个键值对列表
for key, value in user_0.items(): 
	print(f"\nKey: {key}")
	print(f"Value: {value}")
for key in user_0.keys():
	print(name.title())
for value in user_0.values():
	print(value.title())

#直接提取字典中所有的值，可能有大量value是重复的，因此可使用set来确保元素是独一无二的
for value in set(user_0.values()):
	print(value.title())
```

按特定顺序遍历字典中的所有键:





#### 函数

函数是带名字的代码块，用于完成具体的工作。要执行函数定义的特定任务，可调用 该函数。需要在程序中多次执行同一项任务时，无须反复编写完成该任务的代码，只需要调用执行该任务的函数，让Python运行其中的代码即可。你将发现，通过使用函数，程序编写、阅读、测试和修复起来都更加容易。

##### 定义函数

```python
def greet_user():
	"""显示简单的问候语。"""
	print("Hello!")
    
greet_user()
```

##### 向函数传递信息

```python
def greet_user(username):
	"""显示简单的问候语。"""
	print(f"Hello, {username.title()}!")
greet_user('jesse')
```

##### 传参

```python
def describe_pet(animal_type, pet_name):
	"""显示宠物的信息。"""
	print(f"\nI have a {animal_type}.")
	print(f"My {animal_type}'s name is {pet_name.title()}.")
    
describe_pet('hamster', 'harry')
describe_pet('fdsf', '3423')
```

```python
def describe_pet(animal_type, pet_name):
	"""显示宠物的信息。"""
	print(f"\nI have a {animal_type}.")
	print(f"My {animal_type}'s name is {pet_name.title()}.")
	
describe_pet(animal_type='hamster', pet_name='harry')
```

默认值:

```python
def describe_pet(pet_name, animal_type='dog'):
	"""显示宠物的信息。"""
	print(f"\nI have a {animal_type}.")
	print(f"My {animal_type}'s name is {pet_name.title()}.")
describe_pet(pet_name='willie')
```

##### 返回值

```python
def get_formatted_name(first_name, last_name):
	"""返回整洁的姓名。"""
	full_name = f"{first_name} {last_name}"
	return full_name.title()

musician = get_formatted_name('jimi', 'hendrix')
print(musician)
```

函数可返回任何类型的值，包括列表和字典等较复杂的数据结构：

```python
def build_person(first_name, last_name):
	"""返回一个字典，其中包含有关一个人的信息。"""
	person = {'first': first_name, 'last': last_name}
	return person
	
musician = build_person('jimi', 'hendrix')
print(musician)
```

```python
def build_person(first_name, last_name, age=None):
	"""返回一个字典，其中包含有关一个人的信息。"""
	person = {'first': first_name, 'last': last_name}
	if age:
		person['age'] = age
	return person
	
musician = build_person('jimi', 'hendrix', age=27)
print(musician)
```

新增了一个可选形参age ，并将其默认值设置为特殊值**None** （表示变量没有值）。可将**None** 视为占位值。在条件测试中，**None** 相当于**False** 。

##### 在函数中修改列表

```python
def print_models(unprinted_designs, completed_models):
	"""
	模拟打印每个设计，直到没有未打印的设计为止。
	打印每个设计后，都将其移到列表completed_models中。
	"""
	while unprinted_designs:
	current_design = unprinted_designs.pop()
	print(f"Printing model: {current_design}")
	completed_models.append(current_design)

def show_completed_models(completed_models):
	"""显示打印好的所有模型。"""
	print("\nThe following models have been printed:")
	for completed_model in completed_models:
	print(completed_model)


unprinted_designs = ['phone case', 'robot pendant', 'dodecahedron']
completed_models = []
print_models(unprinted_designs, completed_models)
show_completed_models(completed_models)
```

##### 禁止函数修改列表

为解决这个问题，可向函数传递列表的副本而非原件。这样，函数所做的任何修改都只影响副本，而原件丝毫不受影响。

```python
unction_name(list_name_[:])
```

##### 传递任意数量的实参

有时候，预先不知道函数需要接受多少个实参，好在Python允许函数从调用语句中收集任意数量的实参。

```python
def make_pizza(*toppings):
	"""打印顾客点的所有配料。"""
	print(toppings)
	
make_pizza('pepperoni')
make_pizza('mushrooms', 'green peppers', 'extra cheese')
```

形参名*toppings 中的星号让Python创建一个名为toppings 的空元组，并将收到的所有值都封装到这个元组中。

##### 结合使用位置实参和任意数量实参

如果要让函数接受不同类型的实参，必须在函数定义中将接纳任意数量实参的形参放在最后。Python先匹配位置实参和关键字实参，再将余下的实参都收集到最后一个形参中。

```python
def make_pizza(size, *toppings):
    """概述要制作的比萨。"""
    print(f"\nMaking a {size}-inch pizza with the following toppings:")
    for topping in toppings:
    print(f"- {topping}")
    
make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

##### 使用任意数量的关键字实参

有时候，需要接受任意数量的实参，但预先不知道传递给函数的会是什么样的信息。在这种情况下，可将函数编写成能够接受任意数量的键值对——调用语句提供了多少就接受多少。

```python
def build_profile(first, last, **user_info):
    """创建一个字典，其中包含我们知道的有关用户的一切。"""
    ❶ user_info['first_name'] = first
    user_info['last_name'] = last
    return user_info
user_profile = build_profile('albert', 'einstein',
                            location='princeton',
                            field='physics')
print(user_profile)
```

##### 将函数存储在模块中

使用函数的优点之一是可将代码块与主程序分离。通过给函数指定描述性名称，可让主程序容易理解得多。你还可以更进一步，将函数存储在称为模块 的独立文件中，再将模块导入 到主程序中。import 语句允许在当前运行的程序文件中使用模块中的代码。

通过将函数存储在独立的文件中，可隐藏程序代码的细节，将重点放在程序的高层逻辑上。这还能让你在众多不同的程序中重用函数。将函数存储在独立文件中后，可与其他程序员共享这些文件而不是整个程序。知道如何导入函数还能让你使用其他程序员编写的函数库。

###### 导入整个模块

要让函数是可导入的，得先创建模块。模块 是扩展名为.py的文件，包含要导入到程序中的代码。

```python
## pizza.py
def make_pizza(size, *toppings):
    """概述要制作的比萨。"""
    print(f"\nMaking a {size}-inch pizza with the following toppings:")
    for topping in toppings:
    	print(f"- {topping}")

## 在pizza.py所在的目录中创建一个名为making_pizzas.py的文件
import pizza

pizza.make_pizza(16, 'pepperoni')
pizza.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

Python读取making_pizzas.py文件时，代码行import pizza 让Python打开文件pizza.py，并将其中的所有函数都复制到这个程序中。你看不到复制的代码，因为在这个程序即将运行时，Python在幕后复制了这些代码。你只需知道，在making_pizzas.py中，可使用pizza.py中定义的所有函数。

###### 导入特定的函数

通过用逗号分隔函数名，可根据需要从模块中导入任意数量的函数：

```python
from module_name import function_0, function_1, function_2
```

比如：

```python
from pizza import make_pizza
make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

###### 使用as 给函数指定别名

```python
from pizza import make_pizza as mp
mp(16, 'pepperoni')
mp(12, 'mushrooms', 'green peppers', 'extra cheese')
```

###### 使用as 给模块指定别名

```python
import pizza as p
p.make_pizza(16, 'pepperoni')
p.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

###### 导入模块中的所有函数

```python
from pizza import *
make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

##### 函数编写指南

- 应给函数指定描述性名称，且只在其中使用小写字母和下划线。描述性名称可帮助你和别人明白代码想要做什么。给模块命名时也应遵循上述约定。

- 每个函数都应包含简要地阐述其功能的注释。该注释应紧跟在函数定义后面，并采用文档字符串格式。

- 给形参指定默认值时，等号两边不要有空格； 对于函数调用中的关键字实参，也应遵循这种约定； 

  ```python
  def function_name(parameter_0, parameter_1='default value')
  
  function_name(value_0, parameter_1='value')
  ```

#### 类

##### 创建和使用类

###### 创建Dog 类

```python
#类名首字母大写
class Dog:
    """一次模拟小狗的简单尝试。"""
    def __init__(self, name, age):
        """初始化属性name和age。"""
        self.name = name
        self.age = age
        
    def sit(self):
        """模拟小狗收到命令时蹲下。"""
        print(f"{self.name} is now sitting.")
        
    def roll_over(self):
        """模拟小狗收到命令时打滚。"""
        print(f"{self.name} rolled over!")
```

\_\_init\_\_(): 是一个特殊方法，每当你根据Dog 类创建新实例时，Python都会自动运行它。在这个方法的名称中，开头和末尾各有两个下划线，这是一种约定，旨在避免Python默认方法与普通方法发生名称冲突。务必确保__init__() 的两边都有两个下划线，否则当你使用类来创建实例时，将不会自动调用这个方法，进而引发难以发现的错误。

- 形参self : 必须位于其他形参的前面; Python调用这个方法来创建Dog 实例时，将自动传入实参self 。每个与实例相关联的方法调用都自动传递实参self ，它是一个指向实例本身的引用，让实例能够访问类中的属性和方法。
- 以self 为前缀的变量可供类中的所有方法使用，可以通过类的任何实例来访问。self.name = name 获取与形参name 相关联的值，并将其赋给变量name ，然后该变量被关联到当前创建的实例。self.age = age 的作用与此类似。像这样可通过实例访问的变量称为属性 。

##### 根据类创建实例

```python
my_dog = Dog('Willie', 6)
print(f"My dog's name is {my_dog.name}.")
print(f"My dog is {my_dog.age} years old.")
# 访问属性
my_dog.name
# 调用方法
my_dog.sit()
my_dog.roll_over()
```

##### 使用类和实例

```python
class Car:
    """一次模拟汽车的简单尝试。"""
    def __init__(self, make, model, year):
        """初始化描述汽车的属性。"""
        self.make = make
        self.model = model
        self.year = year
        #给属性指定默认值
        self.odometer_reading = 0
        
    def get_descriptive_name(self):
        """返回整洁的描述性信息。"""
        long_name = f"{self.year} {self.make} {self.model}"
        return long_name.title()
    
    #通过方法修改属性的值
    def update_odometer(self, mileage):
        """将里程表读数设置为指定的值。"""
        self.odometer_reading = mileage
    
my_new_car = Car('audi', 'a4', 2019)
print(my_new_car.get_descriptive_name())
#修改属性的值
my_new_car.odometer_reading = 23
```

##### 继承

一个类继承 另一个类时，将自动获得另一个类的所有属性和方法。原有的类称为父类 ，而新类称为子类 。子类继承了父类的所有属性和方法，同时还可以定义自己的属性和方法。

###### 子类的方法`__init__()`

在既有类的基础上编写新类时，通常要调用父类的方法\__init\_\_() 。这将初始化在父类`__init__()` 方法中定义的所有属性，从而让子类包含这些属性。

```python
class Car:
    """一次模拟汽车的简单尝试。"""
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0
        
    def get_descriptive_name(self):
        long_name = f"{self.year} {self.make} {self.model}"
        return long_name.title()
        
    def read_odometer(self):
        print(f"This car has {self.odometer_reading} miles on it.")
        
    def update_odometer(self, mileage):
        if mileage >= self.odometer_reading:
        self.odometer_reading = mileage
        else:
        print("You can't roll back an odometer!")
        
    def increment_odometer(self, miles):
    	self.odometer_reading += miles
        
class ElectricCar(Car):
    """电动汽车的独特之处。"""
    def __init__(self, make, model, year):
        """初始化父类的属性。"""
        super().__init__(make, model, year)
        

my_tesla = ElectricCar('tesla', 'model s', 2019)
print(my_tesla.get_descriptive_name())
```

###### 给子类定义属性和方法

让一个类继承另一个类后，就可以添加区分子类和父类所需的新属性和新方法了。

```python
class ElectricCar(Car):
    """电动汽车的独特之处。"""
    def __init__(self, make, model, year):
        """
        初始化父类的属性。
        再初始化电动汽车特有的属性。
        """
        super().__init__(make, model, year)
        self.battery_size = 75
        
    def describe_battery(self):
        """打印一条描述电瓶容量的消息。"""
        print(f"This car has a {self.battery_size}-kWh battery.")
        
my_tesla = ElectricCar('tesla', 'model s', 2019)
print(my_tesla.get_descriptive_name())
my_tesla.describe_battery()
```

###### 重写父类的方法

假设Car 类有一个名为fill_gas_tank() 的方法，它对全电动汽车来说毫无意义，因此你可能想重写它。

```python
class ElectricCar(Car):
    --snip--
    def fill_gas_tank(self):
        """电动汽车没有油箱。"""
        print("This car doesn't need a gas tank!")
```

###### 将实例用作属性

```python
class Car:
	--snip--
    
class Battery:
    """一次模拟电动汽车电瓶的简单尝试。"""
    def __init__(self, battery_size=75):
        """初始化电瓶的属性。"""
        self.battery_size = battery_size
        
    def describe_battery(self):
        """打印一条描述电瓶容量的消息。"""
        print(f"This car has a {self.battery_size}-kWh battery.")
    
class ElectricCar(Car):
    """电动汽车的独特之处。"""
    def __init__(self, make, model, year):
        """
        初始化父类的属性。
        再初始化电动汽车特有的属性。
        """
        super().__init__(make, model, year)
        self.battery = Battery()

my_tesla = ElectricCar('tesla', 'model s', 2019)
print(my_tesla.get_descriptive_name())
my_tesla.battery.describe_battery()
```

##### 导入类

随着不断给类添加功能，文件可能变得很长，即便妥善地使用了继承亦如此。为遵循Python的总体理念，应让文件尽可能整洁。Python在这方面提供了帮助，允许将类存储在模块中，然后在主程序中导入所需的模块。

在Python中，一个.py文件就称为一个模块；

###### 导入单个类

将Car 类存储在一个名为car.py的模块中，该模块将覆盖前面使用的文件car.py。

```python
#car.py
"""一个可用于表示汽车的类。"""
class Car:
    """一次模拟汽车的简单尝试。"""
    def __init__(self, make, model, year):
        """初始化描述汽车的属性。"""
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0
        
    def get_descriptive_name(self):
        """返回整洁的描述性名称。"""
        long_name = f"{self.year} {self.make} {self.model}"
        return long_name.title()
    
    def read_odometer(self):
        """打印一条消息，指出汽车的里程。"""
        print(f"This car has {self.odometer_reading} miles on it.")
        
    def update_odometer(self, mileage):
        """
        将里程表读数设置为指定的值。
        拒绝将里程表往回调。
        """
        if mileage >= self.odometer_reading:
        self.odometer_reading = mileage
        else:
        print("You can't roll back an odometer!")
        
    def increment_odometer(self, miles):
        """将里程表读数增加指定的量。"""
        self.odometer_reading += miles
```

创建另一个文件my_car.py，在其中导入Car 类并创建其实例：

```python
#import 语句让Python打开模块car 并导入其中的Car 类
from car import Car

my_new_car = Car('audi', 'a4', 2019)
print(my_new_car.get_descriptive_name())
my_new_car.odometer_reading = 23
my_new_car.read_odometer()
```

###### 在一个模块中存储多个类

Battery 类和ElectricCar 类都写模块car.py文件；

代码略

###### 从一个模块中导入多个类

```python
from car import Car, ElectricCar

my_beetle = Car('volkswagen', 'beetle', 2019)
print(my_beetle.get_descriptive_name())
my_tesla = ElectricCar('tesla', 'roadster', 2019)
print(my_tesla.get_descriptive_name())
```

###### 导入整个模块

```python
import car

my_beetle = car.Car('volkswagen', 'beetle', 2019)
print(my_beetle.get_descriptive_name())
my_tesla = car.ElectricCar('tesla', 'roadster', 2019)
print(my_tesla.get_descriptive_name())
```

导入了整个car 模块。接下来，使用语法module_name.ClassName 访问需要的类。

###### 导入模块中的所有类

```python
from module_name import *
```

但不推荐使用这种导入方式：

- 这种导入方式没有明确地指出使用了模块中的哪些类。
- 这种方式还可能引发名称方面的迷惑。如果不小心导入了一个与程序文件中其他东西同名的类，将引发难以诊断的错误。

需要从一个模块中导入很多类时，最好导入整个模块，并使用module_name.ClassName 语法来访问类。这样做时，虽然文件开头并没有列出用到的所有类，但你清楚地知道在程序的哪些地方使用了导入的模块。这也避免了导入模块中的每个类可能引发的名称冲突。

###### 在一个模块中导入另一个模块

有时候，需要将类分散到多个模块中，以免模块太大或在同一个模块中存储不相关的类。将类存储在多个模块中时，你可能会发现一个模块中的类依赖于另一个模块中的类。在这种情况下，可在前一个模块中导入必要的类。

下面将Car 类存储在一个模块中，并将ElectricCar 类和Battery类存储在另一个模块中。将第二个模块命名为electric_car.py（这将覆盖前面创建的文件electric_car.py），并将Battery 类和ElectricCar 类复制到这个模块中：

electric_car.py

```python
"""一组可用于表示电动汽车的类。"""
from car import Car

class Battery:
	--snip--
class ElectricCar(Car):
	--snip--
```

car.py

```python
"""一个可用于表示汽车的类。"""

class Car:
	--snip--
```

现在可以分别从每个模块中导入类，以根据需要创建任何类型的汽车了：

my_cars.py

```python
from car import Car
from electric_car import ElectricCar

my_beetle = Car('volkswagen', 'beetle', 2019)
print(my_beetle.get_descriptive_name())
my_tesla = ElectricCar('tesla', 'roadster', 2019)
print(my_tesla.get_descriptive_name())
```

###### 使用别名

使用模块来组织项目代码时，别名大有裨益。导入类时，也可为其指定别名。

```python
from electric_car import ElectricCar as EC
```

###### 自定义工作流程

一开始应让代码结构尽可能简单。先尽可能在一个文件中完成所有的工作，确定一切都能正确运行后，再将类移到独立的模块中。如果你喜欢模块和文件的交互方式，可在项目开始时就尝试将类存储到模块中。先找出让你能够编写出可行代码的方式，再尝试改进代码。

##### Python标准库

Python标准库 是一组模块，我们安装的Python都包含它。你现在对函数和类的工作原理已有大致的了解，可以开始使用其他程序员编写好的模块了。可以使用标准库中的任何函数和类，只需在程序开头包含一条简单的import 语句即可。

##### 类编码风格

- 类名应采用驼峰命名法 ，即将类名中的每个单词的首字母都大写，而不使用下划线。实例名和模块名都采用小写格式，并在单词之间加上下划线。
- 对于每个类，都应紧跟在类定义后面包含一个文档字符串。这种文档字符串简要地描述类的功能，并遵循编写函数的文档字符串时采用的格式约定。每个模块也都应包含一个文档字符串，对其中的类可用于做什么进行描述。
- 可使用空行来组织代码，但不要滥用。在类中，可使用一个空行来分隔方法；而在模块中，可使用两个空行来分隔类。
- 需要同时导入标准库中的模块和你编写的模块时，先编写导入标准库模块的import 语句，再添加一个空行，然后编写导入你自己编写的模块的import 语句。

#### 文件和异常

将学习处理文件，让程序能够快速地分析大量数据；你将学习错误处理，避免程序在面对意外情形时崩溃；你将学习异常 ，它们是Python创建的特殊对象，用于管理程序运行时出现的错误；你还将学习模块json ，它让你能够保存用户数据，以免在程序停止运行后丢失。

##### 从文件中读取数据

要使用文本文件中的信息，首先需要将信息读取到内存中。为此，你可以一次性读取文件的全部内容，也可以以每次一行的方式逐步读取。

###### 读取整个文件

当前路径下有pi_digits.txt：

```python
#file_reader.py
with open('pi_digits.txt') as file_object:
	contents = file_object.read()
print(contents)
```

1. 函数open() 接受一个参数：要打开的文件的名称。Python在当前执行的文件所在的目录中查找指定的文件。
2. 函数open() 返回一个表示文件的对象。在这里，open('pi_digits.txt') 返回一个表示文件pi_digits.txt的对象，Python将该对象赋给file_object 供以后使用。
3. 关键字with 在不再需要访问文件后将其关闭。
4. 使用方法file_object.read() 读取这个文件的全部内容，并将其作为一个长长的字符串赋给变量contents 。

该输出唯一不同的地方是末尾多了一个空行（使用Sublime运行不会）。为何会多出这个空行呢？因为read() 到达文件末尾时返回一个空字符串，而将这个空字符串显示出来时就是一个空行。要删除多出来的空行，可在函数调用print() 中使用rstrip(); 

###### 文件路径

根据你组织文件的方式，有时可能要打开不在程序文件所属目录中的文件。

**相对文件路径**让Python到指定的位置去查找，而该位置是相对于当前运行的程序所在目录的:

```python
with open('text_files/filename.txt') as file_object:
```

> 注意 　显示文件路径时，Windows系统使用反斜杠（\ ）而不是斜杠（/ ），但在代码中依然可以使用斜杠。

还可以将文件在计算机中的准确位置告诉Python，这样就不用关心当前运行的程序存储在什么地方了。这称为**绝对文件路径** 。

```python
file_path = '/home/ehmatthes/other_files/text_files/_filename_.txt'
with open(file_path) as file_object:
```

###### 逐行读取

读取文件时，常常需要检查其中的每一行：可能要在文件中查找特定的信息，或者要以某种方式修改文件中的文本。

```python
filename = 'pi_digits.txt'
with open(filename) as file_object:
	for line in file_object:
		print(line)
```

但打印每一行时，发现空白行更多了； 因为在这个文件中，每行的末尾都有一个看不见的换行符，而函数调用print() 也会加上一个换行符，因此每行末尾都有两个换行符：一个来自文件，另一个来自函数调用print() 。要消除这些多余的空白行，可在函数调用print() 中使用rstrip()；

###### 创建一个包含文件各行内容的列表

使用关键字with 时，open() 返回的文件对象只在with 代码块内可用。如果要在with 代码块外访问文件的内容，可在with 代码块内将文件的各行存储在一个列表中，并在with 代码块外使用该列表：可以立即处理文件的各个部分，也可以推迟到程序后面再处理。

在with 代码块中将文件pi_digits.txt的各行存储在一个列表中，再在with 代码块外打印：

```python
filename = 'pi_digits.txt'
with open(filename) as file_object:
	lines = file_object.readlines()
for line in lines:
	print(line.rstrip())
```

###### 使用文件的内容

将文件读取到内存中后，就能以任何方式使用这些数据了。

```python
filename = 'pi_digits.txt'
with open(filename) as file_object:
	lines = file_object.readlines()

pi_string = ''
for line in lines:
	pi_string += line.rstrip()

print(pi_string)
print(len(pi_string))
```

##### 写入文件

保存数据的最简单的方式之一是将其写入文件中。通过将输出写入文件，即便关闭包含程序输出的终端窗口，这些输出也依然存在：可以在程序结束运行后查看这些输出，可以与别人分享输出文件，还可以编写程序来将这些输出读取到内存中并进行处理。

###### 写入空文件

调用open() 时需要提供另一个实参，告诉Python你要写入打开的文件。

```python
filename = 'programming.txt'
with open(filename, 'w') as file_object:
	file_object.write("I love programming.")
```

1. 第一个实参也是要打开的文件的名称。
2. 第二个实参（'w' ）告诉Python，要以写入模式 打开这个文件。打开文件时，可指定读取模式 （'r'）、写入模式 （'w' ）、附加模式 （'a' ）或读写模式 （'r+'）。如果省略了模式实参，Python将以默认的只读模式打开文件。
3. 如果要写入的文件不存在，函数open() 将自动创建它。然而，以写入模式（'w' ）打开文件时千万要小心，因为如果指定的文件已经存在，Python将在返回文件对象前清空该文件的内容。
4. 使用文件对象的方法write() 将一个字符串写入文件。

> 注意 　Python只能将字符串写入文本文件。要将数值数据存储到文本文件中，必须先使用函数str() 将其转换为字符串格式。

###### 写入多行

函数write() 不会在写入的文本末尾添加换行符；要让每个字符串都单独占一行，需要在方法调用write() 中包含换行符：

```python
filename = 'programming.txt'
with open(filename, 'w') as file_object:
    file_object.write("I love programming.\n")
    file_object.write("I love creating new games.\n")
```

###### 附加到文件

如果要给文件添加内容，而不是覆盖原有的内容，可以以附加模式打开文件。以附加模式（'a' ）打开文件时，Python不会在返回文件对象前清空文件的内容，而是将写入文件的行添加到文件末尾。如果指定的文件不存在，Python将为你创建一个空文件。

```python
filename = 'programming.txt'
with open(filename, 'a') as file_object:
	file_object.write("I also love finding meaning in large datasets.\n")
	file_object.write("I love creating apps that can run in a browser.\n")
```

#### 异常

Python使用称为异常 的特殊对象来管理程序执行期间发生的错误。每当发生让Python不知所措的错误时，它都会创建一个异常对象。如果你编写了处理该异常的代码，程序将继续运行；如果未对异常进行处理，程序将停止并显示traceback，其中包含有关异常的报告。

异常是使用try-except 代码块处理的。

##### 使用try-except 代码块

处理ZeroDivisionError 异常的try-except 代码块:

```python
try:
	print(5/0)
except ZeroDivisionError:
	print("You can't divide by zero!")
```

##### else 代码块

通过将可能引发错误的代码放在try-except 代码块中，可提高程序抵御错误的能力。错误是执行除法运算的代码行导致的，因此需要将它放到try-except 代码块中。这个示例还包含一个else 代码块。依赖try 代码块成功执行的代码都应放到else 代码块中：

```python
--snip--
while True:
    --snip--
    if second_number == 'q':
    	break
    try:
    	answer = int(first_number) / int(second_number)
    except ZeroDivisionError:
    	print("You can't divide by 0!")
    else:
    	print(answer)
```

`try-except-else` 代码块的工作原理大致如下。Python尝试执行try 代码块中的代码，只有可能引发异常的代码才需要放在try语句中。有时候，有一些仅在try 代码块成功执行时才需要运行的代码，这些代码应放在else 代码块中。except 代码块告诉Python，如果尝试运行try 代码块中的代码时引发了指定的异常该怎么办。

##### 处理FileNotFoundError 异常

使用文件时，一种常见的问题是找不到文件：查找的文件可能在其他地方，文件名可能不正确，或者这个文件根本就不存在。对于所有这些情形，都可使用try-except 代码块以直观的方式处理。

```python
filename = 'alice.txt'
try:
	with open(filename, encoding='utf-8') as f:
		contents = f.read()
except FileNotFoundError:
	print(f"Sorry, the file {filename} does not exist.")
```

##### 分析文本

```python
filename = 'alice.txt'
try:
	with open(filename, encoding='utf-8') as f:
		contents = f.read()
except FileNotFoundError:
	print(f"Sorry, the file {filename} does not exist.")
else:
    # 计算该文件大致包含多少个单词。
    words = contents.split()
    num_words = len(words)
    print(f"The file {filename} has about {num_words} words.")
```

##### 静默失败

有时候你希望程序在发生异常时保持静默，就像什么都没有发生一样继续运行。要让程序静默失败，可像通常那样编写try 代码块，但在except 代码块中明确地告诉Python什么都不要做。Python有一个pass 语句，可用于让Python在代码块中什么都不要做：

```python
def count_words(filename):
    """计算一个文件大致包含多少个单词。"""
    try:
    	--snip--
    except FileNotFoundError:
    	pass
    else:
    	--snip--

filenames = ['alice.txt', 'siddhartha.txt', 'moby_dick.txt', 'little_women.txt']
for filename in filenames:
	count_words(filename)
```

pass 语句还充当了占位符，提醒你在程序的某个地方什么都没有做，并且以后也许要在这里做些什么。

##### 存储数据

模块json 让你能够将简单的Python数据结构转储到文件中，并在程序再次运行时加载该文件中的数据。你还可以使用json 在Python程序之间分享数据。

###### 使用json.dump() 和json.load()

保存：

```python
import json

numbers = [2, 3, 5, 7, 11, 13]
filename = 'numbers.json'
with open(filename, 'w') as
	json.dump(numbers, f)
```

读取：

```python
import json
filename = 'numbers.json'
with open(filename) as f:
	numbers = json.load(f)
print(numbers)
```

###### 重构

你经常会遇到这样的情况：代码能够正确地运行，但通过将其划分为一系列完成具体工作的函数，还可以改进。这样的过程称为重构。重构让代码更清晰、更易于理解、更容易扩展。

#### `__name__`属性

与Java、C、C++等几种语言不同的是，Python是一种解释型脚本语言，在执行之前不同要将所有代码先编译成中间代码，Python程序运行时是从模块顶行开始，逐行进行翻译执行，所以，最顶层（没有被缩进）的代码都会被执行，所以Python中并不需要一个统一的main()作为程序的入口。

Python解释器在导入模块时，会将模块中没有缩进的代码全部执行一遍（模块就是一个独立的Python文件）。开发人员通常会在模块下方增加一些测试代码，为了避免这些测试代码在模块被导入后执行，可以利用__name__属性。

例子：

```python
# const.py
PI = 3.14
 
def train():
    print("PI:", PI)
 
train()
```

```python
# area.py
from const import PI
 
def calc_round_area(radius):
    return PI * (radius ** 2)
 
def calculate():
    print("round area: ", calc_round_area(2))
 
calculate()
```

现在运行area.py文件，看结果：

可以看见，const.py中的train()也被运行了，实际上我们是不希望它被运行，只是想把const.py中 ***\*PI\**** 变量导入到 area.py。现在把 const.py 改一下：

```python
# const.py
PI = 3.14
def train():
    print("PI:", PI)
 
if __name__ == "__main__":
    train()
```

这样就是其它模块使用import到const模块时， 不会被运行train()，仅在运行const.py时，才会到跑train()；

name__ 是一个系统变量（前后加了双下划线为系统变量，普通变量不能如此命名）

1、如果当前模块为主模块（即调用其他模块的模块），那么此模块名字即为'__main__'

2、如果当前模块被import，那么此模块名字即为文件名字（不加后边的.py）





### Python模块

#### os模块

os模块是Python中整理文件和目录最为常用的模块，该模块提供了非常丰富的方法用来处理文件和目录。

![](https://img2020.cnblogs.com/blog/2630138/202112/2630138-20211220085035487-217047869.png)

更多请参考：https://www.runoob.com/python/os-file-methods.html

#### re模块

在python中使用`re模块`来处理正则表达式，但是它有两种使用方法。

- 直接使用`re模块`的方法，如re.match()

- 事先编译一个正则表达式，通过返回的`regex对象`来调用，如regex.match()
  
  ```
  使用re.compile(pattern,flag=0) 生成`regex对象`:
  ```

  pattern：需要编译的正则表达式
  flag：编译标志位，有re.M、re.S、re.I、re.X可选，\**默认是re.S，\**如果需要多个可用‘|’符号开启，比如‘re.I|re.M’;

| **代码**                 | **说明**                                                     | **Python**          |
| ------------------------ | ------------------------------------------------------------ | ------------------- |
| IgnoreCase               | 匹配时忽略大小写                                             | **re.I**            |
| SingleLine               | 单行模式：整个字符串作为一个处理单独，‘.’可以匹配所有字符，包括换行符 | **re.S**(re.DOTALL) |
| MultiLine                | 多行模式：每一行作为处理单位                                 | **re.M**            |
| IgnorePatternWhitespache | 忽略表达式中的空白字符                                       | **re.X**            |

##### 相关方法

###### match方法

re.match(pattern,string,flags=0) ->match对象

regex对象.match(string[,start[,stop]]) ->match对象

re.match一定是**从整个字符串的开头进行匹配**，而regex对象.match可以指定开始和结束位置，如果找到第一个匹配则立即返回一个match对象，不会再关心后面的内容，如果找不到匹配内容，则返回一个None。

匹配的结果可以在match对象的方法中获取，有如下方法：

1. group([n[,m]])：如果为空或者‘0’，则表示获取整个正则表达式匹配的内容，如果为n，则明确表示获取第n个分组的内容，如果是n,m，则表示获取第n个到m个分组的内容。
2. groups()：返回获取所有分组的内容组成的一个元组
3. group('name')：返回指定name的分组内容
4.  groupdict()：返回所有命名分组的内容组成的一个字典
5.  start()：返回匹配开始的位置
6. end()：返回匹配结束的位置

> 注：match不管是多行还是单行，依次是从字符串的开头开始匹配，而且找到第一个就不找了

```
​```python
>>> import re
 
>>> s = """bottle\nbag\nbig\napple"""
 
#使用re.match一定是从字符串的开头进行匹配，不管是单行还是多行模式，虽然字符串‘bag’在里面，但它不是在字符串的开头，所以匹配不到
>>> re.match('bag',s,re.M)
None
>>> re.match('bag',s,re.S)
None
 
#使用预先编译的方法，match可以指定开始位置
>>> regex = re.compile('bag')
>>> regex.match(s,7).group()
'bag'
 
#match不管是多行还是单行，依次是从字符串的开头开始匹配，而且找到第一个就不找了
>>> re.match('b',s,re.M).group()
'b'
>>> re.match('b',s,re.S).group()
'b'
 
#普通分组和group的使用方法
>>> re.match('(b)o(t+)l(e)',s,re.M).group()
'bottle'
>>> re.match('(b)o(t+)l(e)',s,re.M).group(2)
'tt'
>>> re.match('(b)o(t+)l(e)',s,re.M).group(2,3)
('tt', 'e')
>>> re.match('(b)o(t+)l(e)',s,re.M).groups()
('b', 'tt', 'e')
 
#命名分组，python的命名分组需要使用(?P<name>)格式
>>> re.match('(b)o(t+)l(?P<tag>e)',s,re.M).group(0)
'bottle'
>>> re.match('(b)o(t+)l(?P<tag>e)',s,re.M).group('tag')
'e'
>>> re.match('(b)o(t+)l(?P<tag>e)',s,re.M).groupdict()
{'tag': 'e'}
```

###### search方法

```python
re.search(pattern,string,flags=0) -> match对象

regex对象.search(string[,start[,stop]] -> match对象
```

使用方法和match类似，只是search是从头开始搜索匹配内容，**如果不匹配会继续往后搜索，直到搜索到第一个匹配为止**返回match对象，如果搜索不到，则返回None

```python
>>> import re
 
>>> s = """bottle\nbag\nbig\napple"""
 
#搜索到第一个字符‘le’之后，立即返回，不在匹配后面的内容
>>> re.search('le',s).group()
'le'
 
#search和match一样，不管是单行还是多行模式，找到就返回
>>> re.search('^a',s,re.S).group()
AttributeError: 'NoneType' object has no attribute 'group'
 
>>> re.search('^a',s,re.M).group()
'a'
```

###### fullmatch方法

```
re.fullmatch(pattern,string,flags=0) -> match对象

regex对象.fullmatch(string[,start[,stop]] -> match对象
```

python 3中新增的方法，使用方法和match类似，只是fullmatch是需要**正则表达式和字符串要完全匹配**，如果匹配到了则返回match对象，如果匹配不到，则返回None

```python
>>> s = """bottle\nbag\nbig\napple"""

>>> re.fullmatch('bottle',s,re.M)
None
>>> re.fullmatch('bottle',s,re.S)
None
 
>>> regex = re.compile('bottle')
>>> regex.fullmatch(s,0,6).group()
'bottle'
```

###### findall方法

```python
re.findall(pattern,string,flags=0) -> List

regex对象.findall(string[,start[,stop]] -> List
```

使用方法和match类似，findall会找到**所有满足正则表达式匹配的内容**，然后返回一个列表

```python
>>> s = """bottle\nbag\nbig\napple"""

#找到匹配内容
>>> re.findall('le',s)
['le', 'le']
 
#未找到匹配内容
>>> re.findall('lea',s)
[]
 
#单行模式，查找‘a’字母开头的
>>> re.findall('^a',s,re.S)
[]
#多行模式，查找‘a’字母开头的
>>> re.findall('^a',s,re.M)
['a']
```

###### finditer方法  

```
re.finditer(pattern,string,flags=0) -> Iterator

regex对象.finditer(string[,start[,stop]] -> Iterator
```

使用方法和功能跟findall类似，只是返回的结果不一样。finditer会找到所有满足正则表达式匹配的内容，然后返回一个迭代器，迭代器的每一个元素都是match对象

```python
>>> s = """bottle\nbag\nbig\napple"""

>>> iter = re.finditer('^a',s,re.M)
>>> next(iter).group()
'a'
 
>>> next(iter).group()
StopIteration:
```

###### sub方法

```
re.sub(pattern,replacement,string,count=0,flags=0) -> Str

regex对象.sub(replacement,string,count=0) -> Str
```

**使用replacement的内容替换，pattern匹配到的内容**

count：默认替换所有匹配到的内容，count可以指定替换次数

```python
>>> s = """bottle\nbag\nbig\napple"""

#默认所有匹配到的内容都将被替换
>>> re.sub('a','z',s)
'bottle\nbzg\nbig\nzpple'
 
#指定替换次数
>>> re.sub('a','z',s,count=1)
'bottle\nbzg\nbig\napple'
```

###### subn方法

```
re.subn(pattern,replacement,string,count=0,flags=0) -> Tuple

regex对象.subn(replacement,string,count=0) -> Tuple
```

使用方法和sub一样，只是返回的是一个由被替换后的新字符串和替换次数组成的二元组；

```python
>>> s = """bottle\nbag\nbig\napple"""


>>> re.subn('a','z',s)
('bottle\nbzg\nbig\nzpple', 2)
 
>>> re.subn('a','z',s,count=1)
('bottle\nbzg\nbig\napple', 1)
```

###### split方法

```
 re.split(pattern,string,maxsplit,flags=0) -> List

regex对象.split(string,maxsplit) -> List
```

按照**指定正则表达式对字符串进行分割**

```python
>>> s = """bottle\nbag\nbig\napple"""

#单行模式切割
>>> re.split('\s+',s,re.S)
['bottle', 'bag', 'big', 'apple']
 
#多行模式切割
>>> re.split('\s+',s,re.M)
['bottle', 'bag', 'big', 'apple']
```

#### mmap模块

正常模式下，用 r+ 模式打开文件，可以随意读写，但是要特别小心。readline是否能够使用，要看这个文件每行都多长，如果没有换行，就不能用，就算知道每行的大小，也要带个参数N来控制最大读取数量。readlines是肯定不能用的，就算带参数，也可能直接卡死！read(N)没问题，主要控制是N的大小。总之，传统读写文件的方式可以用，但是不够方便。速度也是个问题，传统的缓存IO方式，涉及到OS内核态的内存和进程虚拟空间内存的内容交换，对于超大文件而言，这种交换会浪费大量的CPU时间和内存。

mmap是另一个方式！它省掉了内核态和用户态页copy这个动作（两态间copy），直接将用户态的虚拟地址与内核态空间进行mapping，进程直接读取内核空间，速度提高了，内存占用也少了。

> 总结来说，常规文件操作为了提高读写效率和保护磁盘，使用了页缓存机制。这样造成读文件时需要先将文件页从磁盘拷贝到页缓存中，由于页缓存处在内核空间，不能被用户进程直接寻址，所以还需要将页缓存中数据页再次拷贝到内存对应的用户空间中。这样，通过了两次数据拷贝过程，才能完成进程对文件内容的获取任务。写操作也是一样，待写入的buffer在内核空间不能直接访问，必须要先拷贝至内核空间对应的主存，再写回磁盘中（延迟写回），也是需要两次数据拷贝。
>
> 而使用mmap操作文件中，创建新的虚拟内存区域和建立文件磁盘地址和虚拟内存区域映射这两步，没有任何文件拷贝操作。而之后访问数据时发现内存中并无数据而发起的缺页异常过程，可以通过已经建立好的映射关系，只使用一次数据拷贝，就从磁盘中将数据传入内存的用户空间中，供进程使用。
>
> **总而言之，常规文件操作需要从磁盘到页缓存再到用户主存的两次数据拷贝。而mmap操控文件，只需要从磁盘到用户主存的一次数据拷贝过程。**说白了，mmap的关键点是实现了用户空间和内核空间的数据直接交互而省去了空间不同数据不通的繁琐过程。因此在某些场景下，mmap效率更高。

**什么时候用mmap？**

1. 将一个普通文件映射到内存中，通常在需要对文件进行**频繁读写**时使用，这样用内存映射读写取代I/O缓存读写，以获得较高的性能；
2. 将特殊文件进行匿名内存映射，可以为关联进程提供共享内存空间；
3. 为无关联的进程提供共享内存空间，一般也是将一个普通文件映射到内存中。

##### 创建 `mmap`对象

```
m=mmap.mmap(fileno, length[, flags[, prot[, access[, offset]]]])
```

**fileno：** 文件描述符，可以是file对象的fileno()方法，或者来自os.open()，在调用mmap()之前打开文件，不再需要文件时要关闭。

```
os.O_RDONLY   以只读的方式打开 Read only
os.O_WRONLY   以只写的方式打开 Write only
os.O_RDWR     以读写的方式打开 Read and write
os.O_APPEND  以追加的方式打开  
os.O_CREAT   创建并打开一个新文件
os.O_EXCL     os.O_CREAT| os.O_EXCL 如果指定的文件存在，返回错误
os.O_TRUNC    打开一个文件并截断它的长度为零（必须有写权限）
os.O_BINARY          以二进制模式打开文件（不转换）
os.O_NOINHERIT        阻止创建一个共享的文件描述符
os.O_SHORT_LIVED
os.O_TEMPORARY        与O_CREAT一起创建临时文件
os.O_RANDOM         缓存优化,但不限制从磁盘中随机存取
os.O_SEQUENTIAL   缓存优化,但不限制从磁盘中序列存取
os.O_TEXT           以文本的模式打开文件（转换）
```

**length：**要映射文件部分的大小（以字节为单位），这个值为0，则映射整个文件，如果大小大于文件当前大小，则扩展这个文件。

**flags**：MAP_PRIVATE：这段内存映射只有本进程可用；mmap.MAP_SHARED：将内存映射和其他进程共享，所有映射了同一文件的进程，都能够看到其中一个所做的更改；
**prot：**mmap.PROT_READ, mmap.PROT_WRITE 和 mmap.PROT_WRITE | mmap.PROT_READ。最后一者的含义是同时可读可写。

**access：**在mmap中有可选参数access的值有
	ACCESS_READ：读访问。
	ACCESS_WRITE：写访问，默认。
	ACCESS_COPY：拷贝访问，不会把更改写入到文件，使用flush把更改写到文件。

##### 相关方法

```python
m.close()
#关闭 m 对应的文件；

m.find(str, start=0)
#从 start 下标开始，在 m 中从左往右寻找子串 str 最早出现的下标；
m.flush([offset, n])
#把 m 中从offset开始的n个字节刷到对应的文件中；

m.move(dstoff, srcoff, n)
#等于 m[dstoff:dstoff+n] = m[srcoff:srcoff+n]，把从 srcoff 开始的 n 个字节复制到从 dstoff 开始的n个字节，可能会覆盖重叠的部分。

m.read(n)
#返回一个字符串，从 m 对应的文件中最多读取 n 个字节，将会把 m 对应文件的位置指针向后移动；

m.read_byte() 
#返回一个1字节长的字符串，从 m 对应的文件中读1个字节，要是已经到了EOF还调用 read_byte()，则抛出异常 ValueError；

m.readline()
#返回一个字符串，从 m 对应文件的当前位置到下一个'\n'，当调用 readline() 时文件位于 EOF，则返回空字符串；

m.resize(n)
#把 m 的长度改为 n，m 的长度和 m 对应文件的长度是独立的；

m.seek(pos, how=0)
#同 file 对象的 seek 操作，改变 m 对应的文件的当前位置；

m.size()
#返回 m 对应文件的长度（不是 m 对象的长度len(m)）；

m.tell()
#返回 m 对应文件的当前位置；

m.write(str)
#把 str 写到 m 对应文件的当前位置，如果从 m 对应文件的当前位置到 m 结尾剩余的空间不足len(str)，则抛出 ValueError；

m.write_byte(byte)
#把1个字节（对应一个字符）写到 m 对应文件的当前位置，实际上 m.write_byte(ch) 等于 m.write(ch)。如果 m 对应文件的当前位置在 m 的结尾，也就是 m 对应文件的当前位置到 m 结尾剩余的空间不足1个字节，write() 抛出异常ValueError，而 write_byte() 什么都不做。
```

