# 可迭代对象/迭代器/生成器

## **可迭代对象（Iteratable Object）**

实现了`__iter__`方法的对象就是可迭代对象。可通过`dir(obj)`查看对象有哪些方法

## **迭代器（Iterator）**

实现了`__iter__`和`__next__`方法的对象。

通过可迭代对象创建得到。python内置的`iter(obj)`方法可以创建一个迭代器，`iter()`方法的本质就是调用可迭代对象的`__iter__`方法。

迭代器可以通过内置的`next()`方法取值，且迭代的过程只能向前，不能后退。`next()`方法本质就是调用迭代器对象的`__next__`方法。`next()`方法一次只能迭代一个元素，直到所有元素迭代完，抛出**`StopIteration`**异常。

迭代器是一次性的，迭代完所有元素就无法再遍历，要再次遍历只能创建新的迭代器。

```python
l = [1,2,3]
l_iter = iter(l)
# 方法一：for 遍历迭代器 l_iter
for i in l_iter:
  print(i)
# 方法二：next 方法遍历迭代器 l_iter
next(l_iter)
next(l_iter)
next(l_iter)
next(l_iter)
```

**区分迭代对象和迭代器**

```python
# from collections import Iterable, Iterator
from collections.abc import Iterable, Iterator # python3.9+
l = [1,2,3]
l_iter = iter(l)

print(isinstance(l, Iterable))      # True
print(isinstance(l, Iterator))      # False
print(isinstance(l_iter, Iterable)) # True
print(isinstance(l_iter, Iterator)) # True
```

**总结：**迭代器一定是可迭代对象，迭代对象不一定是迭代器。

- 可迭代对象：`__iter__`方法返回一个迭代器，**自身无法取值**。
- 迭代器：`__iter__`方法返回一个迭代器就是自己，`__next__`方法返回集合中下一个元素。

对于集合这一类的可迭代对象，for循环的本质就是为其创建一个迭代器，然后不断调用next()方法取值直到抛出**`StopIteration`**异常，退出循环。

可以自定义可迭代对象和迭代器

```python
# from collections import Iterable, Iterator
from collections.abc import Iterable, Iterator # python3.9+

# 可迭代对象
class MyList():
    def __init__(self):
        self.elements = [1,2,3]
    # 返回一个迭代器，并将自己元素的引用传递给迭代器
    def __iter__(self):
        return MyListIterator(self.elements)

# 迭代器
class MyListIterator():
    def __init__(self, elements):
        self.index = 0
        self.elements = elements
    # 返回self，self就是实例化的对象，也就是调用者自己。
    def __iter__(self):
        return self
    # 实现取值
    def __next__(self):
        # 迭代完所有元素抛出异常
        if self.index >= len(self.elements):
            raise StopIteration
        value = self.elements[self.index]
        self.index += 1
        return value

l = MyList()
print(isinstance(l, Iterable))      # True
print(isinstance(l, Iterator))      # False
l_iter = l.__iter__()
print(isinstance(l_iter, Iterable)) # True
print(isinstance(l_iter, Iterator)) # True

print(next(l_iter))
print(next(l_iter))
print(next(l_iter))
print(next(l_iter))
```

注意自定义的可迭代对象的`__iter__`方法如果必须是返回的一个真正的迭代器，否则调用`iter()`方法会出错。

```python
class IterObj:
    
    def __iter__(self):
        return self 
        
it = IterObj()
print(iter(it))
# TypeError: iter() returned non-iterator of type 'IterObj'
```

为了引出生成器，看下迭代对象和迭代器的内存占用情况：

```python
l = [1,2,3,4,5]
l_iter = iter(l)
print(l.__sizeof__())      # 88
print(l_iter.__sizeof__()) # 32
```

迭代对象的内存是随着元素增加而逐渐增加的，迭代器的内存是固定的，只是迭代对象的一个引用。所以迭代器是不能节省内存的，但生成器可以。

## 生成器（Generator）

生成器是一种**特殊的迭代器。**它既有迭代器的功能，可通过`next`方法迭代取元素；又有独特的优点，**节省内存**。

有2种创建生成器的方法：

1. **`()`**语法，将列表生成式的**`[]`**换成**`()`**就即可创建生成器
2. 使用**`yield`**关键字将普通函数变成生成器函数

生成器和迭代器一样，可以通过`next()`方法迭代取元素，要区分生成器和迭代器可以通过**`isgenerator`**方法

```python
from collections.abc import Iterable, Iterator
from inspect import isgenerator
 
g = (i for i in range(3))
print(type(g))   # <class 'generator'>
print(isinstance(g, Iterable)) # True
print(isinstance(g, Iterator)) # True
print(isgenerator(g))          # True
print(next(g))
print(next(g))
print(next(g))
```

生成器是时间换空间，不像迭代对象已经创建好了所有元素，而是每次迭代取值的时候才计算一次，消耗cpu但节省内存。

## yield 关键字

当函数中出现`yield`关键字时，该函数就是一个生成器(generator)。

函数执行到`yield`时，返回`yield`后面的变量，暂停函数运行。当其它地方调用`next/send`时会继续运行，再次运行时从`yield`的下一句开始执行。

看个例子：

```python
from collections.abc import Iterable, Iterator
from inspect import isgenerator

def func():
    a = 1
    x = yield a
    print('x: ', x)  # None
    b = 100
    yield b
    
g = func()
print(type(g))   # <class 'generator'>
print(isinstance(g, Iterable)) # True
print(isinstance(g, Iterator)) # True
print(isgenerator(g))          # True
print(next(g)) # 1
print(next(g)) # 100
print(next(g)) # StopIteration
```

![Untitled](./png/Untitled.png)

函数中`print`的`x`是`None`，是因为`yield a`执行完就没有值了，所以`x`赋值得到的是`None`。可以在外面通过`send(var)`的方式传值进来，会将`var`传递给`yield`表达式。

```python
def func():
    print('start func...')
    a = 1
    a = yield 'abc'
    print('a: ', a)
    b = 100
    yield b
    b += 1
    print('end func...')

g = func()
print(next(g))
print('-------')
print(g.send(23))
print('=======')
print(next(g))
```

![Untitled](./png/Untitled%201.png)

- 第一次调用时必须先`next()`或`send(None)`，否则会报错。`send`后之所以为`None`是因为这时候没有上一个`yield`。所以，`next()`等同于`send(None)`
- `send(var)`与`next()`都有返回值，返回当前迭代遇到`yield`时，`yield`后面表达式的值
- 最后一次迭代`print(next(g))`不能少，否则函数`func`没有执行完，最后的`print`语句不会打印。但是直接执行`next(g)`最后程序结束又会报错。正确的做法应该是主动去捕获StopIteration异常。如果是用for循环，则for循环机制帮你判断了。

```python
def func():
    print('start func...')
    a = 1
    a = yield 'abc'
    print('a: ', a)
    b = 100
    yield b
    b += 1
    print('end func...')

g = func()
print(next(g))
print('-------')
print(g.send(23))
print('=======')
try:
    print(next(g))
except StopIteration:
    print('iter end')
```

例子：计算斐波拉契数列，当max比较大的时候，generator的优势就体现出来了，不会占用过多内存，是一边迭代计算一边返回结果的

```python
def fib(max):
    n,a,b =0,0,1
    while n < max:
        yield b
        a,b =b,a+b
        n = n+1
    return 'done'
 
a = fib(10)
print(fib(10))  # <generator object fib at 0x7f664cd90740>
for i in fib(6):
    print(i)
```

通过yield实现生产者消费者（协程）：

```python
def producer(c):
    n = 0
    while n < 5:
        n += 1
        print('producer {}'.format(n))
        r = c.send(n)
        print('consumer return {}'.format(r))

def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('consumer {} '.format(n))
        r = 'ok'

if __name__ == '__main__':
    c = consumer()
    next(c)  # 启动consumer
    producer(c)

```

![Untitled](./pic/Untitled%202.png)