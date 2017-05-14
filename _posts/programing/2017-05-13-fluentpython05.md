---
nav: blog
layout: post
title: "流畅的python - 一等函数"
author: "wangchao"
tags:
  - python
  - 'function'
  - '函数'
  - '内置函数'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- **一等对象:**
  - 在运行时创建
  - 能赋值给变量或数据结构中的元素
  - 能作为参数传递给函数
  - 能作为函数的返回结果.
  - 【在 Python 中，所有函数都是一等对象。】
- **函数**
  - 每个函数都有`__doc__` 属性，用于生成函数的帮助文档.
  - 函数是 `function` 类的实例.
  - **高级函数：**
    - 接受函数为参数，或者把函数作为结果返回的函数。 如内置函数： `map` 、 `sorted` 、 `filter` 、`reduce` 等等.
    - 现在的 列表推导和生成器表达式可以代替 `map` 、 `filter` 、 `reduce` 等函数. 并且更易读. [python3中，`reduce` 函数方法到了`functools`模块中.]
  - **归约函数:**
    - 把某个操作连续应用到序列的元素上，累计之前的结果，把一系列值归约成一个值。 如: `sum` 、 `reduce` . 其他的还有： `all` 和 `any`
    - **all(iterable):** 如果 `iterable` 的每个元素都是真值，返回 `True`； `all([])` 返回 `True`。
    - **any(iterable):** 如果 `iterable` 中至少有一个元素是真值，就返回 `True`；`any([])` 返回 `False`。
  - **匿名函数:**
    - 在 Python 表达式内用 `lambda` 关键字创建匿名函数。 并且 `lambda` 函数的定义体只能使用纯表达式。【lambda 函数的定义体中不能赋值，也不能使用 while 和 try 等 Python 语句。】
    - 在参数列表中最适合使用匿名函数。 如： `sorted(fruits, key=lambda word: word[::-1])`
    - `lambda` 句法只是语法糖：与 `def` 语句一样，`lambda` 表达式会创建函数对象。

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=lambda word: word[::-1])
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
```

- **可调用对象**
  - 如果想判断对象能否调用，可以使用内置的 `callable()` 函数。 Python 数据模型文档列出了 7 种可调用对象。
  - **用户定义的函数：** 使用 `def` 语句或 `lambda` 表达式创建。
  - **内置函数：** 使用 C 语言（CPython）实现的函数，如 `len` 或 `time.strftime`。
  - **内置方法：** 使用 C 语言实现的方法，如 `dict.get`。
  - **类方法：** 在类的定义体中定义的函数。
  - **类：** 调用类时会运行类的 `__new__` 方法创建一个实例，然后运行 `__init__` 方法，初始化实例，最后把实例返回给调用方。
  - **类的实例：** 如果类定义了 `__call__` 方法，那么它的实例可以作为函数调用。
  - **生成器函数：** 使用 `yield` 关键字的函数或方法。调用生成器函数返回的是生成器对象。
- **自定义可调用类型**
  - 不仅 Python 函数是真正的对象，任何 Python 对象都可以表现得像函数。为此，只需实现实例方法 __call__。

```python
In [4]: abs, str, 13
Out[4]: (<function abs>, str, 13)

In [5]: [callable(obj) for obj in (abs, str, 13)]
Out[5]: [True, True, False]

# 自定义可调用对象
import random

class BingoCage:

    def __init__(self, items):
        self._items = list(items)  # 接受任何可迭代对象；在本地构建一个副本，防止列表参数的意外副作用。
        random.shuffle(self._items)  # 随机排序

    def pick(self):
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')

    def __call__(self):  # 快捷方式.
        return self.pick()

# 应用：
In [9]: bingo = BingoCage(range(3))

In [10]: bingo.pick()
Out[10]: 2

In [11]: bingo()
Out[11]: 1

In [12]: callable(bingo)
Out[12]: True
```

- **函数内省**
  - 函数对象有很多属性。使用 `dir` 函数可以探知, 其中大多数属性是 Python 对象共有的.
  - `__dict__` : 存储赋予它的用户属性. 这相当于一种基本形式的注解。一般来说，为函数随意赋予属性不是很常见的做法，但是 `Django` 框架这么做了。 参考: [The Django admin site](https://docs.djangoproject.com/en/1.10/ref/contrib/admin/)

```python
def upper_case_name(obj):
    return ("%s %s" % (obj.first_name, obj.last_name)).upper()
upper_case_name.short_description = 'Customer name'  # Django 管理后台使用这个方法时，在记录列表中会出现指定的描述文本

# 函数专有而用户定义的一般对象没有的属性
In [13]: class C:pass

In [14]: obj = C()

In [15]: def func():pass

In [16]: sorted(set(dir(func)) - set(dir(obj))) # 计算差集，然后排序，得到类的实例没有而函数有的属性列表
Out[16]:
['__annotations__',
 '__call__',
 '__closure__',
 '__code__',
 '__defaults__',
 '__get__',
 '__globals__',
 '__kwdefaults__',
 '__name__',
 '__qualname__']
```

**名称** | **类型** | **说明**
---------|----------|-----------
`__annotations__` | `dict` | 参数和返回值的注解
`__call__` | `method-wrapper` | 实现 `()` 运算符；即可调用对象协议
`__closure__` | `tuple`  | 函数闭包，即自由变量的绑定（通常是 `None`）
`__code__` | `code` | 编译成字节码的函数元数据和函数定义体
`__defaults__` | `tuple` | 形式参数的默认值
`__get__` | `method-wrapper` | 实现只读描述符协议（参见第 20 章）
`__globals__` | `dict` | 函数所在模块中的全局变量
`__kwdefaults__` | `dict` | 仅限关键字形式参数的默认值
`__name__` | `str` | 函数名称
`__qualname__` | `str` | 函数的限定名称，如 `Random.choice`（ 参阅[PEP 3155](https://www.python.org/dev/peps/pep-3155/)）
