# Python-Web 项目

## 前言

本项目来源：[廖雪峰-Python教程-实战](https://www.liaoxuefeng.com/wiki/1016959663602400/1018138223191520)

通过个人，对该项目进行复现学习，并作一些笔记。

## 一、搭建开发环境

**开发环境：**

- Pyhton 版本： 3.7.x
- aiohttp : 异步框架
- jinja2 ：前端模板引擎
- aiomysql：MySQL 的异步驱动程序

**项目结构：**

```
awesome-python3-webapp/  <-- 根目录
|
+- backup/               <-- 备份目录
|
+- conf/                 <-- 配置文件
|
+- dist/                 <-- 打包目录
|
+- www/                  <-- Web目录，存放.py文件
|  |
|  +- static/            <-- 存放静态文件
|  |
|  +- templates/         <-- 存放模板文件
|
+- ios/                  <-- 存放iOS App工程
|
+- LICENSE               <-- 代码LICENSE
```

## 二、第一个 Web APP 项目

通过复现官网的项目会发现，有的 API 弹出用法被废弃，但依然可以正常使用。

直接使用 `Chrome`  浏览器打开会下载得到一个包含网页文本的文件，需要修改，加入 `content_type` 即可正常运行，`return web.Response(body=b'<h1>Awesome</h1>', content_type='text/html')`

于是完整的代码如下所示，

```python
import logging; logging.basicConfig(level=logging.INFO)
import asyncio, os, json,time
from datetime import datetime
from aiohttp import web

def index(request):
    return web.Response(body=b'<h1>Awesome</h1>', content_type='text/html')

@asyncio.coroutine
def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET','/',index)
    srv = yield from loop.create_server(app.make_handler(),'127.0.0.1',9000)
    logging.info('server started at http://127.0.0.1:9000...')
    return srv

loop = asyncio.get_event_loop()
loop.run_until_complete(init(loop))
loop.run_forever()
```

`loop` ：是 `asyncio` 异步I/O 模块中的一个事件循环，

通过这个事件循环作为参数，传入给 `init ` 去初始化一个 Web 程序，然后 Web 程序监听本地 9000 端口

**index 函数：**

使用了 `request` 参数为，`aiohttp.request`

其中，引入了 `aiohttp.web` 为 `web` ，使用了 `Response`

声明如下：`class aiohttp.web.Response(*, status=200, headers=None, content_type=None, body=None, text=None)`

**init 函数：**

这是一个协程，其中等待 `srv` 去创建服务器的事件，然后将 `srv` 返回。

`loop.create_server()`则利用`asyncio`创建TCP服务。

**loop 循环启动**

然后引用 `asyncio` 模块中的事件循环，然后处理 Web 服务器事件直到结束。

## 三、ORM 框架

对于 `MySQL` 数据库操作，我们采用 `aiomysql` 驱动进行异步操作。

### 3.1 创建连接池

如果我们每一次数据库操作都需要连接、释放，那势必带来巨大的资源损耗。

我们可以把可用的连接放到一个全局的连接池里，供使用者重复利用。

```python
@asyncio.coroutine
def create_pool(loop, **kw):
    logging.info('create database connection pool...')
    global __pool
    __pool = yield from aiomysql.create_pool(
        host=kw.get('host', 'localhost'),
        port=kw.get('port', 3306),
        user=kw['user'],
        password=kw['password'],
        db=kw['db'],
        charset=kw.get('charset', 'utf8'),
        autocommit=kw.get('autocommit', True),
        maxsize=kw.get('maxsize', 10),
        minsize=kw.get('minsize', 1),
        loop=loop
    )
```

`create_pool` ：用来创建一个 mysql 连接池，里面输入一些需要的参数，如果没有有些会补充默认参数。



### 3.2 SELECT 语句

要执行 SELECT 语句，我们需要传入 SQL 语句和参数。

```python
@asyncio.coroutine
def select(sql, args, size=None):
    log(sql, args)
    global __pool
    with (yield from __pool) as conn:
        cur = yield from conn.cursor(aiomysql.DictCursor)
        yield from cur.execute(sql.replace('?', '%s'), args or ())
        if size:
            rs = yield from cur.fetchmany(size)
        else:
            rs = yield from cur.fetchall()
        yield from cur.close()
        logging.info('rows returned: %s' % len(rs))
        return rs
```

> SQL语句的占位符是`?`，而MySQL的占位符是`%s`，`select()`函数在内部自动替换。注意要始终坚持使用带参数的SQL，而不是自己拼接SQL字符串，这样可以防止SQL注入攻击。

函数流程如下：

1. 从连接池取到一个连接 `conn`
2. 以字典形式去取游标  `conn.cursor` ,即 `cur`
3. `cur.execute(sql.replace('?', '%s'), args or ())` 去执行一条语句
4. 根据 `size` 判断取出的条数
5. 等待关闭 `cur`
6. 返回取到的 `rs` 资源集



### 3.3 Insert, Update, Delete

要执行语句，可以定义一个通用的 `execute() `函数，然后返回一个影响了多少行的语句。

```python
@asyncio.coroutine
def execute(sql, args):
    log(sql)
    with (yield from __pool) as conn:
        try:
            cur = yield from conn.cursor()
            yield from cur.execute(sql.replace('?', '%s'), args)
            affected = cur.rowcount
            yield from cur.close()
        except BaseException as e:
            raise
        return affected
```

 函数的流程如下：

1. 取出一个 `conn` 连接

2. 取出游标，等待执行

3. `cur.rowcount` 得到影响的行数，返回

   

   

### 3.4 ORM

定义一个 `User` 对象，并把它跟 `users` 表关联起来，

```python
from orm import Model, StringField, IntegerField

class User(Model):
    __table__ = 'users'

    id = IntegerField(primary_key=True)
    name = StringField()
```

**Field 类以及其子类**

```python
class Field(object):

    def __init__(self, name, column_type, primary_key, default):
        self.name = name
        self.column_type = column_type
        self.primary_key = primary_key
        self.default = default

    def __str__(self):
        return '<%s, %s:%s>' % (self.__class__.__name__, self.column_type, self.name)
```
**StringField**

```python
class StringField(Field):

    def __init__(self, name=None, primary_key=False, default=None, ddl='varchar(100)'):
        super().__init__(name, ddl, primary_key, default)
```



**Model** : 继承于 `dict` 字典，

```python
class Model(dict, metaclass=ModelMetaclass):

    def __init__(self, **kw):
        super(Model, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

    def getValue(self, key):
        return getattr(self, key, None)

    def getValueOrDefault(self, key):
        value = getattr(self, key, None)
        if value is None:
            field = self.__mappings__[key]
            if field.default is not None:
                value = field.default() if callable(field.default) else field.default
                logging.debug('using default value for %s: %s' % (key, str(value)))
                setattr(self, key, value)
        return value
```



**Meta Class ：ModelMetaClass**

`__new__` 方法在类定义的时候被调用去构建类，

1. 排除 Model 类本身
2. table 表格名称获取，从 `__table__` 参数，或者类名
3. 查找主键
4. 获取 `fields` 名称
5. 使用 `fields` 名称构建成为 

```python
class ModelMetaclass(type):

    def __new__(cls, name, bases, attrs):
        # 排除Model类本身:
        if name=='Model':
            return type.__new__(cls, name, bases, attrs)
        # 获取table名称:
        tableName = attrs.get('__table__', None) or name
        logging.info('found model: %s (table: %s)' % (name, tableName))
        # 获取所有的Field和主键名:
        mappings = dict()
        fields = []
        primaryKey = None
        for k, v in attrs.items():
            if isinstance(v, Field):
                logging.info('  found mapping: %s ==> %s' % (k, v))
                mappings[k] = v
                if v.primary_key:
                    # 找到主键:
                    if primaryKey:
                        raise RuntimeError('Duplicate primary key for field: %s' % k)
                    primaryKey = k
                else:
                    fields.append(k)
        if not primaryKey:
            raise RuntimeError('Primary key not found.')
        for k in mappings.keys():
            attrs.pop(k)
        escaped_fields = list(map(lambda f: '`%s`' % f, fields))
        attrs['__mappings__'] = mappings # 保存属性和列的映射关系
        attrs['__table__'] = tableName
        attrs['__primary_key__'] = primaryKey # 主键属性名
        attrs['__fields__'] = fields # 除主键外的属性名
        # 构造默认的SELECT, INSERT, UPDATE和DELETE语句:
        attrs['__select__'] = 'select `%s`, %s from `%s`' % (primaryKey, ', '.join(escaped_fields), tableName)
        attrs['__insert__'] = 'insert into `%s` (%s, `%s`) values (%s)' % (tableName, ', '.join(escaped_fields), primaryKey, create_args_string(len(escaped_fields) + 1))
        attrs['__update__'] = 'update `%s` set %s where `%s`=?' % (tableName, ', '.join(map(lambda f: '`%s`=?' % (mappings.get(f).name or f), fields)), primaryKey)
        attrs['__delete__'] = 'delete from `%s` where `%s`=?' % (tableName, primaryKey)
        return type.__new__(cls, name, bases, attrs)
```

