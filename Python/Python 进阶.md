# Python 进阶



## 前言

本篇延续 Python 基础，在 Python 基础中，我们提及一些简单的编程，以及函数。

这篇将针对函数式编程、面向对象编程以及其他相关知识的补充。

本笔记的代码例子大多来自于参考资料中原文代码。

其中，影响比较深刻是 [廖雪峰的官方网站 - Python 教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017063826246112)

## 一、函数式编程

函数式编程，即平常我们所说的面向过程编程，即将我们的一部分程序代码抽象出来。

> 函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数。

### 1.1 高阶函数

如果一个函数可以当成一个变量，传入另一个函数，那么另一个函数称为 **高阶函数**

#### 1.1.1 map/reduce

在廖雪峰的教程中，他提及了这篇论文 [MapReduce: Simplified Data Processing on Large Clusters](http://research.google.com/archive/mapreduce.html) 。

论文中，提到了 MapReduce 这么一种编程模型。

**Map：**

Map 中需要传入两个参数：

- 函数
- Iterable 对象

然后将 Iterable 中的变量经过函数的处理，形成一个新的 Iterable 对象。

**Python 实现：**

```python
>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

**Reduce：**

```python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```

**例子：**

1. str2int：

   ```python
   >>> from functools import reduce
   >>> def fn(x, y):
   ...     return x * 10 + y
   ...
   >>> def char2num(s):
   ...     digits = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}
   ...     return digits[s]
   ...
   >>> reduce(fn, map(char2num, '13579'))
   13579
   ```

#### 1.1.2 filter 过滤器

`filter` 过滤器与 `Map` 函数类似，将过滤函数作用于序列中。

不同的是，该过滤函数仅仅返回 `true` / `false` ，通过 `true` / `false` 去判断在返回的序列中的舍留。



**例子：删除空字符串**

```python
def not_empty(s):
    return s and s.strip()

list(filter(not_empty, ['A', '', 'B', None, 'C', '  ']))
# 结果: ['A', 'B', 'C']
```

#### 1.1.3 sorted 排序

使用内置的 `sorted` 即可对 `list` 进行排序：

```python
>>> sorted([36, 5, -12, 9, -21])
[-21, -12, 5, 9, 36]
```

 `sorted` 也是一个高阶函数，通过 `key` 来指定一个函数去处理：

```python
>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]
```

添加 `reserve` 参数进行反转

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']
```



### 1.2 返回函数

 在一个函数里面封装一个函数，然后返回，

```python
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum
```



**闭包**

 这里得到的三个函数引用同一个 `i` ，但是 f 没有立即被执行。

```python
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()

>>> f1()
9
>>> f2()
9
>>> f3()
9
```

如果必须得使用循环，那就多嵌套一层，让它立即执行，

```python
def count():
    def f(j):
        def g():
            return j*j
        return g
    fs = []
    for i in range(1, 4):
        fs.append(f(i)) # f(i)立刻被执行，因此i的当前值被传入f()
    return fs
```



### 1.3 匿名函数

如 C++ 、 Java 、Js 中都有相关的函数。

在 Python 中通过，`ambda x: x * x`

```python
def f(x):
    return x * x
```



### 1.4 装饰器

装饰器，就是在不改变原来业务的基础上，新增一些业务，相当于 Java 的面向切面编程。

**log 二层嵌套**

```python
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```

如， 对于 `log` 注解

```python
@log
def now():
    print('2015-3-25')
    
>>> now()
call now():
2015-3-25
```

即为，

```python
now = log(now)
```



**log 三层嵌套**

定义，在多一层参数层，

```python
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
```

如果执行，

```python
@log('execute')
def now():
    print('2015-3-25')
```

等同于，

```python
>>> now = log('execute')(now)
```

函数本身是一个对象， 那么发现 `name` 没有被继承过来，

```python
>>> now.__name__
'wrapper'
```

通过 Python 内置的 `functools.wraps` 即可

```python
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
```

### 1.5 偏函数

当我们需要经常使用带参数的函数的时候，我们可能手动定义一个函数

```python
def int2(x, base=2):
    return int(x, base)
```

不过，Python 有内置功能

```python
>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
>>> int2('1010101')
85
```

## 二、模块

### 2.1 使用模块

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

' a test module '

__author__ = 'Michael Liao'

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__':
    test()
```

只有直接运行该模块的时候，才会将模块变量 `__name__`  设置为 `__main__`

**作用域**

Python 中的变量的作用域通过 `_` 来表示，但仅仅是形式表示，如果你刻意想要获取，也是可以的。

- `__xx__` ：是特殊变量
- `_` 或 `__` 开头的是：非公开变量

### 2.2 第三方模块

通过 `pip` 管理工具去安装，通过 `import` 引入。

**模块搜索路径**

- 当我们试图加载第一个模块的时候，Python 先从当前目录去查找
- 默认情况下，Python解释器会搜索当前目录、所有已安装的内置模块和第三方模块，搜索路径存放在`sys`模块的`path`变量中：
  - 运行时修改：修改 `sys` 模块中的 `path`
  - 环境变量修改：设置 `PYTHONPATH`

## 三、面向对象编程

### 3.1 类和实例

一个类与它的构造函数：

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score
```

可以自由给一个类绑定属性，与 JS 很相似，

```python
>>> bart.name = 'Bart Simpson'
>>> bart.name
'Bart Simpson'
```

**通过封装**

下面封装了一个方法，让他可以去访问我们的对象，

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
```



### 3.2 访问限制

通过添加 `__` ，这时名字已经无法访问了，而 `_` 开头的依然可以直接访问，

```python
>>> bart = Student('Bart Simpson', 59)
>>> bart.__name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute '__name'
```

可以在内部，添加 `get`、`set` 方法去允许访问。

而其原理只是 `name` 变成了 `_Student__name` 这么一个名字。



### 3.3 继承和多态

这是一个父类，有个方法 `run`

```python
class Animal(object):
    def run(self):
        print('Animal is running...')
```

那么，子类为

```python
class Dog(Animal):
    pass

class Cat(Animal):
    pass
```

**多态** ：假设我们定义了这么一个函数，那么传入不同的类的实例，就会运行不同的结果。

```python
def run_twice(animal):
    animal.run()
    animal.run()
```

传入 `Dog` 、`Cat` 实现了不一样的运行结果。

因为 Python 是一个动态语言，它不需要像 Java 一样严格遵循是 Animal 的一个子类，它只需要说它类似于 Animal ，有一个 run 的函数。即它能跑的，它像动物的这么一层抽象。

如果需要严格多态，可以 `isinstance()` 去判断。

### 3.4 获取对象信息 

**type 获取类型**

```
>>> type(123)
<class 'int'>
```

**使用isinstance()**

```
>>> isinstance(h, Husky)
True
```

**使用 dir()**

```
>>> dir('ABC')
['__add__', '__class__',..., '__subclasshook__', 'capitalize', 'casefold',..., 'zfill']
```

使用 `len()` 必须添加 `__len__` 属性

配合`getattr()`、`setattr()`以及`hasattr()`，我们可以直接操作一个对象的状态：



### 3.5 实例属性与类属性

不通过 self 定义的属性，均为类属性。

```
>>> class Student(object):
...     name = 'Student'
...
```

## 四、面向对象高级编程

### 4.1 使用 `__slots__` 

对于一个类或者实例，我们可以任意添加新的属性或方法。

通过下面，可以给实例对象绑定一个新方法：

```
>>> def set_age(self, age): # 定义一个函数作为实例方法
...     self.age = age
...
>>> from types import MethodType
>>> s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
>>> s.set_age(25) # 调用实例方法
>>> s.age # 测试结果
25
```

给类绑定一个新方法：

```
>>> def set_score(self, score):
...     self.score = score
...
>>> Student.set_score = set_score
```

**使用`__slots__`**

使用 `slot` 可以定义一个类允许添加的一组属性，

```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

那么，

```python
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```

使用`__slots__`要注意，`__slots__`定义的属性仅对当前类实例起作用，对继承的子类是不起作用的：



### 4.2 使用 `@property` 

对于一个类，修改获取属性封装后变得麻烦了，

```python
class Student(object):

    def get_score(self):
         return self._score

    def set_score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

通过 `@property` 去实现，实现了 `getter` 和 `setter` 的功能，

```python
class Student(object):

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

### 4.3 多继承

如下实现了一个多继承，

```python
class Bat(Mammal, Flyable):
    pass
```

逻辑上，继承是只有一条主线，可以混入一些东西进来，成为 `MixIn`

```python
class Dog(Mammal, RunnableMixIn, CarnivorousMixIn):
    pass
```

### 4.4 定制类

 **`__str__`** , 可以去主动定义 `print` 的行为，

```
 def __str__(self):
        return 'Student object (name=%s)' % self.name
    __repr__ = __str__
```

`__repr__` 可以去定义调试，`__str__` 的行为



**`__iter__`** ，如果希望能使用 `for ` 去运行，

使用 `iter` 返回一个可迭代对象，通过 `next` 去不断迭代获得下一个值。

```python
  def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000: # 退出循环的条件
            raise StopIteration()
        return self.a # 返回下一个值
```

 **`__getitem__`**  可以让我们像数组下标一样取元素，

然而如果使用 切片，需要作判断，

```python
class Fib(object):
    def __getitem__(self, n):
        if isinstance(n, int): # n是索引
            a, b = 1, 1
            for x in range(n):
                a, b = b, a + b
            return a
        if isinstance(n, slice): # n是切片
            start = n.start
            stop = n.stop
            if start is None:
                start = 0
            a, b = 1, 1
            L = []
            for x in range(stop):
                if x >= start:
                    L.append(a)
                a, b = b, a + b
            return L
```

引用廖雪峰原文，进行补充，

> 但是没有对step参数作处理：
>
> 也没有对负数作处理，所以，要正确实现一个`__getitem__()`还是有很多工作要做的。
>
> 此外，如果把对象看成`dict`，`__getitem__()`的参数也可能是一个可以作 key 的object，例如`str`。
>
> 与之对应的是`__setitem__()`方法，把对象视作 list 或 dict 来对集合赋值。最后，还有一个`__delitem__()`方法，用于删除某个元素。
>
> 总之，通过上面的方法，我们自己定义的类表现得和Python自带的list、tuple、dict没什么区别，这完全归功于动态语言的“鸭子类型”，不需要强制继承某个接口。

 **`__getattr__`**

当一个实例找不到对应的属性值的，他会去 `__getattr__` 寻求帮助，

```python
class Student(object):

    def __init__(self):
        self.name = 'Michael'

    def __getattr__(self, attr):
        if attr=='score':
            return 99
```

**一个链式递归调用的例子：**

 如果要写SDK，给每个URL对应的API都写一个方法，那得累死，而且，API一旦改动，SDK也要改。

```python
class Chain(object):

    def __init__(self, path=''):
        self._path = path

    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))

    def __str__(self):
        return self._path

    __repr__ = __str__
```

这样，无论API怎么变，SDK都可以根据URL实现完全动态的调用，而且，不随API的增加而改变！



 **`__call__`** 通过实现类的 `call`  函数可以使得一个实例变成可调用的，

```python
class Student(object):
    def __init__(self, name):
        self.name = name

    def __call__(self):
        print('My name is %s.' % self.name)
```

 通过 `callable()` 函数可以判断一个实例是否可以被调用



### 4.5 枚举类

用整数实现枚举，

```python
JAN = 1
FEB = 2
MAR = 3
...
NOV = 11
DEC = 12
```

而Python有个内置类，可以生成枚举对象，

```python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

```

枚举输出，所有枚举的月份，

```python
for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)
Jan => Month.Jan , 1
Feb => Month.Feb , 2
Mar => Month.Mar , 3
Apr => Month.Apr , 4
May => Month.May , 5
Jun => Month.Jun , 6
Jul => Month.Jul , 7
Aug => Month.Aug , 8
Sep => Month.Sep , 9
Oct => Month.Oct , 10
Nov => Month.Nov , 11
Dec => Month.Dec , 12
```

使用 `@unique` 装饰器，可以帮助我们检测没有重复值。

### 4.6 元类

`type()`

动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的。

我们可以发现 `Hello` 类的类是 `type`

```python
>>> print(type(Hello))
<class 'type'>
```

于是 , 我们想是不是可以动态地创建一个类,

```python
>>> def fn(self, name='world'): # 先定义函数
...     print('Hello, %s.' % name)
...
>>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
>>> h = Hello()
>>> h.hello()
Hello, world.
```

其中 `type`  函数接受三个参数 : 

- 类名
- 继承的类的元组
- 函数

**metaclass**

除了使用 `type` 去创建一个类,还可以使用 `metaclass` 

可以先创建 `metaclass` 的,然后创建 `class`

```python
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)
```

然后创建一个类,

```python
class MyList(list, metaclass=ListMetaclass):
    pass
```

跟普通 `list` 类相比,它就增加了一个 `add` 方法.

### 4.7 ORM 框架例子

当我们想定义一个 `ORM` 框架 , 里面的 `User` , 我们期待这样的代码:

```python
class User(Model):
    # 定义类的属性到列的映射：
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')

# 创建一个实例：
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
# 保存到数据库：
u.save()
```

首先定义一个 `Field`  类,里面包含名字和类型,

```python
class Field(object):

    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type

    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name)
```

根据不同的类型,派生出不同的 `Field` ,

```python
class StringField(Field):

    def __init__(self, name):
        super(StringField, self).__init__(name, 'varchar(100)')

class IntegerField(Field):

    def __init__(self, name):
        super(IntegerField, self).__init__(name, 'bigint')
```

TODO





## 五、错误、调试和测试

### 5.1 错误处理

通过 `try` 、`except` 、`finally` 捕获处理异常，

```python
try:
    print('try...')
    r = 10 / 0
    print('result:', r)
except ZeroDivisionError as e:
    print('except:', e)
finally:
    print('finally...')
print('END')
```

**调用栈**

```python
# err.py:
def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    bar('0')

main()

Traceback (most recent call last):
  File "err.py", line 11, in <module>
    main()
  File "err.py", line 9, in main
    bar('0')
  File "err.py", line 6, in bar
    return foo(s) * 2
  File "err.py", line 3, in foo
    return 10 / int(s)
ZeroDivisionError: division by zero
```

 **logging 模块**

`logging` 模块可以用来显示错误，

```python
def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

main()
print('END')
```

**raise 抛出错误**

```python
 raise FooError('invalid value: %s' % s)
```

### 5.2 调试

**断言**

通过 `assert` 假定结果，如果结果相违背，则会抛出异常，

```python
def foo(s):
    n = int(s)
    assert n != 0, 'n is zero!'
    return 10 / n

def main():
    foo('0')
```



**logging 输出日志**

```
import logging
logging.basicConfig(level=logging.INFO)
```

`logging` 的好处就是他有不同的输出日志等级，可以筛选到`debug`，`info`，`warning`，`error` 等不同级别的调试信息。



**pdb**

TODO ，可以单步，加断点等调试方法。

### 5.3 单元测试

### 5.4 文档测试



## 六、I/O 编程

### 6.1 文件读写

首先打开一个文件，`r` 表示读取，

```python
>>> f = open('/Users/michael/test.txt', 'r')
```

使用 `read` 方法一次获取文件的所有内容，

```python
>>> f.read()
'Hello, world!'
```

最后调用关闭文件，

```python
>>> f.close()
```

为了保证，我们能够正确的关闭文件，进行

```pyhton
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
        f.close()
```

可以使用 `with` 进行处理，

```python
with open('/path/to/file', 'r') as f:
    print(f.read())
```

**file-like 对象**  

在 `Python` 中，只要有 `open` 函数 返回带有 `read()` 方法，即为 `file-like Object` ，`StringIO` 就是其中一种。



**二进制文件**

二进制文件需要使用 `rb` 方式打开，

```
>>> f = open('/Users/michael/test.jpg', 'rb')
>>> f.read()
b'\xff\xd8\xff\xe1\x00\x18Exif\x00\x00...' # 十六进制表示的字节
```

**字符编码**

通过 `encoding` 指定字符编码，通过 `errors` 指定错误处理方法，

```
>>> f = open('/Users/michael/gbk.txt', 'r', encoding='gbk', errors='ignore')
```

**写文件**

普通的写文件，

```python
>>> f = open('/Users/michael/test.txt', 'w')
>>> f.write('Hello, world!')
>>> f.close()
```

使用 `with` 写文件进行保险，

```python
with open('/Users/michael/test.txt', 'w') as f:
    f.write('Hello, world!')
```

### 6.2 StringIO 和 BytesIO

**StringIO**

很多时候 `I/O` 不一定是文件，也可以是内存中进行读写，

`StringIO` 就是在内存中读写 `str`。

通过 `write` 去写 `str` ，通过 `getvalue` 去获得写入后的 `str`

```python
>>> from io import StringIO
>>> f = StringIO()
>>> f.write('hello')
5
>>> f.write(' ')
1
>>> f.write('world!')
6
>>> print(f.getvalue())
hello world!
```

**BytesIO**

`BytesIO` 实现了在内存中读写 `bytes` ，

```python
>>> from io import BytesIO
>>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
>>> f.read()
b'\xe4\xb8\xad\xe6\x96\x87'
```



### 6.3 操作文件和目录

```python
>>> import os
>>> os.name # 操作系统类型
'posix'
```

如果是 `posix` 说明是类 `unix` 系统，否则 `nt` 即为 `Windows` 系统 ，通过

`os.name()` 获取完整信息



**环境变量**

操作系统中定义的环境变量，

```
os.environ
```

通过 `os.environ.get('key')`，可以获取特定的环境变量的值。



**操作文件和目录**

```python
# 查看当前目录的绝对路径:
>>> os.path.abspath('.')
'/Users/michael'
# 在某个目录下创建一个新目录，首先把新目录的完整路径表示出来:
>>> os.path.join('/Users/michael', 'testdir')
'/Users/michael/testdir'
# 然后创建一个目录:
>>> os.mkdir('/Users/michael/testdir')
# 删掉一个目录:
>>> os.rmdir('/Users/michael/testdir')
```

拼接路径的话，要使用 `os.path.join()`

拆分路径，使用`os.path.split()`

获取，文件拓展名，使用 `os.path.splitext()`

文件操作，

```python
# 对文件重命名:
>>> os.rename('test.txt', 'test.py')
# 删掉文件:
>>> os.remove('test.py')
```

利用 Python 特性来过滤文件，找到目录底下的文件夹

```python
>>> [x for x in os.listdir('.') if os.path.isdir(x)]
['.lein', '.local', '.m2', '.npm', '.ssh', '.Trash', '.vim', 'Applications', 'Desktop', ...]
```

找到目录底下 `.py` 文件，

```python
>>> [x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']
['apis.py', 'config.py', 'models.py', 'pymonitor.py', 'test_db.py', 'urls.py', 'wsgiapp.py']
```

### 6.4 序列化

程序在运行的过程中，所有的变量都是存在内存中，

然而，无论是我们存储到文件中，还是在网络上传输，都需要进行序列化，

通过 `pickle` 可以实现序列化，

```python
>>> import pickle
>>> d = dict(name='Bob', age=20, score=88)
>>> pickle.dumps(d)
b'\x80\x03}q\x00(X\x03\x00\x00\x00ageq\x01K\x14X\x05\x00\x00\x00scoreq\x02KXX\x04\x00\x00\x00nameq\x03X\x03\x00\x00\x00Bobq\x04u.'
```

`pickle.dumps()` 方法把任意对象序列化成一字节码，然后写入到一个 `file-like Object`，

```python
>>> f = open('dump.txt', 'wb')
>>> pickle.dump(d, f)
>>> f.close()
```

反序列化，

```python
>>> f = open('dump.txt', 'rb')
>>> d = pickle.load(f)
>>> f.close()
>>> d
{'age': 20, 'score': 88, 'name': 'Bob'}
```

**JSON**

要在不同编程语言中传递对象，就必须格式化成一种标准的形式，一般采用 `XML` 或者 `JSON` 的方式。

通过内置的 `json` 模块，即可

```python
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)
'{"age": 20, "score": 88, "name": "Bob"}'
```

反序列化，

```python
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> json.loads(json_str)
{'age': 20, 'score': 88, 'name': 'Bob'}
```

在 `Python` 中 `dict` 和 `json`可以自由相互转换，但是 `class` 对象不行。

我们可以写函数，进行相互转换

```python
def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }

def dict2student(d):
    return Student(d['name'], d['age'], d['score'])
```

通常 `class`  实例都有一个 `__dict__` 属性，就是一个 `dict`，用来存储实例变量。



## 七、进程与线程

在操作系统中，每个运行的程序即是一个进程，其中还有更小的运行单位即为线程。

### 7.1 多进程

进程只能通过操作系统去创建，于是在 `unix` 系统中使用 `os` 模块：

```python
import os

print('Process (%s) start...' % os.getpid())
# Only works on Unix/Linux/Mac:
pid = os.fork()
if pid == 0:
    print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
else:
    print('I (%s) just created a child process (%s).' % (os.getpid(), pid))
```

在 `Windows` 系统中，通过 `Process` 类来创建，

```python
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Run child process %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Child process will start.')
    p.start()
    p.join()
    print('Child process end.')
```

使用进程池，TODO

```python
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    print('All subprocesses done.')
```

**子进程** TODO

### 7.2 多线程

Python 中提供了两个模块用于操纵线程，`_thread` 和 `threading` 两个模块，

```python
import time, threading

# 新线程执行的代码:
def loop():
    print('thread %s is running...' % threading.current_thread().name)
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(1)
    print('thread %s ended.' % threading.current_thread().name)

print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print('thread %s ended.' % threading.current_thread().name)
```

多进程之间的变量互不干扰，而线程间，变量有共享的情况，

```python
import time, threading

# 假定这是你的银行存款:
balance = 0

def change_it(n):
    # 先存后取，结果应该为0:
    global balance
    balance = balance + n
    balance = balance - n

def run_thread(n):
    for i in range(100000):
        change_it(n)

t1 = threading.Thread(target=run_thread, args=(5,))
t2 = threading.Thread(target=run_thread, args=(8,))
t1.start()
t2.start()
t1.join()
t2.join()
print(balance)
```

解决，

```python
def run_thread(n):
    for i in range(100000):
        # 先要获取锁:
        lock.acquire()
        try:
            # 放心地改吧:
            change_it(n)
        finally:
            # 改完了一定要释放锁:
            lock.release()
```

**多核CPU 支持**

理论上，按照 CPU 核心数我们写个死循环，创建对应数量的线程，应该是可以跑满 CPU 每个核心的，但是，

```
import threading, multiprocessing

def loop():
    x = 0
    while True:
        x = x ^ 1

for i in range(multiprocessing.cpu_count()):
    t = threading.Thread(target=loop)
    t.start()
```

>因为Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，任何Python线程执行前，必须先获得GIL锁，然后，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。
>
>不过，也不用过于担心，Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个Python进程有各自独立的GIL锁，互不影响。



### 7.3 ThreadLocal
有时，我们希望线程间的变量独立开来，不能共享。如果使用局部变量，那么又产生嵌套传参的问题。
我们想到，使用一个全局的字典，把Thread 当作 key值去取到对应的Student 参数。
```python
global_dict = {}

def std_thread(name):
    std = Student(name)
    # 把std放到全局变量global_dict中：
    global_dict[threading.current_thread()] = std
    do_task_1()
    do_task_2()

def do_task_1():
    # 不传入std，而是根据当前线程查找：
    std = global_dict[threading.current_thread()]
    ...

def do_task_2():
    # 任何函数都可以查找出当前线程的std变量：
    std = global_dict[threading.current_thread()]
```

于是，ThreadLocal 应蕴而生。
```python
import threading
    
# 创建全局ThreadLocal对象:
local_school = threading.local()

def process_student():
    # 获取当前线程关联的student:
    std = local_school.student
    print('Hello, %s (in %s)' % (std, threading.current_thread().name))

def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()

t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
```
ThreadLocal 是一个全局变量，但在每个线程中互不干扰。

### 7.4 进程 vs 线程
如果我们需要多任务，那么我们通常会使用 Master-Worker 的设计模式。
- Master 负责分配任务
- Worker 负责执行任务
进程的稳定强，一个挂掉了不会影响其他进程，但资源消耗多。
线程挂掉了，会导致进程也挂掉，影响其他线程，但资源消耗少

**CPU密集型 VS I/O密集型**
CPU密集型要充分利用多核，使用 C语言之类可以提高效率。
I/O密集型，C语言之类提升的效率有限，可以使用脚本语言。

对应到Python语言，单线程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。我们会在后面讨论如何编写协程。

**异步 I/O**

### 7.5 分布式进程

TODO

## 八、正则表达式

TODO



### 九、virtualenv 虚拟环境

`virtualenv` 就是使用一套虚拟环境隔离 `Python` 的运行环境。

使用 `pip` 安装 `virtualenv`：

```python
$ pip3 install virtualenv
```

然后，假定我们要开发一个新的项目，需要一套独立的 `Python` 运行环境，可以这么做：

1. 创建目录：

 ```bash
   Mac:~ michael$ mkdir myproject
   Mac:~ michael$ cd myproject/
   Mac:myproject michael$
 ```

2. 创建一个独立的 `Python` 运行环境，命名为 `venv`

   ```
   virtualenv --no-site-packages venv
   ```

添加 `--no-site-packages` 那么，原本的包不会被添加进来。

创建后，

3. 使用 `source` 进入环境

   ```python
   Mac:myproject michael$ source venv/bin/activate
   (venv)Mac:myproject michael$
   ```

那么即可在该环境里面畅游，

4. 使用 `deactive` 退出该环境

   ```python
   (venv)Mac:myproject michael$ deactivate 
   Mac:myproject michael$ 
   ```



## 十、异步 I/O

在程序中我们通常需要处理 I/O，平常我们通过多线程/进程的方式来进行处理 I/O。当一个线程/进程被阻塞了之后，就切换到另一个没有阻塞的线程进行运行，但这样切换势必造成损耗，而且系统也不可能无上限的增加线程。

异步 I/O 即也是非阻塞 I/O，当我们的代码需要处理一个耗时巨大的操作的时候，我们如果只发出 I/O 指令，不等待 I/O 结果直接进行下一步运行，不进行阻塞。等 I/O 结束后，通知 CPU 进行处理。

异步 I/O 的实现需要一个消息循环，在消息循环中，主线程不断地重复 “读取消息-处理消息” 这一过程。

```python
loop = get_event_loop()
while True:
    event = loop.get_event()
    process_event(event)
```

在图形界面程序中，早就有所应用。一个负责 GUI 的程序，不停地读取事件消息，然后处理，由于处理得很快，大家无法感知。

### 10.1 协程

协程，又称微线程，纤程。英文名Coroutine。

协程的概念很早就提出来了，但直到最近几年才在某些语言（如Lua）中得到广泛应用。

```python
def A():
    print('1')
    print('2')
    print('3')

def B():
    print('x')
    print('y')
    print('z')
    
1
2
x
y
3
z
```

`A、B` 得执行有点像多线程，但实际上只在一个线程中执行。

所有有什么优势呢？

- 减少开销：所有的协程切换都由程序控制，那么可以大大减少线程切换的开销。
- 无需加锁：因为只有一个线程，所有无需加锁，共享变量只需要加状态即可。

因此，在 Python 中充分利用 CPU 的方法，就是多进程 + 协程。



**Python 中协程的实现**

Python 中通过 `Generator` 机制去实现，这是一个生产者-消费者的例子，

```python
def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)
```

在 `produce` 中拿到了一个 `consumer` 的生成器，然后启动 `c.send(None)` 启动消费者消费。

然后消费者发现是 `None` ，并没有消费。

然后进行循环，`produce` 产生 n 号产品=> 消费者消耗 n号产品 => 消费者告诉生产者我吃完了。



### 10.2 asyncio

`asyncio` 是Python 的一个标准库，编程模型就是一个事件循环。我们获取到 `asyncio`  中一个 `EventLoop` 的引用，然后将协程放入到 `EventLoop` 里面进行处理。

**例子1：**

这里，定义了一个 `hello` 的协程，里面有个 1秒的睡眠 I/O 等待，这个时候事件循环就可以去处理别的事情。

```python
import asyncio

@asyncio.coroutine
def hello():
    print("Hello world!")
    # 异步调用asyncio.sleep(1):
    r = yield from asyncio.sleep(1)
    print("Hello again!")

# 获取EventLoop:
loop = asyncio.get_event_loop()
# 执行coroutine
loop.run_until_complete(hello())
loop.close()
```

**例子2：**使用 tasks 封装两个协程

```python
import threading
import asyncio

@asyncio.coroutine
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

Hello world! (<_MainThread(MainThread, started 140735195337472)>)
Hello world! (<_MainThread(MainThread, started 140735195337472)>)
(暂停约1秒)
Hello again! (<_MainThread(MainThread, started 140735195337472)>)
Hello again! (<_MainThread(MainThread, started 140735195337472)>)
```

**例子3：**异步网络连接

```python
import asyncio

@asyncio.coroutine
def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = yield from connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    yield from writer.drain()
    while True:
        line = yield from reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

wget www.sohu.com...
wget www.sina.com.cn...
wget www.163.com...
(等待一段时间)
(打印出sohu的header)
www.sohu.com header > HTTP/1.1 200 OK
www.sohu.com header > Content-Type: text/html
...
(打印出sina的header)
www.sina.com.cn header > HTTP/1.1 200 OK
www.sina.com.cn header > Date: Wed, 20 May 2015 04:56:33 GMT
...
(打印出163的header)
www.163.com header > HTTP/1.0 302 Moved Temporarily
www.163.com header > Server: Cdn Cache Server V2.0
```



### 10.3 async/await

这是 Python 原本的协程异步操作写法，

```python
@asyncio.coroutine
def hello():
    print("Hello world!")
    r = yield from asyncio.sleep(1)
    print("Hello again!")
```

这是新的写法，

```python
async def hello():
    print("Hello world!")
    r = await asyncio.sleep(1)
    print("Hello again!")
```



### 10.4 aiohttp

`asyncio`实现了TCP、UDP、SSL等协议，`aiohttp`则是基于`asyncio`实现的HTTP框架。

```python
import asyncio

from aiohttp import web

async def index(request):
    await asyncio.sleep(0.5)
    return web.Response(body=b'<h1>Index</h1>')

async def hello(request):
    await asyncio.sleep(0.5)
    text = '<h1>hello, %s!</h1>' % request.match_info['name']
    return web.Response(body=text.encode('utf-8'))

async def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', index)
    app.router.add_route('GET', '/hello/{name}', hello)
    srv = await loop.create_server(app.make_handler(), '127.0.0.1', 8000)
    print('Server started at http://127.0.0.1:8000...')
    return srv

loop = asyncio.get_event_loop()
loop.run_until_complete(init(loop))
loop.run_forever()
```

## 参考

- 《Python 编程 从入门到实践》—— Eric Malthes【著】 袁国忠【译】
- [廖雪峰的官方网站 - Python 教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017063826246112)

- [RealPython](https://realpython.com/)

 