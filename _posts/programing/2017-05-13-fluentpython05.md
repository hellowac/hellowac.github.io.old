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

- [可调用对象](#可调用对象)
- [函数内省](#函数内省)
- [仅限关键字参数](#仅限关键字参数)
- [函数剖析](#函数剖析)
- [函数注解](#函数注解)
- [常用的包](#常用的包)
- [冻结参数](#冻结参数)

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

- **可调用对象** <span id="可调用对象"></span>
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

- **函数内省** <span id="函数内省"></span>
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

- **仅限关键字参数** <span id="仅限关键字参数"></span>

```python
def tag(name, *content, cls=None, **attrs):
    """生成一个或多个HTML标签"""
    if cls is not None:
        attrs['class'] = cls
    if attrs:
        attr_str = ''.join(' %s="%s"' % (attr, value)
                           for attr, value
                           in sorted(attrs.items()))
    else:
        attr_str = ''
    if content:
        return '\n'.join('<%s%s>%s</%s>' %
                         (name, attr_str, c, name) for c in content)
    else:
        return '<%s%s />' % (name, attr_str)

# 应用：
In [2]: tag('br')  # 传入单个定位参数，生成一个指定名称的空标签。
Out[2]: '<br />'

In [3]: tag('br','hello')  # 第一个参数后面的任意个参数会被 *content 捕获，存入一个元组。
Out[3]: '<br>hello</br>'

In [4]: print(tag('p','hello','world')) #
<p>hello</p>
<p>world</p>

In [5]: tag('p', 'hello', id=33)  # tag 函数签名中没有明确指定名称的关键字参数会被 **attrs 捕获，存入一个字典。
Out[5]: '<p id="33">hello</p>'

In [6]: print(tag('p','hello','world',cls='sidebar'))  # cls 参数只能作为关键字参数传入。
<p class="sidebar">hello</p>
<p class="sidebar">world</p>

In [7]: tag(content="testing",name="img")   # 调用 tag 函数时，即便第一个定位参数也能作为关键字参数传入。
Out[7]: '<img content="testing" />'

In [8]: my_tag = {'name':'img', 'title':'Sunset Boulevard',
   ...:           'src':'sunset.jpg','cls':'framed'}

In [9]: tag(**my_tag)  # 在 my_tag 前面加上 **，字典中的所有元素作为单个参数传入，同名键会绑定到对应的具名参数上，余下的则被 **attrs 捕获。
Out[9]: '<img class="framed" src="sunset.jpg" title="Sunset Boulevard" />'
```

定义函数时若想指定仅限关键字参数，要把它们放到前面有 `*` 的参数后面。如果不想支持数量不定的定位参数，但是想支持仅限关键字参数，在签名中放一个 `*` :

```python
>>> def f(a, *, b):  # 仅限关键字参数不一定要有默认值,强制必须传入实参.
...     return a, b
...
>>> f(1, b=2)
(1, 2)
```

参考 [廖雪峰-函数的参数](#http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431752945034eb82ac80a3e64b9bb4929b16eeed1eb9000) 和 [python3-cookbook](http://python3-cookbook.readthedocs.io/zh_CN/latest/c07/p02_functions_that_only_accept_keyword_arguments.html)

- **函数剖析** <span id="函数剖析"></span>
  - 函数对象有个 `__defaults__` 属性，它的值是一个元组，里面保存着定位参数和关键字参数的默认值。
  - 仅限关键字参数的默认值在 `__kwdefaults__` 属性中。
  - 参数的名称在 `__code__` 属性中，它的值是一个 `code` 对象引用，自身也有很多属性。

```python
def clip(text, max_len=80):
    """Return text clipped at the last space before or after max_len
    """
    end = None
    if len(text) > max_len:
        space_before = text.rfind(' ', 0, max_len)
        if space_before >= 0:
            end = space_before
        else:
            space_after = text.rfind(' ', max_len)
            if space_after >= 0:
              end = space_after
    if end is None: # no spaces were found
        end = len(text)
    return text[:end].rstrip()

# 应用：
In [15]: clip.__defaults__
Out[15]: (80,)

In [16]: clip.__code__
Out[16]: <code object clip at 0x109ab2a50, file "<ipython-input-14-1e0b203dc701>", line 1>

In [17]: clip.__code__.co_varnames
Out[17]: ('text', 'max_len', 'end', 'space_before', 'space_after')

In [19]: clip.__code__.co_argcount
Out[19]: 2

# 提取函数签名
In [22]: from inspect import signature

In [23]: sig = signature(clip)  # 返回 inspect.Signature 对象.

In [24]: sig
Out[24]: <Signature (text, max_len=80)>

In [25]: str(sig)
Out[25]: '(text, max_len=80)'

In [26]: for name,param in sig.parameters.items():  #  将参数名和 inspect.Parameter 对象对应起来
    ...:     print(param.kind, ':', name, '=', param.default)  # Parameter 属性有自己的属性， 如：name、default 和 kind。 inspect._empty 值表示没有默认值
    ...:
POSITIONAL_OR_KEYWORD : text = <class 'inspect._empty'>
POSITIONAL_OR_KEYWORD : max_len = 80
```

- `inspect.Parameter` 对象的 `kind` 属性 可选值有：
  - `POSITIONAL_OR_KEYWORD` : 可以通过定位参数和关键字参数传入的形参
  - `VAR_POSITIONAL` : 定位参数元组。
  - `VAR_KEYWORD` : 关键字参数字典。
  - `KEYWORD_ONLY` : 仅限关键字参数（Python 3 新增）。
  - `POSITIONAL_ONLY` : 仅限定位参数；目前，Python 声明函数的句法不支持，但是有些使用 C 语言实现且不接受关键字参数的函数（如 `divmod` ）支持。
- `inspect.Signature ` 对象的 `bind` 方法 :
  - 把任意个参数绑定到签名中的形参上，所用的规则与实参到形参的匹配方式一样。框架可以使用这个方法在真正调用函数前验证参数

```python
In [33]: import inspect

In [34]: sig = inspect.signature(tag)

In [35]: my_tag = {'name': 'img', 'title': 'Sunset Boulevard',
    ...:           'src': 'sunset.jpg', 'cls': 'framed'}

In [36]: bound_args = sig.bind(**my_tag)  # 返回 inspect.BoundArguments 对象

In [37]: bound_args
Out[37]: <BoundArguments (name='img', cls='framed', attrs={'title': 'Sunset Boulevard', 'src': 'sunset.jpg'})>

In [39]: for arg_name, arg_value in bound_args.arguments.items():
    ...:     print(arg_name, '=', arg_value)
    ...:
name = img
cls = framed
attrs = {'title': 'Sunset Boulevard', 'src': 'sunset.jpg'}

In [40]: del my_tag['name']

In [41]: bound_args = sig.bind(**my_tag)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-41-1e763c041430> in <module>()
----> 1 bound_args = sig.bind(**my_tag)
...
...

/usr/local/Cellar/python3/3.6.1/Frameworks/Python.framework/Versions/3.6/lib/python3.6/inspect.py in _bind(self, args, kwargs, partial)
   2846                             msg = 'missing a required argument: {arg!r}'
   2847                             msg = msg.format(arg=param.name)
-> 2848                             raise TypeError(msg) from None
   2849             else:
   2850                 # We have a positional argument to process

TypeError: missing a required argument: 'name'  # 缺少 name 参数.
```

参考 [inspect 模块](http://python.usyiyi.cn/translate/python_352/library/inspect.html)

- **函数注解** <span id="函数注解"></span>
  - 为函数声明中的参数和返回值附加元数据.
  - `def clip(text:str, max_len:'int > 0'=80) -> str: `
  - 各个参数在 `:` 之后增加注解表达式.
  - 如果参数有 `默认值`，注解放在 `参数名` 和 `=` 号之间.
  - 如果想注解 `返回值`，在 `)` 和函数声明末尾的 `:` 之间添加 `->` 和一个 `表达式` [可以是任何类型]。
  - 【注解中最常用的类型是类（如 `str` 或 `int`）和字符串（如 `'int > 0'`）】
  - 注解不会做任何处理，只是存储在函数的 `__annotations__` 属性（字典）中。 Python 不做检查、不做强制、不做验证，什么操作都不做。
    - `clip.__annotations__; {'max_len': 'int > 0', 'return': str, 'text': str}`
  - 注解只是元数据，可以供 IDE、框架和装饰器等工具使用。
  - 在未来, Bobo 等框架可以支持注解，并进一步自动处理请求。例如，使用 `price:float` 注解的参数可以自动把查询字符串转换成函数期待的 `float` 类型；`quantity:'int > 0'` 这样的字符串注解可以转换成对参数的验证。
  - 提取注解：

```python
# from inspect import signature
In [46]: def clip(text:str, max_len:'int > 0'=80) -> str: pass

In [47]: clip.__annotations__
Out[47]: {'max_len': 'int > 0', 'return': str, 'text': str}

In [48]: sig = signature(clip)  # 返回一个 Signature 对象

In [49]: sig.return_annotation  # 一个 return_annotation 属性
Out[49]: str

In [50]: for param in sig.parameters.values():  # parameters 属性,字典, 把参数名映射到 Parameter 对象上. 每个 Parameter 对象自己也有 annotation 属性.
    ...:     note = repr(param.annotation).ljust(13)
    ...:     print(note, ':', param.name, '=', param.default)
    ...:
<class 'str'> : text = <class 'inspect._empty'>
'int > 0'     : max_len = 80
```

- **常用的包** <span id="常用的包"></span>

- **operator模块**
  - `operator.mul` ：两个数相乘的 代替 `lambda a, b: a*b` 这样的函数.
  - `operator.itemgetter` : 根据元组的某个字段给元组列表排序. 代替 简单的 `lambda` 表达式.
    - `itemgetter` 使用 `[]` 运算符，不仅支持序列，还支持映射和任何实现 `__getitem__` 方法的类。
    - `itemgetter(1)` 代替如： `lambda fields: fields[1]`
    - `itemgetter(1,0)` 代替如：`lambda fields: field[1], field[0]`
    - `itemgetter('key')` 代替如：`lambda fields: field['key']`
  - `operator.attrgetter` : 根据名称提取对象的属性, 如果把多个属性名传给 `attrgetter` ，返回提取的值构成的元组。
    - `attrgetter('attr')` 代替如：`lambda obj: obj.attr`
    - `attrgetter('attr1', 'attr2')` 代替如：`lambda obj: obj.attr1, obj.attr2`
    - `attrgetter('attr1', 'attr2.subattr1')` 代替如： `lambda obj: obj.attr1, obj.attr2.subattr1`, 甚至是嵌套属性.
  - `operator.methodcaller` : 创建函数在对象上调用参数指定的方法.
  -
  - 更多参考：[operator模块](http://python.usyiyi.cn/translate/python_352/library/operator.html)

```python
# itemgetter 用例
>>> metro_data = [
...     ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
...     ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
...     ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
...     ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
...     ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
... ]
>>>
>>> from operator import itemgetter
>>> for city in sorted(metro_data, key=itemgetter(1)):
...     print(city)
...
('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833))
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
('Mexico City', 'MX', 20.142, (19.433333, -99.133333))
('New York-Newark', 'US', 20.104, (40.808611, -74.020386))
>>> cc_name = itemgetter(1, 0)
>>> for city in metro_data:
...     print(cc_name(city))
...
('JP', 'Tokyo')
('IN', 'Delhi NCR')
('MX', 'Mexico City')
('US', 'New York-Newark')
('BR', 'Sao Paulo')
In [55]: a = [dict(a=1),dict(a=2),dict(a=3)]

In [56]: import operator

In [57]: for b in a :
    ...:     print(operator.itemgetter('a')(b))
    ...:
1
2
3

# attrgetter 用例
>>> from collections import namedtuple
>>> LatLong = namedtuple('LatLong', 'lat long')  # 定义 LatLong
>>> Metropolis = namedtuple('Metropolis', 'name cc pop coord')  # 再定义 Metropolis
>>> metro_areas = [Metropolis(name, cc, pop, LatLong(lat, long))  # 使用嵌套的元组拆包提取 (lat, long)
...     for name, cc, pop, (lat, long) in metro_data]
>>> metro_areas[0]
Metropolis(name='Tokyo', cc='JP', pop=36.933, coord=LatLong(lat=35.689722,
long=139.691667))
>>> metro_areas[0].coord.lat  # 深入 metro_areas[0]，获取它的纬度。
35.689722
>>> from operator import attrgetter
>>> name_lat = attrgetter('name', 'coord.lat')  # 定义一个 attrgetter，获取 name 属性和嵌套的 coord.lat 属性。
>>>
>>> for city in sorted(metro_areas, key=attrgetter('coord.lat')):  # 使用 attrgetter，按照纬度排序城市列表。
...     print(name_lat(city))  # 使用定义的 attrgetter city，只显示城市名和纬度。
...
('Sao Paulo', -23.547778)
('Mexico City', 19.433333)
('Delhi NCR', 28.613889)
('Tokyo', 35.689722)
('New York-Newark', 40.808611)

# methodcaller 用例
In [58]: s = 'The time has come'

In [59]: upcase = operator.methodcaller('upper')

In [60]: upcase(s)
Out[60]: 'THE TIME HAS COME'

In [61]: hiphenate = operator.methodcaller('replace',' ','-')

In [62]: hiphenate(s)
Out[62]: 'The-time-has-come'
```

- **冻结参数** <span id="冻结参数"></span>
  - `functools` 模块提供了一系列高阶函数，其中最为人熟知的或许是 `reduce`. 余下的函数中，最有用的是 `partial` 及其变体,`partialmethod`.
  - `functools.partial` : 部分应用一个函数. 基于一个函数创建一个新的可调用对象，把原函数的某些参数固定。
    - 第一个参数是一个可调用对象，后面跟着任意个要绑定的定位参数和关键字参数。
    - [functools.py 的源码](https://hg.python.org/cpython/file/default/Lib/functools.py)表明，`functools.partial` 类是使用 C 语言实现的，而且默认使用这个实现。如果这个实现不可用，从 `Python 3.4` `起，functools` 模块为 `partial` 提供了纯 `Python` 实现。
  - `functools.partialmethod` 函数（Python 3.4 新增）的作用与 partial 一样，不过是用于处理方法的。
  - `functools.lru_cache` : 会做备忘（memoization）,一种自动优化措施，它会存储耗时的函数调用结果，避免重新计算。



```python
In [72]: from operator import mul

In [73]: from functools import partial

In [74]: triple = partial(mul,3)  # 把第一个定位参数定为 3

In [75]: triple(4)  # 测试 triple 函数
Out[75]: 12

In [76]: list(map(triple, range(10,20)))  # 在 map 中使用 triple ， 这里不能使用mul.
Out[76]: [30, 33, 36, 39, 42, 45, 48, 51, 54, 57]

# 使用 partial 构建一个便利的 Unicode 规范化函数
In [77]: import unicodedata, functools

In [78]: nfc = functools.partial(unicodedata.normalize, 'NFC')

In [79]: s1 = 'café'

In [80]: s2 = 'cafe\u0301'

In [81]: s1,s2
Out[81]: ('café', 'café')

In [82]: s1 == s2
Out[82]: False

In [83]: nfc(s1) == nfc(s2)
Out[83]: True

# 冻结生成图片的参数.
In [86]: picture = partial(tag,'img',cls='pic-frame')

In [87]: picture(src="wumpus.jpeg")
Out[87]: '<img class="pic-frame" src="wumpus.jpeg" />'

In [88]: picture
Out[88]: functools.partial(<function tag at 0x109c9e2f0>, 'img', cls='pic-frame')

In [89]: picture.func
Out[89]: <function __main__.tag>

In [90]: picture.args
Out[90]: ('img',)

In [91]: picture.keywords
Out[91]: {'cls': 'pic-frame'}
```
