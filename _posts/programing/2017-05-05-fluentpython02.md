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

**切片:**

```python
>>> l = [10, 20, 30, 40, 50, 60]
>>> l[:2] # 在下标2的地方分割
[10, 20]
>>> l[2:]
[30, 40, 50, 60]
>>> l[:3] # 在下标3的地方分割
[10, 20, 30]
 >>> l[3:]
[40, 50, 60]
```

- 当下标为0且不包含最后一个元素时，可以快速确定要取得元素个数：如：`range(3)` 和 `my_list[:3]`
- 可以快速根据 end 和 start 确定 元素个数： (`stop - start`)
- 快速分割成不叠加的两个部分： `my_list[:x]` 和 `my_list[x:]`

**间隔切片：**

用 `s[a:b:c]` 的形式对 `s` 在 `a` 和 `b` 之间以 `c` 为间隔取值。`c` 的值还可以为负，负值意味着反向取值。

```python
>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'
```

`a:b:c` 这种用法只能作为索引或者下标用在 `[]` 中来返回一个切片对象：`slice(a, b, c)`

对 `seq[start:stop:step]` 进行求值的时候，Python 会调用 `seq.__getitem__(slice(start, stop, step))`

```python
>>> invoice = """
... 0.....6................................40........52...55........
... 1909  Pimoroni PiBrella                    $17.50    3    $52.50
... 1489  6mm Tactile Switch x20                $4.95    2     $9.90
... 1510  Panavise Jr. - PV-201                $28.00    1    $28.00
... 1601  PiTFT Mini Kit 320x240               $34.95    1    $34.95
... """
>>> SKU = slice(0, 6)
>>> DESCRIPTION = slice(6, 40)
>>> UNIT_PRICE = slice(40, 52)
>>> QUANTITY = slice(52, 55)
>>> ITEM_TOTAL = slice(55, None)
>>> line_items = invoice.split('\n')[2:]
>>> for item in line_items:
...     print(item[UNIT_PRICE], item[DESCRIPTION])
...
    $17.50   Pimoroni PiBrella
     $4.95   6mm Tactile Switch x20
    $28.00   Panavise Jr. - PV-201
    $34.95   PiTFT Mini Kit 320x240
```

**numpy中的多维切片**

形如 `a[i, j]` 或 `a[m:n, k:l]` 这种形式，python对以元组的方式传入切片对象，`a.__getitem__((i, j))` , 但 Python 内置的序列类型都是一维的，因此它们只支持单一的索引，成对出现的索引是没有用的。

更多参考[Tentative NumPy Tutorial](http://scipy.github.io/old-wiki/pages/Tentative_NumPy_Tutorial)

**给切片赋值**

如果把切片放在赋值语句的左边，或作为 `del` 操作的对象，就可以对序列进行嫁接、切除或就地修改操作.

```python
In [6]: l = list(range(10))
In [7]: l
Out[7]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
In [8]: l[2:5] = [20,30]  # iterable 的个数无关
In [9]: l
Out[9]: [0, 1, 20, 30, 5, 6, 7, 8, 9]
In [10]: l[2:5] = [20,30,40] # iterable 的个数无关
In [11]: l
Out[11]: [0, 1, 20, 30, 40, 6, 7, 8, 9]
In [14]: del l[2:5]  # 删除指定区间
In [15]: l
Out[15]: [0, 1, 50, 6, 7, 8, 9]
In [16]: l[3::2] = [11,12]  # 间隔赋值
In [17]: l
Out[17]: [0, 1, 50, 11, 7, 12, 9]
In [18]: l[3::2] = [11,12,13,14,15]  # 间隔赋值不能越界
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-18-132b3ef5c1f6> in <module>()
----> 1 l[3::2] = [11,12,13,14,15]

ValueError: attempt to assign sequence of size 5 to extended slice of size 2

In [19]: l[2:5] = 100  # 赋值的对象必须是 iterable
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-19-e500d102d83b> in <module>()
----> 1 l[2:5] = 100

TypeError: can only assign an iterable

In [20]: l[2:5] = [100]

In [21]: l
Out[21]: [0, 1, 100, 12, 9]
```

**序列中的 `+` 和 `*` 操作符**

默认序列是支持 `+` 和 `*` 操作的。通常 `+` 号两侧的序列由相同类型的数据所构成，在拼接的过程中，两个被操作的序列都不会被修改，Python 会新建一个包含同样类型数据的序列来作为拼接的结果。

```python
# 把一个序列复制几份然后再拼接起来，更快捷的做法是把这个序列乘以一个整数
>>> l = [1, 2, 3]
>>> l * 5
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]
>>> 5 * 'abcd'
'abcdabcdabcdabcdabcd'
# + 和 * 都遵循这个规律，不修改原有的操作对象，而是构建一个全新的序列。
```

如果在 `a * n` 这个语句中，序列 `a` 里的元素是对其他可变对象的引用的话，就需要格外注意了，因为这个式子的结果可能会出乎意料。

比如，想用 my_list = [[]] * 3 来初始化一个由列表组成的列表，但是你得到的列表里包含的 3 个元素其实是 3 个引用，而且这 3 个引用指向的都是同一个列表。

可以用列表推导代替.

```python
>>> board = [['_'] * 3 for i in range(3)]  # 建立一个包含 3 个列表的列表，被包含的 3 个列表各自有 3 个元素。
>>> board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> board[1][2] = 'X' # 把第 1 行第 2 列的元素标记为 X，再打印出这个列表。
>>> board
[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]

# 看似更快捷的方法，但是都是指向的同一个对象。
>>> weird_board = [['_'] * 3] * 3 # 其实包含 3 个指向同一个列表的引用
>>> weird_board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> weird_board[1][2] = 'O' # 试图标记第 1 行第 2 列的元素，就立马暴露了列表内的 3 个引用指向同一个对象.
>>> weird_board
[['_', '_', 'O'], ['_', '_', 'O'], ['_', '_', 'O']]
```

**序列的增量赋值：`+=` 和 `*=`**

`+=` 背后的特殊方法是 `__iadd__` （用于“就地加法”）。但是如果一个类没有实现这个方法的话，Python 会退一步调用 `__add__`.

对于 `a += b`

如果 `a` 实现了 `__iadd__` 方法，就会调用这个方法。同时对可变序列（例如 `list`、`bytearray` 和 `array.array`）来说，`a` 会就地改动，就像调用了 `a.extend(b)` 一样。但是如果 `a` 没有实现 `__iadd__` 的话，`a += b` 这个表达式的效果就变得跟 `a = a + b` 一样了：首先计算 `a + b`，得到一个新的对象，然后赋值给 `a`。

`*=` 也同 `+=` 一样，但是特殊方法是`__imul__`。

```python
In [22]: l = [1,2]

In [23]: id(l)
Out[23]: 4506049352

In [24]: l *= 2  # 对象地址不会改变,对于可变序列

In [25]: l
Out[25]: [1, 2, 1, 2]

In [26]: id(l)
Out[26]: 4506049352

In [27]: l = l + [2,3]  # 会生成新的对象并赋值地址. 对于可变序列

In [28]: l
Out[28]: [1, 2, 1, 2, 2, 3]

In [29]: id(l)
Out[29]: 4506147080

In [30]: l += [99] # 对象地址不会改变， 对于可变序列

In [31]: l
Out[31]: [1, 2, 1, 2, 2, 3, 99]

In [32]: id(l)
Out[32]: 4506147080

In [33]: t = (1,2,3) # 不可变序列

In [34]: id(t)
Out[34]: 4504761184

In [35]: t *= 2  # 对象地址会改变, 对于不可变序列

In [36]: t
Out[36]: (1, 2, 3, 1, 2, 3)

In [37]: id(t)
Out[37]: 4505756008

In [38]: t += (4,5)  # 对象地址会改变, 对于不可变序列

In [39]: t
Out[39]: (1, 2, 3, 1, 2, 3, 4, 5)

In [40]: id(t)
Out[40]: 4505767112

# str 是一个例外，因为对字符串做 += 实在是太普遍了，所以 CPython 对它做了优化。
# 为 str 初始化内存的时候，程序会为它留出额外的可扩展空间，因此进行增量操作的时候，并不会涉及复制原有字符串到新位置这类操作。
```

**`+=`对于不可变序列的边界问题**

```python
>>> t = (1, 2, [30, 40])
>>> t[2] += [50, 60]
```

到底会发生下面 4 种情况中的哪一种？

`a`. `t` 变成 `(1, 2, [30, 40, 50, 60])`。

`b`. 因为 `tuple` 不支持对它的元素赋值，所以会抛出 `TypeError` 异常。

`c`. 以上两个都不是。

`d`. `a` 和 `b` 都是对的。

实际是 `d` 选项.

如：

```python
In [41]: t = (1,2,[30,40])

In [42]: t[2] += [50,60]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-42-fb586dd7f384> in <module>()
----> 1 t[2] += [50,60]

TypeError: 'tuple' object does not support item assignment

In [43]: t
Out[43]: (1, 2, [30, 40, 50, 60])

# 背后的字节码：
In [47]: import dis

In [48]: dis.dis('s[a] += b')
  1           0 LOAD_NAME                0 (s)
              2 LOAD_NAME                1 (a)
              4 DUP_TOP_TWO
              6 BINARY_SUBSCR                   # 将 s[a] 的值存入 TOS（Top Of Stack，栈的顶端）。
              8 LOAD_NAME                2 (b)
             10 INPLACE_ADD                     # 计算 TOS += b。这一步能够完成，是因为 TOS 指向的是一个可变对象（也就是元组中的列表）
             12 ROT_THREE
             14 STORE_SUBSCR                    # s[a] = TOS 赋值。这一步失败，是因为 s 是不可变的元组（也就是元组 t）。
             16 LOAD_CONST               0 (None)
             18 RETURN_VALUE
```

3 个教训:

- 不要把可变对象放在元组里面。
- 增量赋值不是一个原子操作。它虽然抛出了异常，但还是完成了操作。
- 查看 Python 的字节码并不难，而且它对我们了解代码背后的运行机制很有帮助。

**排序**

内置方法 `sorted` 和 列表自带方法 `list.sort` 之间排序的区别：

- 是否会复制原有列表： `sorted` 会，返回排序后的复制过的列表. `list.sort` 不会,会就地排序.但返回None. 【Python 的一个惯例：如果一个函数或者方法对对象进行的是就地改动，那它就应该返回 None，好让调用者知道传入的参数发生了变动，而且并未产生新的对象。】
- 共同的参数：
  - `key` ： 一个只有一个参数的函数，这个函数会被用在序列里的每一个元素上，所产生的结果将是排序算法依赖的对比关键字。 如：`key=str.lower` 、`key=len`
  - `reverse` ：如果被设定为 True，被排序的序列里的元素会以降序输出（也就是说把最大值当作最小值来排序）。这个参数的默认值是 False。
  - 【可选参数 `key` 还可以在内置函数 `min()` 和 `max()` 中起作用。另外，还有些标准库里的函数也接受这个参数，像 `itertools.groupby()` 和 `heapq.nlargest()` 等。】

```python
In [53]: fruits = ['grape', 'raspberry', 'apple', 'banana']

In [54]: fruits.sort()   # 就地排序

In [55]: fruits
Out[55]: ['apple', 'banana', 'grape', 'raspberry']

In [56]: sorted(fruits,key=len)   # 根据长度排序
Out[56]: ['apple', 'grape', 'banana', 'raspberry']

In [58]: sorted(fruits,key=len,reverse=True)  # 从大到小.
Out[58]: ['raspberry', 'banana', 'apple', 'grape']

In [61]: fruits.sort(key=len,reverse=True)  # 就地排序，并且根据字符长度，以及从大到小

In [62]: fruits
Out[62]: ['raspberry', 'banana', 'apple', 'grape']
```
