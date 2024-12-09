# 基础

[https://github.com/jackfrued/Python-100-Days/blob/master/Day16-20/16-20.Python语言进阶.md](https://github.com/jackfrued/Python-100-Days/blob/master/Day16-20/16-20.Python%E8%AF%AD%E8%A8%80%E8%BF%9B%E9%98%B6.md)

## 变量

- 整型：可以处理任意大小的整数，所以只有`int`类型。支持二进制（如`0b100`，即4）、八进制（如`0o100`，即64）、十进制（`100`）和十六进制（`0x100`，即256）的表示法。
- 浮点型：即小数，支持数学写法（如`123.456`）也支持科学计数法（如`1.23456e2`）
- 字符串类型：单引号或双引号括起来的任意文本。还有原始字符串，字节字符串，Unicode字符串表示法
- 布尔型：只有`True`、`False`两种值
- 复数型：形如`3+5j`

Python 内置了函数可以获取变量类型或者转换变量类型

- `type()`：获取变量类型
- `int()`：将一个数值或字符串转换成整数，可以指定进制。
- `float()`：将一个字符串转换成浮点数。
- `str()`：将指定的对象转换成字符串形式，可以指定编码。
- `chr()`：将整数转换成该编码对应的字符串（一个字符）。
- `ord()`：将字符串（一个字符）转换成对应的编码（整数）。

## 函数

### 可变参数

```python
# 在参数名前面的*表示args是一个可变参数
def add(*args):
    total = 0
    for val in args:
        total += val
    return total
```

### 变量作用域

Python查找变量时按照“局部作用域”、“嵌套作用域”、“全局作用域”和“内置作用域”的顺序进行搜索。所谓的“内置作用域”就是Python内置的那些标识符，如`input`、`print`、`int`等都属于内置作用域。

```python
def foo():
    b = 'hello'

    # Python中可以在函数内部再定义函数
    def bar():
        c = True
        print(a)
        print(b)
        print(c)

    bar()
    # print(c)  # NameError: name 'c' is not defined

if __name__ == '__main__':
    a = 100
    # print(b)  # NameError: name 'b' is not defined
    foo()
```

代码能够顺利的执行并且打印出100、hello和True。

- 变量`a`是一个全局变量（global variable），属于全局作用域，因为它没有定义在任何一个函数中。
- 变量`b`是一个定义在函数中的局部变量（local variable），属于局部作用域，在`foo`函数的外部并不能访问到它；但对于`foo`函数内部的`bar`函数来说，变量`b`属于嵌套作用域，在`bar`函数中我们是可以访问到它的。
- 变量`c`属于局部作用域，在`bar`函数之外是无法访问的。

```python
def foo():
    a = 200
    print(a)  # 200

if __name__ == '__main__':
    a = 100
    foo()
    print(a)  # 100
```

上述代码并不能通过函数调用修改全局变量`a`的值。

`foo`函数优先搜索局部变量`a`。如果希望在`foo`函数中修改全局变量`a`代码如下：

```python
def foo():
    global a # 表明a来自于全局作用域
    a = 200
    print(a)  # 200

if __name__ == '__main__':
    a = 100
    foo()
    print(a)  # 200
```

`global`表明`foo`中的变量`a`来自于全局作用域。如果全局作用域中没有`a`，那么下面一行的代码就会定义变量`a`并将其**置于全局作用域**

同理，如果希望函数内部的函数能够修改嵌套作用域中的变量，可以使用`nonlocal`关键字来指示变量来自于嵌套作用域

## 字符串

```python
str1 = 'hello, world!'
# 通过内置函数len计算字符串的长度
print(len(str1)) # 13
# 获得字符串首字母大写的拷贝
print(str1.capitalize()) # Hello, world!
# 获得字符串每个单词首字母大写的拷贝
print(str1.title()) # Hello, World!
# 获得字符串变大写后的拷贝
print(str1.upper()) # HELLO, WORLD!
# 从字符串中查找子串所在位置
print(str1.find('or')) # 8
print(str1.find('shit')) # -1
# 与find类似但找不到子串时会引发异常
# print(str1.index('or'))
# print(str1.index('shit'))
# 检查字符串是否以指定的字符串开头
print(str1.startswith('He')) # False
print(str1.startswith('hel')) # True
# 检查字符串是否以指定的字符串结尾
print(str1.endswith('!')) # True
# 将字符串以指定的宽度居中并在两侧填充指定的字符
print(str1.center(50, '*'))
# 将字符串以指定的宽度靠右放置左侧填充指定的字符
print(str1.rjust(50, ' '))
str2 = 'abc123456'
# 检查字符串是否由数字构成
print(str2.isdigit())  # False
# 检查字符串是否以字母构成
print(str2.isalpha())  # False
# 检查字符串是否以数字和字母构成
print(str2.isalnum())  # True
str3 = '  jackfrued@126.com '
print(str3)
# 获得字符串修剪左右两侧空格之后的拷贝
print(str3.strip())
```

## 列表

创建遍历列表

```python
list1 = [1, 3, 5, 7, 100]
print(list1) # [1, 3, 5, 7, 100]
# 乘号表示列表元素的重复
list2 = ['hello'] * 3
print(list2) # ['hello', 'hello', 'hello']
# 计算列表长度(元素个数)
print(len(list1)) # 5
# 下标(索引)运算
print(list1[0]) # 1
print(list1[4]) # 100
# print(list1[5])  # IndexError: list index out of range
print(list1[-1]) # 100
print(list1[-3]) # 5
list1[2] = 300
print(list1) # [1, 3, 300, 7, 100]
# 通过循环用下标遍历列表元素
for index in range(len(list1)):
    print(list1[index])
# 通过for循环遍历列表元素
for elem in list1:
    print(elem)
# 通过enumerate函数处理列表之后再遍历可以同时获得元素索引和值
for index, elem in enumerate(list1):
    print(index, elem)
```

添加移除元素

```python
list1 = [1, 3, 5, 7, 100]
# 添加元素
list1.append(200)
list1.insert(1, 400)
# 合并两个列表
# list1.extend([1000, 2000])
list1 += [1000, 2000]
print(list1) # [1, 400, 3, 5, 7, 100, 200, 1000, 2000]
print(len(list1)) # 9
# 先通过成员运算判断元素是否在列表中，如果存在就删除该元素
if 3 in list1:
	list1.remove(3)
if 1234 in list1:
    list1.remove(1234)
print(list1) # [1, 400, 5, 7, 100, 200, 1000, 2000]
# 从指定的位置删除元素
list1.pop(0)
list1.pop(len(list1) - 1)
print(list1) # [400, 5, 7, 100, 200, 1000]
# 清空列表元素
list1.clear()
print(list1) # []
```

切片、排序操作

切片索引是形如`[开始索引:结束索引:跨度]`的语法，通过指定**开始索引**（默认值无穷小）、**结束索引**（默认值无穷大）和**跨度**（默认值1），从数组中取出指定部分的元素并构成新的数组。因为开始索引、结束索引和步长都有默认值，所以它们都可以省略，如果不指定步长，第二个冒号也可以省略。

```python

fruits = ['grape', 'apple', 'strawberry', 'waxberry']
fruits += ['pitaya', 'pear', 'mango']
# 列表切片
fruits2 = fruits[1:4]
print(fruits2) # apple strawberry waxberry
# 可以通过完整切片操作来复制列表
fruits3 = fruits[:]
print(fruits3) # ['grape', 'apple', 'strawberry', 'waxberry', 'pitaya', 'pear', 'mango']
fruits4 = fruits[-3:-1]
print(fruits4) # ['pitaya', 'pear']
# 可以通过反向切片操作来获得倒转后的列表的拷贝
fruits5 = fruits[::-1]
print(fruits5) # ['mango', 'pear', 'pitaya', 'waxberry', 'strawberry', 'apple', 'grape']

# ====== 排序 ======
list1 = ['orange', 'apple', 'zoo', 'internationalization', 'blueberry']
list2 = sorted(list1)
# sorted函数返回列表排序后的拷贝不会修改传入的列表
# 函数的设计就应该像sorted函数一样尽可能不产生副作用
list3 = sorted(list1, reverse=True)
# 通过key关键字参数指定根据字符串长度进行排序而不是默认的字母表顺序
list4 = sorted(list1, key=len)
print(list1)
print(list2)
print(list3)
print(list4)
# 给列表对象发出排序消息直接在列表对象上进行排序
list1.sort(reverse=True)
print(list1)
```

## 生成式和生成器

```python
f = [x for x in range(1, 10)]
print(f)
f = [x + y for x in 'ABCDE' for y in '1234567']
print(f)
# 用列表的生成表达式语法创建列表容器
# 用这种语法创建列表之后元素已经准备就绪所以需要耗费较多的内存空间
f = [x ** 2 for x in range(1, 1000)]
print(sys.getsizeof(f))  # 查看对象占用内存的字节数
print(f)
# 请注意下面的代码创建的不是一个列表而是一个生成器对象
# 通过生成器可以获取到数据但它不占用额外的空间存储数据
# 每次需要数据的时候就通过内部的运算得到数据(需要花费额外的时间)
f = (x ** 2 for x in range(1, 1000))
print(sys.getsizeof(f))  # 相比生成式生成器不占用存储数据的空间
print(f)
for val in f:
    print(val)

# 还可以通过yield关键字将一个普通函数改造成生成器函数
# 斐波拉切数列的生成器
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
        yield a

for val in fib(20):
    print(val)
```

## 元组

元组的元素不能修改

```python
# 定义元组
t = ('骆昊', 38, True, '四川成都')
print(t)
# 获取元组中的元素
print(t[0])
print(t[3])
# 遍历元组中的值
for member in t:
    print(member)
# 重新给元组赋值
# t[0] = '王大锤'  # TypeError
# 变量t重新引用了新的元组原来的元组将被垃圾回收
t = ('王大锤', 20, True, '云南昆明')
print(t)
# 将元组转换成列表
person = list(t)
print(person)
# 列表是可以修改它的元素的
person[0] = '李小龙'
person[1] = 25
print(person)
# 将列表转换成元组
fruits_list = ['apple', 'banana', 'orange']
fruits_tuple = tuple(fruits_list)
print(fruits_tuple)
```

1. 如果不需要对元素进行添加、删除、修改的时候，可以考虑使用元组，多线程环境更喜欢使用不变对象，方便维护。当然如果一个方法要返回多个值，使用元组也是不错的选择。
2. 元组在创建时间和占用的空间上面都优于列表。我们可以使用sys模块的getsizeof函数来检查存储同样的元素的元组和列表各自占用了多少

## 集合

不允许有重复元素，可以进行交集、并集、差集等运算

```python
# 创建集合的字面量语法
set1 = {1, 2, 3, 3, 3, 2}
print(set1)
print('Length =', len(set1))
# 创建集合的构造器语法(面向对象部分会进行详细讲解)
set2 = set(range(1, 10))
set3 = set((1, 2, 3, 3, 2, 1))
print(set2, set3)
# 创建集合的推导式语法(推导式也可以用于推导集合)
set4 = {num for num in range(1, 100) if num % 3 == 0 or num % 5 == 0}
print(set4)

# 添加/删除元素
set1.add(4)
set1.add(5)
set2.update([11, 12])
set2.discard(5)
if 4 in set2:
    set2.remove(4)
print(set1, set2)
print(set3.pop())
print(set3)

# 集合的交集、并集、差集、对称差运算
print(set1 & set2)
# print(set1.intersection(set2))
print(set1 | set2)
# print(set1.union(set2))
print(set1 - set2)
# print(set1.difference(set2))
print(set1 ^ set2)
# print(set1.symmetric_difference(set2))
# 判断子集和超集
print(set2 <= set1)
# print(set2.issubset(set1))
print(set3 <= set1)
# print(set3.issubset(set1))
print(set1 >= set2)
# print(set1.issuperset(set2))
print(set1 >= set3)
# print(set1.issuperset(set3))
```

## 字典

```python
# 创建字典的字面量语法
scores = {'骆昊': 95, '白元芳': 78, '狄仁杰': 82}
print(scores)
# 创建字典的构造器语法
items1 = dict(one=1, two=2, three=3, four=4)
# 通过zip函数将两个序列压成字典
items2 = dict(zip(['a', 'b', 'c'], '123'))
# 创建字典的推导式语法
items3 = {num: num ** 2 for num in range(1, 10)}
print(items1, items2, items3)
# 通过键可以获取字典中对应的值
print(scores['骆昊'])
print(scores['狄仁杰'])
# 对字典中所有键值对进行遍历
for key in scores:
    print(f'{key}: {scores[key]}')
# 更新字典中的元素
scores['白元芳'] = 65
scores['诸葛王朗'] = 71
scores.update(冷面=67, 方启鹤=85)
print(scores)
if '武则天' in scores:
    print(scores['武则天'])
print(scores.get('武则天'))
# get方法也是通过键获取对应的值但是可以设置默认值
print(scores.get('武则天', 60))
# 删除字典中的元素
print(scores.popitem())
print(scores.popitem())
print(scores.pop('骆昊', 100))
# 清空字典
scores.clear()
print(scores)
```

## 面向对象

```python
class Student(object):

    # __init__是一个特殊方法用于在创建对象时进行初始化操作
    # 通过这个方法我们可以为学生对象绑定name和age两个属性
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def study(self, course_name):
        print('%s正在学习%s.' % (self.name, course_name))

    def watch_movie(self):
        if self.age < 18:
            print('%s只能观看《熊出没》.' % self.name)
        else:
            print('%s正在观看岛国爱情大电影.' % self.name)

def main():
    # 创建学生对象并指定姓名和年龄
    stu1 = Student('骆昊', 38)
    # 给对象发study消息
    stu1.study('Python程序设计')
    # 给对象发watch_av消息
    stu1.watch_movie()
    stu2 = Student('王大锤', 15)
    stu2.study('思想品德')
    stu2.watch_movie()

if __name__ == '__main__':
    main()
```

在 python 中属性和方法的访问权限只有公开的和私有的两种。如果希望属性是私有的，在给属性命名时可以用两个下划线作为开头。

python 并没有从语法上严格保证私有属性或方法的私密性，它只是给私有的属性和方法换了一个名字来妨碍对它们的访问，事实上如果你知道更换名字的规则仍然可以访问到它们。

```python
class Test:

    def __init__(self, foo):
        self.__foo = foo

    def __bar(self):
        print(self.__foo)
        print('__bar')

def main():
    test = Test('hello')
    # AttributeError: 'Test' object has no attribute '__bar'
    test.__bar()
    # AttributeError: 'Test' object has no attribute '__foo'
    print(test.__foo)

if __name__ == "__main__":
    main()

# --------------------------------------------------------------

class Test:

    def __init__(self, foo):
        self.__foo = foo

    def __bar(self):
        print(self.__foo)
        print('__bar')

def main():
    test = Test('hello')
    test._Test__bar()   # 类名以单下划线开头
    print(test._Test__foo)

if __name__ == "__main__":
    main()
```

实际开发中，并不建议将属性设置为私有的，因为这会导致子类无法访问（后面会讲到）。所以大多数Python程序员会遵循一种命名惯例就是让属性名以**单下划线**开头来表示属性是受保护的，本类之外的代码在访问这样的属性时应该要保持慎重。这种做法并不是语法上的规则，单下划线开头的属性和方法外界仍然是可以访问的，所以更多的时候它是一种暗示或隐喻。

类的封装，可以使用`@property`包装器包装`getter`和`setter`方法，使得对属性的访问既安全又方便

```python
class Person(object):

    def __init__(self, name, age):
        self._name = name
        self._age = age

    # 访问器 - getter方法
    @property
    def name(self):
        return self._name

    # 访问器 - getter方法
    @property
    def age(self):
        return self._age

    # 修改器 - setter方法
    @age.setter
    def age(self, age):
        self._age = age

    def play(self):
        if self._age <= 16:
            print('%s正在玩飞行棋.' % self._name)
        else:
            print('%s正在玩斗地主.' % self._name)

def main():
    person = Person('王大锤', 12)
    person.play()
    person.age = 22
    person.play()
    # person.name = '白元芳'  # AttributeError: can't set attribute

if __name__ == '__main__':
    main()
```

python 是动态语言，动态语言允许在程序运行时给对象动态的绑定新的属性和方法，也可以对已经绑定的属性和方法进行解绑定。如果想要限定类对象绑定的属性，可通过`__slot__`变量进行限定。

```python
class Person(object):

    # 限定Person对象只能绑定_name, _age和_gender属性
    __slots__ = ('_name', '_age', '_gender')

    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        return self._name

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, age):
        self._age = age

    def play(self):
        if self._age <= 16:
            print('%s正在玩飞行棋.' % self._name)
        else:
            print('%s正在玩斗地主.' % self._name)

def main():
    person = Person('王大锤', 22)
    person.play()
    person._gender = '男'
    # AttributeError: 'Person' object has no attribute '_is_gay'
    # person._is_gay = True
```

静态方法 & 类方法

静态方法中无法调用任何类属性和类方法，可以调用其它静态方法

一般使用类方法来创建工厂方法。工厂方法为不同的用例返回类对象(类似于重载构造函数)

```python
class Foo(object):

    a = 1
    def __init__(self):
        self._x = 2

    """类三种方法语法形式"""

    def instance_method(self):  # 可访问 a, _x
        print("是类{}的实例方法，只能被实例对象调用".format(Foo))

    @staticmethod
    def static_method():      # 不可访问 a, _x
        print("是静态方法")

    @classmethod
    def class_method(cls):   # 可访问 a, 不可访问_x
        print("是类方法")

foo = Foo()
foo.instance_method()
foo.static_method()
foo.class_method()
print('----------------')
Foo.static_method()
Foo.class_method()
```

- 实例方法：可以获取类属性、构造函数定义的变量，属于 method 类型。只能通过实例化调用。
- 静态方法：不能获取类属性、构造函数定义的变量，属于 function 类型。两种调用方式：类.方法名 ，实例化调用。
- 类方法 ：可以获取类属性，不能获取构造函数定义的变量，属于 method 类型。两种调用方式：类.方法名 ，实例化调用（不推荐）。

继承和多态

通过`abc`模块的`ABCMeta`元类和`abstractmethod`包装器来达到抽象类的效果

```python
from abc import ABCMeta, abstractmethod

class Pet(object, metaclass=ABCMeta):
    """宠物"""

    def __init__(self, nickname):
        self._nickname = nickname

    @abstractmethod
    def make_voice(self):
        """发出声音"""
        pass

class Dog(Pet):
    """狗"""

    def make_voice(self):
        print('%s: 汪汪汪...' % self._nickname)

class Cat(Pet):
    """猫"""

    def make_voice(self):
        print('%s: 喵...喵...' % self._nickname)

def main():
    pets = [Dog('旺财'), Cat('凯蒂'), Dog('大黄')]
    for pet in pets:
        pet.make_voice()

if __name__ == '__main__':
    main()

# RTTI
def main():
    emps = [
        Manager('刘备'), Programmer('诸葛亮'),
        Manager('曹操'), Salesman('荀彧'),
        Salesman('吕布'), Programmer('张辽'),
        Programmer('赵云')
    ]
    for emp in emps:
        if isinstance(emp, Programmer):
            emp.working_hour = int(input('请输入%s本月工作时间: ' % emp.name))
        elif isinstance(emp, Salesman):
            emp.sales = float(input('请输入%s本月销售额: ' % emp.name))
        # 同样是接收get_salary这个消息但是不同的员工表现出了不同的行为(多态)
        print('%s本月工资为: ￥%s元' %
              (emp.name, emp.get_salary()))

```

魔法方法

## 文件读写

| 操作模式 | 具体含义 |
| --- | --- |
| `'r'` | 读取 （默认） |
| `'w'` | 写入（会先截断之前的内容） |
| `'x'` | 写入，如果文件已经存在会产生异常 |
| `'a'` | 追加，将内容写入到已有文件的末尾 |
| `'b'` | 二进制模式 |
| `'t'` | 文本模式（默认） |
| `'+'` | 更新（既可以读又可以写） |

### 读写文本文件

通过`encoding`参数指定编码（如果不指定默认为None，那么在读取文件时使用的是操作系统默认的编码），如果不能保证保存文件时使用的编码方式与encoding参数指定的编码方式是一致的，那么就可能因无法解码字符而导致读取失败

```python
f = open('test.txt', 'r', encoding='utf-8')
print(f.read())
f.close()
```

如果`open`函数指定的文件并不存在或者无法打开，那么将引发异常状况导致程序崩溃。可以使用异常机制对可能在运行时发生状况的代码进行处理

```python
def main():
    f = None
    try:
        f = open('致橡树.txt', 'r', encoding='utf-8')
        print(f.read())
    except FileNotFoundError:
        print('无法打开指定的文件!')
    except LookupError:
        print('指定了未知的编码!')
    except UnicodeDecodeError:
        print('读取文件时解码错误!')
    finally:
        if f:
            f.close()

if __name__ == '__main__':
    main()
```

finally 代码块一定会执行，保证了资源的释放。也可以使用上下文语法，通过`with`关键字指定文件对象的上下文环境并在离开上下文环境时自动释放文件资源

```python
def main():
    try:
        with open('致橡树.txt', 'r', encoding='utf-8') as f:
            print(f.read())
    except FileNotFoundError:
        print('无法打开指定的文件!')
    except LookupError:
        print('指定了未知的编码!')
    except UnicodeDecodeError:
        print('读取文件时解码错误!')

if __name__ == '__main__':
    main()
```

文件读取除了 read 方法，还可使用`for-in`循环逐行读取或者用`readlines`方法将文件按行读取到一个列表容器中

```python
import time

def main():
    # 一次性读取整个文件内容
    with open('致橡树.txt', 'r', encoding='utf-8') as f:
        print(f.read())

    # 通过for-in循环逐行读取
    with open('致橡树.txt', mode='r') as f:
        for line in f:
            print(line, end='')
            time.sleep(0.5)
    print()

    # 读取文件按行读取到列表中
    with open('致橡树.txt') as f:
        lines = f.readlines()
    print(lines)
    

if __name__ == '__main__':
    main()
```

写文本文件

```python
def main():
    try:
        f = open('test.txt', 'w', encoding='utf-8')
        for i in range(0, 100):
            f.write(str(i) + '\n')
    except IOError as ex:
        print(ex)
        print('写文件时发生错误!')
    finally:
        f.close()
    print('操作完成!')

if __name__ == '__main__':
    main()
```

### 读写二进制文件

```python
# 实现了复制图片文件的功能
def main():
    try:
        with open('guido.jpg', 'rb') as fs1:
            data = fs1.read()
            print(type(data))  # <class 'bytes'>
        with open('吉多.jpg', 'wb') as fs2:
            fs2.write(data)
    except FileNotFoundError as e:
        print('指定的文件无法打开.')
    except IOError as e:
        print('读写文件时出现错误.')
    print('程序执行结束.')

if __name__ == '__main__':
    main()
```

### 读写json文件

可以将字典或列表以JSON格式保存到文件中。json模块有4个比较重要的函数：

- `dump` - 将Python对象按照JSON格式序列化到文件中
- `dumps` - 将Python对象处理成JSON格式的字符串
- `load` - 将文件中的JSON数据反序列化成对象
- `loads` - 将字符串的内容反序列化成Python对象

```python
import json

mydict = {
    'name': '骆昊',
    'age': 38,
    'qq': 957658,
    'friends': ['王大锤', '白元芳'],
    'cars': [
        {'brand': 'BYD', 'max_speed': 180},
        {'brand': 'Audi', 'max_speed': 280},
        {'brand': 'Benz', 'max_speed': 320}
    ]
}
try:
    with open('data.json', 'w', encoding='utf-8') as fs:
        json.dump(mydict, fs)
except IOError as e:
    print(e)
print('保存数据完成!')

# 访问网络API获取数据
import requests
import json

resp = requests.get('http://api.tianapi.com/guonei/?key=APIKey&num=10')
data_model = json.loads(resp.text)
for news in data_model['newslist']:
    print(news['title'])
```

## 多进程 & 多线程

Python的os模块提供了`fork()`函数。由于Windows系统没有`fork()`调用，因此要实现跨平台的多进程编程，可以使用multiprocessing模块的`Process`类来创建子进程，而且该模块还提供了更高级的封装，例如批量启动进程的进程池（`Pool`）、用于进程间通信的队列（`Queue`）和管道（`Pipe`）等。

```python
from multiprocessing import Process
from os import getpid
from random import randint
from time import time, sleep

def download_task(filename):
    print('启动下载进程，进程号[%d].' % getpid())
    print('开始下载%s...' % filename)
    time_to_download = randint(5, 10)
    sleep(time_to_download)
    print('%s下载完成! 耗费了%d秒' % (filename, time_to_download))

def main():
    start = time()
    p1 = Process(target=download_task, args=('Python从入门到住院.pdf', ))
    p1.start()
    p2 = Process(target=download_task, args=('Peking Hot.avi', ))
    p2.start()
    p1.join()
    p2.join()
    end = time()
    print('总共耗费了%.2f秒.' % (end - start))

if __name__ == '__main__':
    main()
```

推荐使用threading模块，该模块对多线程编程提供了更好的面向对象的封装

```python
from random import randint
from threading import Thread
from time import time, sleep

def download(filename):
    print('开始下载%s...' % filename)
    time_to_download = randint(5, 10)
    sleep(time_to_download)
    print('%s下载完成! 耗费了%d秒' % (filename, time_to_download))

def main():
    start = time()
    t1 = Thread(target=download, args=('Python从入门到住院.pdf',))
    t1.start()
    t2 = Thread(target=download, args=('Peking Hot.avi',))
    t2.start()
    t1.join()
    t2.join()
    end = time()
    print('总共耗费了%.3f秒' % (end - start))

if __name__ == '__main__':
    main()

# 也可以通过继承Thread类的方式来创建自定义的线程类，然后再创建线程对象并启动线程
class DownloadTask(Thread):

    def __init__(self, filename):
        super().__init__()
        self._filename = filename

    def run(self):
        print('开始下载%s...' % self._filename)
        time_to_download = randint(5, 10)
        sleep(time_to_download)
        print('%s下载完成! 耗费了%d秒' % (self._filename, time_to_download))

def main():
    start = time()
    t1 = DownloadTask('Python从入门到住院.pdf')
    t1.start()
    t2 = DownloadTask('Peking Hot.avi')
    t2.start()
    t1.join()
    t2.join()
    end = time()
    print('总共耗费了%.2f秒.' % (end - start))

if __name__ == '__main__':
    main()
```

Python的多线程并不能发挥CPU的多核特性，这一点只要启动几个执行死循环的线程就可以得到证实了。之所以如此，是因为Python的解释器有一个“全局解释器锁”（GIL）的东西，任何线程执行前必须先获得GIL锁，然后每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行，这是一个历史遗留问题，但是即便如此，就如我们之前举的例子，使用多线程在提升执行效率和改善用户体验方面仍然是有积极意义的。

## 可变/不可变对象(赋值，拷贝，参数传递)

对于内置类型

- 可变对象：`list dict set`
- 不可变对象：`tuple string int float bool`

### copy模块

python 自带的 copy 模块，提供了深复制(copy.deepcopy)和浅复制(copy.copy)两种方法

- 赋值：只是简单的拷贝对象的引用，赋值后两个对象的`id`是一样的
- 浅拷贝(copy)：拷贝对象，但不拷贝对象的子对象。拷贝后2个对象的`id`不一样，但2个对象的子对象是同一个，`id`是一样的
- 深拷贝(deepcopy)：拷贝对象，对象的子对象也会拷贝。拷贝后2个对象的`id`不一样，对象的子对象`id`也不一样

对于不可变对象，赋值，浅拷贝，深拷贝是一样的。深浅拷贝是针对组合对象的，即拷贝对象中还引用了其它对象，如果没有则深浅拷贝是一样的。

```python
# 赋值，类似引用
# a,b指向同一个对象，赋值不会新开辟空间
a={1:{1,2,3}}
b=a
print(id(a))
print(id(b))
print(a is b)  # True
print(a == b ) # True

# 浅拷贝
import copy
a={1:{1,2,3}}
b=copy.copy(a)
print(a is b)   # False
print(a == b )  # True
print(id(a))    # 140271179329472
print(id(b))    # 140271201641536
print(id(a[1])) # 140271179670336
print(id(b[1])) # 140271179670336

# 深拷贝
import copy
a={1:{1,2,3}}
b=copy.deepcopy(a)
print(a is b)   # False
print(a == b )  # True
print(id(a))    # 140588674674624
print(id(b))    # 140588675345216
print(id(a[1])) # 140588675015488
print(id(b[1])) # 140588675015936
```

在定义类的时候，通过定义__copy__和__deepcopy__方法，可以改变copy的默认行为

```python
import copy
import functools

@functools.total_ordering
class MyClass:
  def __init__(self, name):
    self.name = name
  def __eq__(self, other):
    return self.name == other.name
  def __gt__(self, other):
    return self.name > other.name
  def __copy__(self):
    print('__copy__()')
    return MyClass(self.name)
  def __deepcopy__(self, memo):
    print('__deepcopy__({})'.format(memo))
    return MyClass(copy.deepcopy(self.name, memo))

a = MyClass('a')
sc = copy.copy(a)
dc = copy.deepcopy(a)
```

- 函数传递参数都是引用传递。如果函数参数为不可变对象，
- 函数默认参数最好设为不可变参数

## WITH 语句

对于一些需要做事先设置，事后销毁的任务，python提供了with语句来方便进行处理。比如读取一个文件的内容，可能会忘记关闭句柄；或者读取文件数据发生异常，忘记处理。使用with语句就能很好的处理。

```python
# 不使用with语句
file = open("/tmp/foo.txt")
try:
    data = file.read()
finally:
    file.close()

# 使用with语句
with open("/tmp/foo.txt") as file:
    data = file.read()

# 如果有多项也可以
with open('1.txt') as f1, open('2.txt') as  f2:
    # do something

```

with 语句处理机制是，要求with后面紧跟的对象有`__enter__()`方法和`__exit__()`方法。with后面的语句求值得到的对象，会调用`__enter__()`方法，并将该方法的返回值赋给as后面的变量，当后面的代码执行完后，调用`__exit__()`方法。

```python
class Sample:
    def __enter__(self):
        print("In __enter__()")
        return "Foo"
 
    def __exit__(self, type, value, trace):
        print("In __exit__()")
 
def get_sample():
    return Sample()
 
with get_sample() as sample:
    print("sample:", sample)
```

`__exit__()`方法有三个参数type(错误类型)，value(错误类型的值)，trace(错误发的位置)，可以更好的处理异常。

```python
class Sample:
    def __enter__(self):
        return self
 
    def __exit__(self, type, value, trace):
        print("type:", type)
        print("value:", value)
        print("trace:", trace)

    def do_something(self):
        bar = 1/0
        return bar + 10
 
with Sample() as sample:
    sample.do_something()
```

**上下文管理协议（Context Management Protocol）**：包含`__enter__()`和`__exit__()`方法，支持该协议的对象要实现这两个方法。

**上下文管理器（Context Manager）**：支持上下文管理协议的对象，这种对象实现了`__enter__()`和`__exit__()`方法。上下文管理器定义执行 with 语句时要建立的运行时上下文，负责执行 with 语句块上下文中的进入与退出操作。通常使用 with 语句调用上下文管理器，也可以通过直接调用其方法来使用。

**运行时上下文（runtime context）**：由上下文管理器创建，通过上下文管理器的`__enter__()`和`__exit__()`方法实现，`__enter__()`方法在语句体执行之前进入运行时上下文，`__exit__()`在语句体执行完后从运行时上下文退出。with 语句支持运行时上下文这一概念。

**上下文表达式（Context Expression）**：with 语句中跟在关键字 with 之后的表达式，该表达式要返回一个上下文管理器对象。

**语句体（with-body）**：with 语句包裹起来的代码块，在执行语句体之前会调用上下文管理器的 `__enter__()` 方法，执行完语句体之后会执行 `__exit__()`方法。

with 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。

## yield 关键字

## PEB 8风格指南

### 命名

1. 变量、函数和属性应该使用小写字母来拼写，如果有多个单词就使用下划线进行连接。
2. 类中受保护的实例属性，应该以一个下划线开头。
3. 类中私有的实例属性，应该以两个下划线开头。
4. 类和异常的命名，应该每个单词首字母大写。
5. 模块级别的常量，应该采用全大写字母，如果有多个单词就用下划线进行连接。
6. 类的实例方法，应该把第一个参数命名为`self`以表示对象自身。
7. 类的类方法，应该把第一个参数命名为`cls`以表示该类自身。

### 表达式和语句

1. 采用内联形式的否定词，而不要把否定词放在整个表达式的前面。例如`if a is not b`就比`if not a is b`更容易让人理解。
2. 不要用检查长度的方式来判断字符串、列表等是否为`None`或者没有元素，应该用`if not x`这样的写法来检查它。
3. 就算`if`分支、`for`循环、`except`异常捕获等中只有一行代码，也不要将代码和`if`、`for`、`except`等写在一起，分开写才会让代码更清晰。
4. `import`语句总是放在文件开头的地方。
5. 引入模块的时候，`from math import sqrt`比`import math`更好。
6. 如果有多个`import`语句，应该将其分为三部分，从上到下分别是Python**标准模块**、**第三方模块**和**自定义模块**，每个部分内部应该按照模块名称的**字母表顺序**来排列。