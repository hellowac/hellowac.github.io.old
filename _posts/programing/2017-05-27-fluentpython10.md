---
nav: blog
layout: post
title: "流畅的python - 序列协议"
author: "wangchao"
tags:
  - python
  - '对象'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [序列协议](#序列协议)
- [序列对象表述](#序列对象表述)
- [动态存取属性](#动态存取属性)
- [散列和等值测试](#散列和等值测试)
- [对象格式化](#对象格式化)


**序列协议** <span id="序列协议"></span>

基本的序列协议： `__len__` 和 `__getitem__` .


```python
# 在 class Vector 类中.
  def __len__(self):
      return len(self._components)  # 委托给对象中的序列属性

  def __getitem__(self, index):
        return self._components[index]  # 委托给对象中的序列属性

# 第二版, slice 说明.
  def __getitem__(self, index):
      cls = type(self) # 获取实例所属的类.
      if isinstance(index, slice):  # 内置类型， slice， 有 start、stop 和 step 数据属性，以及 indices 方法
          return cls(self._components[index])  # 创建一个新的类型.
      elif isinstance(index, numbers.Integral):  # 整数的抽象基类,能让 API 更灵活且更容易更新
          return self._components[index]  # 返回相应的元素.
      else:
          msg = '{.__name__} indices must be integers'  # 参考了标准错误.
          raise TypeError(msg.format(cls))

# slice 中 indices 方法说明：用于优雅地处理缺失索引和负数索引，以及长度超过目标序列的切片
>>> slice(None, 10, 2).indices(5)  # 'ABCDE'[:10:2] 等同于 'ABCDE'[0:5:2]
(0, 5, 2)
>>> slice(-3, None, None).indices(5)  # 'ABCDE'[-3:] 等同于 'ABCDE'[2:5:1]
(2, 5, 1)

```

`slice.indices` 参考： [slice.indices](https://docs.python.org/3/reference/datamodel.html?highlight=indices#slice.indices) 以及 中文版 [slice.indices](http://python.usyiyi.cn/translate/python_352/reference/datamodel.html#slice.indices)

**序列对象表述** <span id="序列对象表述"></span>

```python
# 在 class Vector 类中.
def __repr__(self):
        components = reprlib.repr(self._components)  # 获取 self._components 的有限长度表示形式, 如 array('d', [0.0, 1.0, 2.0, 3.0, 4.0, ...])
        components = components[components.find('['):-1]  # 掉前面的 "array('d'" 和后面的 ")"
        return 'Vector({})'.format(components)
```

- **`reprlib.repr` 的方式:**  这个函数用于生成大型结构或递归结构的安全表示形式，它会限制输出字符串的长度，用 '`...`' 表示截断的部分。
- 调用 `repr()` 函数的目的是调试，因此绝对不能抛出异常。如果 `__repr__` 方法的实现有问题，那么必须处理，尽量输出有用的内容，让用户能够识别目标对象。
- 在面向对象编程中，协议是非正式的接口，只在文档中定义，在代码中不定义。
  - 例如，Python 的序列协议只需要 `__len__` 和 `__getitem__` 两个方法。
  - 任何类（如 Spam），只要使用标准的签名和语义实现了这两个方法，就能用在任何期待序列的地方。
  - Spam 是不是哪个类的子类无关紧要，只要提供了所需的方法即可。
- 协议是非正式的，没有强制力，因此如果你知道类的具体使用场景，通常只需要实现一个协议的部分。
  - 例如，为了支持迭代，只需实现 `__getitem__` 方法，没必要提供 __len__ 方法。

**动态存取属性** <span id="动态存取属性"></span>

```python
#  在 class Vector 类中. 实现, v.x 快捷取下标为0的值.
  shortcut_names = 'xyzt' # x 取下标为0 的元素, y取下标为1的元素.等等.

  def __getattr__(self, name):
      cls = type(self)  # 获取类型

      if len(name) == 1:  # 如果属性名只有一个字母，可能是 shortcut_names 中的一个。
          pos = cls.shortcut_names.find(name)  # 查找那个字母的位置
          if 0 <= pos < len(self._components):  # 范围内，返回数组中对应的元素。
              return self._components[pos]
      msg = '{.__name__!r} object has no attribute {!r}'  # 获取失败了.
      raise AttributeError(msg.format(cls, name))

# 但此处，如果动态设置属性的话，将会覆盖快捷取值的方式，
>>> v = Vector(range(5))
>>> v
Vector([0.0, 1.0, 2.0, 3.0, 4.0])
>>> v.x  # 或下标为0的元素
0.0
>>> v.x = 10  # 动态赋值
>>> v.x  # 得到新值。
10
>>> v
Vector([0.0, 1.0, 2.0, 3.0, 4.0])  # 元数据没变。

# 实现 __setter__ 方法.
    def __setattr__(self, name, value):
        cls = type(self)
        if len(name) == 1:  # 特别处理名称是单个字符的属性
            if name in cls.shortcut_names:  # 如果 name 是 xyzt 中的一个，设置特殊的错误消息。
                error = 'readonly attribute {attr_name!r}'
            elif name.islower():  # 如果 name 是小写字母，为所有小写字母设置一个错误消息。
                error = "can't set attributes 'a' to 'z' in {cls_name!r}"
            else:
                error = ''  # 否则跳过.
            if error:  # 如果有错误消息内容，抛出
                msg = error.format(cls_name=cls.__name__, attr_name=name)
                raise AttributeError(msg)
        super().__setattr__(name, value)  # 默认情况：在超类上调用 __setattr__ 方法，提供标准行为。

# 在类中声明 __slots__ 属性可以防止设置新实例属性；
# 因此，可能想使用这个功能，而不像这里所做的，实现 __setattr__ 方法。
# 可是，不建议只为了避免创建实例属性而使用 __slots__ 属性。
# __slots__ 属性只应该用于节省内存，而且仅当内存严重不足时才应该这么做。

```
- **特别注意** ：多数时候，如果实现了 __getattr__ 方法，那么也要定义 __setattr__ 方法，以防对象的行为不一致。

**散列和等值测试** <span id="散列和等值测试"></span>

实现 `__hash__` 方法。加上 `__eq__` 方法，会把 `Vector` 实例变成可散列的对象。

```python
# 阶乘的3中方式

>>> n = 0
>>> for i in range(1, 6):  # for 循环
...     n ^= i
...
>>> n
1
>>> import functools
>>> functools.reduce(lambda a, b: a^b, range(6))  # lambda 表达式
1
>>> import operator
>>> functools.reduce(operator.xor, range(6))  # lambda 转化为函数.
1

# 实现 hash
from array import array
import reprlib
import math
import functools  # 使用 reduce 函数
import operator  # 使用 xor 函数.


class Vector:
    typecode = 'd'

    # 排版需要，省略了很多行...

    def __eq__(self, other):  # 和 __hash__ 要结合使用.
        return tuple(self) == tuple(other)  # 数列少的时候还好, 下面会优化.

    def __hash__(self):
        hashes = (hash(x) for x in self._components)  # 创建一个生成器表达式，惰性计算各个分量的散列值。
        return functools.reduce(operator.xor, hashes, 0)  # 0 是初始值

    # 省略了很多行...


# 优化 __eq__ 函数:

def __eq__(self, other):
    if len(self) != len(other):  # 长度最先判断.
        return False
    for a, b in zip(self, other):  # zip函数,生成各个序列的 拆包形式变量。
        if a != b:  # 只要有两个分量不同，返回 False，退出。
            return False
    return True  # 否则，对象是相等的。

# 更优化的：
def __eq__(self, other):
    return len(self) == len(other) and all(a == b for a, b in zip(self, other))  # 利用 all 函数.
```

- **注意：**
  - 使用 `reduce` 函数时最好提供第三个参数，`reduce(function, iterable, initializer)`，这样能避免这个异常：`TypeError: reduce() of empty sequence with no initial value`（这个错误消息很棒，说明了问题，还提供了解决方法）。
  - 如果序列为空，`initializer` 是返回的结果；否则，在归约中使用它作为第一个参数，因此应该使用恒等值。比如，对 `+`、`|` 和 `^` 来说， `initializer` 应该是 `0`；而对 `*` 和 `&` 来说，应该是 `1`。
- `映射归约`：把各个元素应用到函数上，生成一个新序列（映射，`map`），然后计算聚合值（归约，`reduce`）
- `zip` 函数： 当一个可迭代对象耗尽后，不发出警告就停止。
  - `itertools.zip_longest` 函数的行为有所不同：使用可选的 `fillvalue`（默认值为 `None`）填充缺失的值，因此可以继续产出，直到最长的可迭代对象耗尽。
  - 参考内置函数： [built-in-functions](http://python.usyiyi.cn/translate/python_352/library/functions.html)

**对象格式化** <span id="对象格式化"></span>

扩展[格式规范微语言](https://docs.python.org/3/library/string.html#formatspec)时，最好避免重用内置类型支持的格式代码。

浮点数的格式代码有 `'eEfFgGn%'`，整数使用的格式代码有 `'bcdoxXn'`，字符串使用的是 `'s'`。

```python

# 类 Vector 中定义。
    def angle(self, n):  # 计算某个角坐标
        r = math.sqrt(sum(x * x for x in self[n:]))
        a = math.atan2(r, self[n-1])
        if (n == len(self) - 1) and (self[-1] < 0):
            return math.pi * 2 - a
        else:
            return a

    def angles(self):  # 创建生成器表达式，按需计算所有角坐标。
        return (self.angle(n) for n in range(1, len(self)))

    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('h'):  # 超球面坐标
            fmt_spec = fmt_spec[:-1]
            coords = itertools.chain([abs(self)],
                                     self.angles())  # 使用 itertools.chain 函数生成生成器表达式，无缝迭代向量的模和各个角坐标。
            outer_fmt = '<{}>'  # 配置使用尖括号显示球面坐标
        else:
            coords = self
            outer_fmt = '({})'  # 配置使用圆括号显示笛卡儿坐标。
        components = (format(c, fmt_spec) for c in coords)  # 创建生成器表达式，按需格式化各个坐标元素。
        return outer_fmt.format(', '.join(components))  # 把以逗号分隔的格式化分量插入尖括号或圆括号。
    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(memv)
```

**Vector 完整版**

```python
from array import array
import reprlib
import math
import numbers
import functools
import operator
import itertools


class Vector:
    typecode = 'd'

    def __init__(self, components):
        self._components = array(self.typecode, components)

    def __iter__(self):
        return iter(self._components)

    def __repr__(self):
        components = reprlib.repr(self._components)
        components = components[components.find('['):-1]
        return 'Vector({})'.format(components)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(self._components))

    def __eq__(self, other):
        return (len(self) == len(other) and
                all(a == b for a, b in zip(self, other)))

    def __hash__(self):
        hashes = (hash(x) for x in self)
        return functools.reduce(operator.xor, hashes, 0)

    def __abs__(self):
        return math.sqrt(sum(x * x for x in self))

    def __bool__(self):
        return bool(abs(self))

    def __len__(self):
        return len(self._components)

    def __getitem__(self, index):
        cls = type(self)
        if isinstance(index, slice):
            return cls(self._components[index])
        elif isinstance(index, numbers.Integral):
            return self._components[index]
        else:
            msg = '{.__name__} indices must be integers'
            raise TypeError(msg.format(cls))

    shortcut_names = 'xyzt'

    def __getattr__(self, name):
        cls = type(self)
        if len(name) == 1:
            pos = cls.shortcut_names.find(name)
            if 0 <= pos < len(self._components):
                return self._components[pos]
        msg = '{.__name__!r} object has no attribute {!r}'
        raise AttributeError(msg.format(cls, name))

    def angle(self, n):
        r = math.sqrt(sum(x * x for x in self[n:]))
        a = math.atan2(r, self[n-1])
        if (n == len(self) - 1) and (self[-1] < 0):
            return math.pi * 2 - a
        else:
            return a

    def angles(self):
        return (self.angle(n) for n in range(1, len(self)))

    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('h'):  # 超球面坐标
            fmt_spec = fmt_spec[:-1]
            coords = itertools.chain([abs(self)],
                                     self.angles())
            outer_fmt = '<{}>'
        else:
            coords = self
            outer_fmt = '({})'
        components = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(', '.join(components))

    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(memv)
```

```python
# 多维的 向量 类 ，第5版.

# 测试构造函数对整数可迭代元素的支持
In [6]: Vector([3.1, 4.2])
Out[6]: Vector([3.1, 4.2])

In [7]: Vector((3, 4, 5))
Out[7]: Vector([3.0, 4.0, 5.0])

In [8]: Vector(range(10))
Out[8]: Vector([0.0, 1.0, 2.0, 3.0, 4.0, ...])

# 测试 二维 支持.
In [9]: v1 = Vector([3, 4])  # 构建二维对象

In [10]: x,y = v1

In [11]: x,y
Out[11]: (3.0, 4.0)

In [12]: v1
Out[12]: Vector([3.0, 4.0])

In [13]: v1_clone = eval(repr(v1))  # 测试 __repr__

In [14]: v1 == v1_clone
Out[14]: True

In [15]: print(v1)
(3.0, 4.0)

In [16]: octests = bytes(v1)  # 测试 __bytes__

In [17]: octests
Out[17]: b'd\x00\x00\x00\x00\x00\x00\x08@\x00\x00\x00\x00\x00\x00\x10@'

In [18]: abs(v1)  # 测试 __abs__
Out[18]: 5.0

In [19]: bool(v1),bool(Vector([0,0]))  # 测试 __bool__
Out[19]: (True, False)

# 测试类方法 frombytes
In [20]: v1_clone = Vector.frombytes(bytes(v1))

In [21]: v1_clone
Out[21]: Vector([3.0, 4.0])

In [22]: v1 == v1_clone
Out[22]: True

# 测试 三维 .
In [23]: v1 = Vector([3, 4, 5])  # 测试构造方法

In [24]: x, y, z = v1  # 测试 __iter__

In [25]: x, y, z
Out[25]: (3.0, 4.0, 5.0)

In [26]: v1  # 测试 __repr__
Out[26]: Vector([3.0, 4.0, 5.0])

In [27]: v1_clone = eval(repr(v1))  # 测试 __repr__

In [28]: v1 == v1_clone
Out[28]: True

In [29]: print(v1)  # 测试 __str__
(3.0, 4.0, 5.0)

In [30]: abs(v1)  # 测试 __abs__
Out[30]: 7.0710678118654755

In [32]: bool(v1), bool(Vector([0,0,0]))  # 测试 __bool__
Out[32]: (True, False)

# 测试 许多纬度
In [33]: v7 = Vector(range(7))  # 测试 7 个纬度.

In [34]: v7  # 测试 __repr__ 以及 reprlib.repr 方法.
Out[34]: Vector([0.0, 1.0, 2.0, 3.0, 4.0, ...])

In [35]: abs(v7)  # 测试 __abs__
Out[35]: 9.539392014169456

# 测试 对象在字节序之间的转换.
In [36]: v1 = Vector([3, 4, 5])

In [37]: v1_clone = Vector.frombytes(bytes(v1))

In [38]: v1_clone
Out[38]: Vector([3.0, 4.0, 5.0])

# 测试 序列行为
In [39]: v1 = Vector([3, 4, 5])

In [40]: len(v1)
Out[40]: 3

In [41]: v1[0], v1[len(v1)-1], v1[-1]
Out[41]: (3.0, 5.0, 5.0)

# 测试 切片
In [42]: v7 = Vector(range(7))

In [43]: v7[-1]
Out[43]: 6.0

In [44]: v7[1:4]
Out[44]: Vector([1.0, 2.0, 3.0])

In [45]: v7[-1:]
Out[45]: Vector([6.0])

In [46]: v7[1,2]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-46-82fccfd8982a> in <module>()
----> 1 v7[1,2]

....

TypeError: Vector indices must be integers

# 测试 动态属性 取值.
In [47]: v7 = Vector(range(10))

In [48]: v7.x
Out[48]: 0.0

In [49]: v7.y, v7.z, v7.t
Out[49]: (1.0, 2.0, 3.0)

# 测试 未允许的 动态属性
In [50]: v7.k
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-50-0ec6231d4933> in <module>()
...

AttributeError: 'Vector' object has no attribute 'k'

In [51]: v3 = Vector(range(3))

In [52]: v3.t
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-52-8cf023308d9a> in <module>()
...

AttributeError: 'Vector' object has no attribute 't'

In [53]: v3.spam
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-53-5f03e49da060> in <module>()
...

AttributeError: 'Vector' object has no attribute 'spam'

# 测试 散列
In [54]: v1 = Vector([3, 4])

In [55]: v2 = Vector([3.1, 4.2])

In [56]: v3 = Vector([3, 4, 5])

In [57]: v6 = Vector(range(6))

In [58]: hash(v1), hash(v3), hash(v6)
Out[58]: (7, 2, 1)

# 在CPython中，测试 v2 的 hash 值.
In [60]: import sys

In [61]: hash(v2) == (384307168202284039 if sys.maxsize > 2**32 else 357915986)
Out[61]: True

# 测试 二维 中的 笛卡尔坐标 格式化.
In [63]: v1 = Vector([3, 4])

In [64]: format(v1)
Out[64]: '(3.0, 4.0)'

In [65]: format(v1, '.2f')
Out[65]: '(3.00, 4.00)'

In [66]: format(v1, '.3e')
Out[66]: '(3.000e+00, 4.000e+00)'

# 测试 二维 和 七纬 中的 笛卡尔坐标 格式化.
In [67]: v3 = Vector([3, 4, 5])

In [68]: format(v3)
Out[68]: '(3.0, 4.0, 5.0)'

In [69]: format(Vector(range(7)))
Out[69]: '(0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0)'

# 测试 球形 坐标 的 格式化. [自定义的格式化，h结尾]
In [69]: format(Vector(range(7)))
Out[69]: '(0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0)'

In [70]: format(Vector([1, 1]), 'h')
Out[70]: '<1.4142135623730951, 0.7853981633974483>'

In [71]: format(Vector([1, 1]), '.3eh')
Out[71]: '<1.414e+00, 7.854e-01>'

In [72]: format(Vector([1, 1]), '0.5fh')
Out[72]: '<1.41421, 0.78540>'

In [73]: format(Vector([1, 1, 1]), 'h')
Out[73]: '<1.7320508075688772, 0.9553166181245093, 0.7853981633974483>'

In [74]: format(Vector([2, 2, 2]), '.3eh')
Out[74]: '<3.464e+00, 9.553e-01, 7.854e-01>'

In [75]: format(Vector([0, 0, 0]), '0.5fh')
Out[75]: '<0.00000, 0.00000, 0.00000>'

In [76]: format(Vector([-1, -1, -1, -1]), 'h')
Out[76]: '<2.0, 2.0943951023931957, 2.186276035465284, 3.9269908169872414>'

In [77]: format(Vector([2, 2, 2, 2]), '.3eh')
Out[77]: '<4.000e+00, 1.047e+00, 9.553e-01, 7.854e-01>'

In [78]: format(Vector([0, 1, 0, 0]), '0.5fh')
Out[78]: '<1.00000, 1.57080, 0.00000, 0.00000>'
```
