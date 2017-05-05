---
nav: blog
layout: post
title: "流畅的python - 序列构成的数组"
author: "wangchao"
tags:
  - python
  - '数据模型'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

**容器序列** ： `list`、`tuple` 和 `collections.deque` 这些序列能存放不同类型的数据。 【存放的是它们所包含的任意类型的对象的引用】

**扁平序列** : `str`、`bytes`、`bytearray`、`memoryview` 和 `array.array`，这类序列只能容纳一种类型。 【存放的是值而不是引用，其实是一段连续的内存空间，只能存放诸如字符、字节和数值这种基础类型。】

**可变序列** ： `list`、`bytearray`、`array.array`、`collections.deque` 和 `memoryview`.

**不可变序列** : `tuple`、`str` 和 `bytes`。

Python 会忽略代码里 []、{} 和 () 中的换行，因此如果代码里有多行的列表、列表推导、生成器表达式、字典这一类的，可以省略不太好看的续行符 \\。

在python2中，列表推导可能会有变量泄露问题：

```python
Python 2.7.6 (default, Mar 22 2014, 22:59:38)
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> x = 'my precious'
>>> dummy = [x for x in 'ABC']
>>> x
'C'
```

在python3中，则不会：

```python
Python 3.6.1 (default, Apr  4 2017, 09:40:21)
Type "copyright", "credits" or "license" for more information.

IPython 5.3.0 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.
In [7]: x = 'my precious'
In [8]: dummy = [x for x in 'ABC']
In [9]: x
Out[9]: 'my precious'
In [10]: dummy
Out[10]: ['A', 'B', 'C']
```

列表推导可以把一个序列或是其他可迭代类型中的元素过滤或是加工，然后再新建一个列表。

**关于列表推导和filter以及map的性能：**

```python
import timeit

TIMES = 10000

SETUP = """
symbols = '$¢£¥€¤'
def non_ascii(c):
    return c > 127
"""

def clock(label, cmd):
    res = timeit.repeat(cmd, setup=SETUP, number=TIMES)  # 默认重复3次
    print(label, *('{:.3f}'.format(x) for x in res))

clock('listcomp        :', '[ord(s) for s in symbols if ord(s) > 127]')
clock('listcomp + func :', '[ord(s) for s in symbols if non_ascii(ord(s))]')
clock('filter + lambda :', 'list(filter(lambda c: c > 127, map(ord, symbols)))')
clock('filter + func   :', 'list(filter(non_ascii, map(ord, symbols)))')

# listcomp        : 0.015 0.017 0.017
# listcomp + func : 0.023 0.023 0.022
# filter + lambda : 0.020 0.020 0.020
# filter + func   : 0.021 0.021 0.019
```

关于`timeit模块` 参考：[python3-库参考](http://python.usyiyi.cn/translate/python_352/library/timeit.html)

**根据列表推导的顺序，可以根据不同的值来排序**

```python
In [6]: colors = ['black', 'white']
In [7]: sizes = ['S', 'M', 'L']
In [8]: tshirts = [(color, size) for color in colors for size in sizes]  # 根据color来排序
In [9]: tshirts
Out[9]:
[('black', 'S'),
 ('black', 'M'),
 ('black', 'L'),
 ('white', 'S'),
 ('white', 'M'),
 ('white', 'L')]
In [12]: tshirts = [(color, size) for size in sizes for color in colors]  # 根据size来排序
In [13]: tshirts
Out[13]:
[('black', 'S'),
 ('white', 'S'),
 ('black', 'M'),
 ('white', 'M'),
 ('black', 'L'),
 ('white', 'L')]
```

**生成器表达式:** 生成器表达式的语法跟列表推导差不多，只不过把方括号换成圆括号而已。

```python
>>> symbols = '$¢£¥€¤'
>>> tuple(ord(symbol) for symbol in symbols)  # 如果生成器表达式是一个函数调用过程中的唯一参数，那么不需要额外再用括号把它围起来。
(36, 162, 163, 165, 8364, 164)
>>> import array
>>> array.array('I', (ord(symbol) for symbol in symbols))  # array 的构造方法需要两个参数，因此括号是必需的。
array('I', [36, 162, 163, 165, 8364, 164])

# 使用生成器计算笛卡尔集：[当含有大量数据时，可以节省内存.]
In [14]: for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes):
    ...:     print(tshirt)
    ...:
black S
black M
black L
white S
white M
white L
```

**元组除了是不可变的列表，还充当了记录数据的功能，位置和数据相对应**

元组拆包,python3 支持. 参考[pep-3132](https://www.python.org/dev/peps/pep-3132/)

```python
In [15]: lax_coordinates = (33, -118)
In [16]: latitude, longitude = lax_coordinates
In [17]: latitude
Out[17]: 33
In [18]: longitude
Out[18]: -118
```

**占位符 _ :** 如果做的是国际化软件，那么 _ 可能就不是一个理想的占位符，因为它也是 gettext.gettext 函数的常用别名，gettext 模块的[文档](https://docs.python.org/3/library/gettext.html)里提到了这一点。在其他情况下，_ 会是一个很好的占位符。

**用*来处理剩下的元素: python3中.**

```python
In [19]: a, b, *rest = range(5)
In [20]: a,b,rest
Out[20]: (0, 1, [2, 3, 4])
In [21]: a, b, *rest = range(3)
In [22]: a,b,rest
Out[22]: (0, 1, [2])
In [23]: a, b, *rest = range(2)
In [24]: a,b,rest
Out[24]: (0, 1, [])

# 在赋值中，* 前缀只能用在一个变量名前面，但是这个变量可以出现在赋值表达式的任意位置
In [25]: a, *body, c, d = range(5)
In [26]: a,body,c,d
Out[26]: (0, [1, 2], 3, 4)
In [27]: *head,b,c,d= range(5)
In [28]: head,b,c,d
Out[28]: ([0, 1], 2, 3, 4)
```

**命名元组:namedtuple**

```python
>>> from collections import namedtuple
# 创建一个具名元组需要两个参数，一个是类名，另一个是类的各个字段的名字。
# 后者可以是由数个字符串组成的可迭代对象，或者是由空格分隔开的字段名组成的字符串。
>>> City = namedtuple('City', 'name country population coordinates')  
# 存放在对应字段里的数据要以一串参数的形式传入到构造函数中（注意，元组的构造函数却只接受单一的可迭代对象）。
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))  #
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722, 139.691667))
 # 可以通过字段名或者位置来获取一个字段的信息。
>>> tokyo.population
36.933
>>> tokyo.coordinates
(35.689722, 139.691667)
>>> tokyo[1]
'JP'
```

**从普通元组那里继承来的属性之外，具名元组还有一些自己专有的属性：**

```python
>>> City._fields  # 类属性
('name', 'country', 'population', 'coordinates')
>>> LatLong = namedtuple('LatLong', 'lat long')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.613889, 77.208889))
>>> delhi = City._make(delhi_data)   # 类方法,接受一个可迭代对象来生成这个类的一个实例
>>> delhi._asdict()  # 实例方法, 以 collections.OrderedDict 的形式返回
OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population',
21.935), ('coordinates', LatLong(lat=28.613889, long=77.208889))])
>>> for key, value in delhi._asdict().items():
        print(key + ':', value)
name: Delhi NCR

country: IN
population: 21.935
coordinates: LatLong(lat=28.613889, long=77.208889)
>>>
```

**列表和元组的一些区别：**

**方法** | **列表支持度** | **元组支持度** | **说明**
---------|---------------|---------------|-----------
`s.__getnewargs__()` | | • | 在 pickle 中支持更加优化的序列化
`s.__add__(s2)` | • | • | `s + s2`，拼接
`s.__contains__(e)` | • | • | `s` 是否包含 `e`
`s.index(e)` | • | • | 在 `s` 中找到元素 `e` 第一次出现的位置
`s.count(e)` | • | • | `e` 在 `s` 中出现的次数
`s.__getitem__(p)` | •  | • | `s[p]`，获取位置`p` 的元素
`s.__iter__()` | • | • | 获取 `s` 的迭代器
`s.__len__()` | • | •  | `len(s)`，元素的数量
`s.__mul__(n)` | • | • | `s * n`，`n` 个 `s` 的重复拼接
`s.__rmul__(n)` | • | • |  `n * s`，反向拼接 *
`s.__delitem__(p)` | • | | 把位于 `p` 的元素删除
`s.__reversed__()` | • |  | 返回 `s` 的倒序迭代器
`s.__setitem__(p, e)` | • | | `s[p] = e`，把元素 `e `放在位置`p`，替代已经在那个位置的元素
`s.__imul__(n)` | • | | `s *= n`，就地重复拼接
`s.__iadd__(s2)` | • |  | `s += s2`，就地拼接
`s.insert(p, e)` | • | | 在位置 `p` 之前插入元素`e`
`s.append(e)` | • |  | 在尾部添加一个新元素
`s.clear()` |  • | | 删除所有元素
`s.copy()` | • | | 列表的浅复制
`s.extend(it)` | • | | 把可迭代对象 `it` 追加给 `s`
`s.pop([p])` | • | | 删除最后或者是（可选的）位于 `p` 的元素，并返回它的值
`s.remove(e)` | • | | 删除 `s` 中的第一次出现的 `e`
`s.reverse()` | • | | 就地把 `s` 的元素倒序排列
`s.sort([key], [reverse])` | • | | 就地对 `s` 中的元素进行排序，可选的参数有键（`key`）和是否倒序（`reverse`）
