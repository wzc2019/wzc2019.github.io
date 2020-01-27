# Python 基础

[toc]

## 前言

本篇是我对 Python 进行系统学习的总结的一些相关笔记，只进行较为简略的记录。

本笔记的代码例子大多来自于参考资料中原文代码。

其中，影响比较深刻是 [廖雪峰的官方网站 - Python 教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017063826246112)

## 一、简单数据结构

### 1.1 变量

在 Python 中定义一个变量非常简单，如下输出一个 Hello World 。

```python
message = "Hello World!"
print(message)
```

**基本数据类型**

- 整数
- 浮点数
- 字符串
- 布尔值
- 常量（大写，无法强制保证）

### 1.2 字符串

#### 1.2.1 字符编码

一开始出现了 ASCII 码，一个字节用于英文字符表示，后来出现了各国各个字符的编码，如 GBK。

最后，为了改变字符编码混乱的局面推出了 UniCode ，UniCode 一定程度上是一种规范，而 utf-8 就是其中一种实现。而 Python 也是以 UniCode 字符编码的。

#### 1.2.2 常用字符 API

**大小写转换**

`title` 、`lower`、`upper` 等函数。

**整数与字符转化**

通过 `ord` 和 `chr` 函数，可以将字符与整数进行相互转化。

```python
>>> ord('A')
65
>>> ord('中')
20013
>>> chr(66)
'B'
>>> chr(25991)
'文'
```

**bytes 字节形式**

简单定义：

```python
x = b'ABC'
```

通过 `encode()` ，编码为特定的格式：

```python
>>> 'ABC'.encode('ascii')
b'ABC'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

通过 `decode()`，从字节形式获取到字符

```
>>> b'ABC'.decode('ascii')
'ABC'
>>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
'中文'
```
添加 `ignore` 参数，否则会报错，
```
>>> b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
'中'
```

通过文件注释，告诉编辑器格式：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

**获取长度**

通过通用的 `len()` 获取相应的长度

**格式化**

格式化有两种方法：

1. 一种通过 % 符号，

   ```python
   >>> 'Hello, %s' % 'world'
   'Hello, world'
   >>> 'Hi, %s, you have $%d.' % ('Michael', 1000000)
   'Hi, Michael, you have $1000000.'
   ```
   
2. 通过 `format()` 方法，

 ```python
   >>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
   'Hello, 小明, 成绩提升了 17.1%'
 ```


## 二、其他内置类型

### 2.1 list 列表

Python内置的一种数据类型是列表：list。list是一种有序的集合，可以随时添加和删除其中的元素。

**初始化：**通过 `[]` 去包裹，

```python
>>> classmates = ['Michael', 'Bob', 'Tracy']
>>> classmates
['Michael', 'Bob', 'Tracy']
```

**通过下标取值** ，要小心越界

```python
>>> classmates[-1]
'Tracy'
```

**添加数据**

- append() : 添加到末尾
- insert() : 添加到任意位置

**删除数据**

- pop() : 弹出最后一个
- pop(i) : 弹出索引位置
- del() : 删除任意一个

**通过下标修改**

**数据类型可以不同**

```python
>>> L = ['Apple', 123, True]
```



### 2.2 tuple 元组

元组类似于 list，但里面的内容不可改变。

```
>>> classmates = ('Michael', 'Bob', 'Tracy')
```

当只有一个元素的时候，需要加个 `逗号` 来避免二义性。

```python
>>> t = (1,)
>>> t
(1,)
```

**tuple 与 list**

```python
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```

其中里面的 list 是个指针，所以是可变的。



### 2.3 dict 字典

即其他语言中的 `map` 本质上是一种键-值对关系，

 `key` 与存储位置建立一个 `Hash` 映射关系，通过 `key` 计算即可再次得到存储位置，拿到 `value`，提高了读取效率。

**初始化**

```python
>>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
>>> d['Michael']
95
```

**存入与获取** ， 但不可获取不存在的 `key` 值。

```python
>>> d['Adam'] = 67
>>> d['Adam']
67
```

**判断是否存在**

```python
>>> 'Thomas' in d
False
```

**通过 get 方法** ，如果不存在，默认返回 `None`

```python
>>> d.get('Thomas')
>>> d.get('Thomas', -1)
-1
```

**使用 `pop(key)` 删除**

使用 `dict` 优点，查找速度快，但内存消耗更大了。

注意，key 得是一个不可变的对象，一旦可变，求出来的 Hash 值就不同了。

### 2.4 set 集合

**初始化** ，`set ` 中不可包含重复元素，添加重复元素，自动被过滤。

```python
>>> s = set([1, 1, 2, 2, 3, 3])
>>> s
{1, 2, 3}
```

**增加与删除**

通过 `add()`、`remove()` 去实现。

### 2.5 可变与不可变

例如 `list` 就是一个可变对象，可以通过 `sort()` 方法去排序。

```python
>>> a = ['c', 'b', 'a']
>>> a.sort()
>>> a
['a', 'b', 'c']
```

`str` 是一个不可变对象，

a 通过 `replace` 后创建了一个新对象，本身并没有改变。

```python
>>> a = 'abc'
>>> a.replace('a', 'A')
'Abc'
>>> a
'abc'
```

## 三、流程控制

这个东西，其实不同语言的差异实在太小，就一些格式上的差别，没啥可说的

### 3.1 if 控制

就像这样，可以添加 `else` 

```
age = 20
if age >= 18:
    print('your age is', age)
    print('adult')
```

### 3.2 循环控制

```
names = ['Michael', 'Bob', 'Tracy']
for name in names:
    print(name)
```

其他，还有 `while` 、`break`、`continue ` 这里就不多赘述

`range()` 生成一个数列

## 四、函数

### 4.1 函数定义

**定义函数**

```python
def my_abs(x):
    if x >= 0:
        return x
    else:
        return -x
```

**引入函数**

```python
>>> from abstest import my_abs                          │
│>>> my_abs(-9)  
```

**空语句**

```
 pass
```

**参数检查**

如果我们的代码传入的参数类型不对，对于 Python 这种弱类型语言，他无法帮我们进行检查。

我们需要手动检查并抛出异常，

```python
def my_abs(x):
    if not isinstance(x, (int, float)):
        raise TypeError('bad operand type')
    if x >= 0:
        return x
    else:
        return -x
```

**返回多个返回值**

返回多个返回值，就像一个元组，

```python
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
```

### 4.2 函数参数

**位置参数**： 即普通常见的形式。

```python
def power(x):
    return x * x
```

**默认参数**

```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```

**使用不可变参数作为默认参数** , 否则

```python
def add_end(L=[]):
    L.append('END')
    return L

>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
```

修改方法，

```python
def add_end(L=None):
    if L is None:
        L = []
    L.append('END')
    return L
```

**可变参数**

```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```

**通过 * 解构 list**

```python
>>> nums = [1, 2, 3]
>>> calc(*nums)
14
```

**关键字参数与可变参数**

```python
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

那么，可以传入

```python
>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```

传入一个字典，

```python
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, city=extra['city'], job=extra['job'])
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

简化调用，

```python
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

限制关键字

```python
def person(name, age, *, city, job):
    print(name, age, city, job)
```

**参数组合**

```python
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
```

### 4.3 递归函数

递归函数，本质就是栈操作。

```python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)
```

## 五、高级特性

### 5.1 slice 切片

**标准切片** ，如果是 0 开头，可以忽略

```python
>>> L[0:3]
['Michael', 'Sarah', 'Tracy']
```

**倒数切片**

```python
>>> L[-2:]
['Bob', 'Jack']
>>> L[-2:-1]
['Bob']
```

**间隔取数**

```python
>>> L[::5]
[0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
```

不仅仅 list，tuple 、 str 也可以进行切片。



### 5.2 迭代

在 Python 中，通常使用 for 循环进行迭代，

```python
>>> d = {'a': 1, 'b': 2, 'c': 3}
>>> for key in d:
...     print(key)
...
a
c
b
```

**dict 迭代**

默认情况下，dict迭代的是key。如果要迭代value，可以用`for value in d.values()`，如果要同时迭代key和value，可以用`for k, v in d.items()`。

**判断是否可迭代**

```python
>>> from collections import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

**索引对**

```python
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print(i, value)
...
0 A
1 B
2 C
```

**多变量**

```python
>>> for x, y in [(1, 1), (2, 4), (3, 9)]:
...     print(x, y)
...
1 1
2 4
3 9
```

### 5.3 列表生成器

**一个简单的例子**

```python
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

**express for**

```python
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

**+条件判断**

```python
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```

**两层循环**

```python
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

应用：列举目录下的文件名列表

```python
>>> import os # 导入os模块，模块的概念后面讲到
>>> [d for d in os.listdir('.')] # os.listdir可以列出文件和目录
['.emacs.d', '.ssh', '.Trash', 'Adlm', 'Applications', 'Desktop', 'Documents', 'Downloads', 'Library', 'Movies', 'Music', 'Pictures', 'Public', 'VirtualBox VMs', 'Workspace', 'XCode']
```

### 5.4 生成器

创建一个生成器：

```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>
```

使用 `next` 每次生成一个数字：

```python
>>> next(g)
0
>>> next(g)
1
>>> next(g)
4
>>> next(g)
9
>>> next(g)
16
>>> next(g)
25
>>> next(g)
36
>>> next(g)
49
>>> next(g)
64
>>> next(g)
81
>>> next(g)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

可以直接进行迭代：

```python
>>> g = (x * x for x in range(10))
>>> for n in g:
...     print(n)
```

带有 `yield` 的函数，本质就是一个生成器，

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
>>> f = fib(6)
>>> f
<generator object fib at 0x104feaaa0>
```

捕获异常 `StopIteration`，

```python
>>> g = fib(6)
>>> while True:
...     try:
...         x = next(g)
...         print('g:', x)
...     except StopIteration as e:
...         print('Generator return value:', e.value)
...         break
```



**`send` 方法解析**

[Python：generator的send()方法流程分析](https://www.cnblogs.com/MnCu8261/p/6527175.html)

先来一个简单的例子，

```python
def foo():
    print('starting')
    while True:
        r = yield 2
        print(r)

f = foo()
print(f.send(None))
print(f.send(1))

starting
2
1
2
```

1. `f=foo()` : 构造了一个生成器
2. `send(None)` : 相当于发送了一个空参数，然后启动生成器，等同于 `next(f)`

3. `send(1)` : 发送了一个1，给r接收到，r被打印出来，然后返回打印得到 2。

### 5.5 迭代器

直接可以进行迭代的数据类型，将他们统称为 `Iterable` 对象，

- 集合数据类型：如`list`、`tuple`、`dict`、`set`、`str`等；

- `generator`：包括生成器和带`yield` 的函数。

- `Iterable`： 可以作用于 `for`
- `Iterator`：迭代器，`list`、`dict`、`str` 都是 `Iterable` 而不是 `Iterator`

Python 的 `for` 本质上也是 `next` 实现的。 

## 参考

- 《Python 编程 从入门到实践》—— Eric Malthes【著】 袁国忠【译】
- [廖雪峰的官方网站 - Python 教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017063826246112)

- [RealPython](https://realpython.com/)

- [Python：generator的send()方法流程分析](https://www.cnblogs.com/MnCu8261/p/6527175.html)

