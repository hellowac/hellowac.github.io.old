---
nav: blog
layout: post
title: "流畅的python - python风格的对象"
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


- [pythonic的对象](#pythonic的对象)
- [类方法和静态方法](#类方法和静态方法)
- [格式化显示](#格式化显示)
- [私有属性](#私有属性)
- [`__slots__`类属性](#__slots__类属性)


**pythonic的对象** <span id="pythonic的对象"></span>

```python
from array import array
import math


class Vector2d:
    typecode = 'd'  # 在字节序之间转换时所用.

    def __init__(self, x, y):
        self.x = float(x)   # 转换成浮点数, 尽早捕获错误，以防调用函数时传入不当参数。
        self.y = float(y)

    def __iter__(self):
        return (i for i in (self.x, self.y))  # 变成可迭代的对象，这样才能拆包（例如，x, y = my_vector）。 也可以使用 yield 表达式.

    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)  # 用 {!r} 获取各个分量的表示形式，然后插值，构成一个字符串；

    def __str__(self):
        return str(tuple(self))  # 得到一个元组，显示为一个有序对。利用了上面的可迭代性.

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +  # 把 typecode 转换成字节序列
                bytes(array(self.typecode, self)))  # 迭代 Vector2d 实例，得到一个数组，再把数组转换成字节序列。最终的到实例的字节序.

    def __eq__(self, other):
        return tuple(self) == tuple(other)  # 在操作数中构建元组,快速比较所有分量. [在两个操作数都是 Vector2d 实例时可用，不过拿 Vector2d 实例与其他具有相同数值的可迭代对象相比，结果也是 True（如 Vector(3, 4) == [3, 4]）]

    def __abs__(self):
        return math.hypot(self.x, self.y)  # 模是 x 和 y 分量构成的直角三角形的斜边长。

    def __bool__(self):
        return bool(abs(self))  # 使用 abs(self) 计算模，然后把结果转换成布尔值，0.0 是 False，非零值是 True。

    # 当做备用构造方法.
    @classmethod # 类方法修饰符
    def frombytes(cls, octets):  # 不是实例，而是类本身.
        typecode = chr(octets[0]) # 从第一个字节读取 typecoe.
        memv = memoryview(octets[1:]).cast(typecode)  # 创建一个内存试图,并通过 typecode 进行转换. cast 会把同一块内存里的内容打包成一个全新的 memoryview 对象
        return cls(*memv)  # 拆包后的 memv 得到构造参数.

    def __format__(self, fmt_spec=''):
        components = (format(c, fmt_spec) for c in self)  # 应用到各个分量上.
        return '({}, {})'.format(**components) # 代入公式'({}, {})'中.

# 执行：
In [3]: v1 = Vector2d(3,4)

In [4]: print(v1.x,v1.y)  # 直接通过属性访问.
3.0 4.0

In [6]: x,y=v1  # 拆包成变量元组,利用可迭代性

In [7]: x,y
Out[7]: (3.0, 4.0)

In [8]: v1  # 利用对象的字符串表示法，得到类似于构建实例的源码.
Out[8]: Vector2d(3.0, 4.0)

In [9]: v1_clone = eval(repr(v1))  # 也可以使用copy.copy函数更快速.

In [10]: v1 == v1_clone  # 通过 __eq__ 支持 == 比较，便于测试.
Out[10]: True

In [11]: v1_clone
Out[11]: Vector2d(3.0, 4.0)

In [12]: print(v1)
(3.0, 4.0)

In [13]: octets = bytes(v1)  # 调用 __bytes__ ,生成实例的二进制表示形式.

In [14]: octets
Out[14]: b'd\x00\x00\x00\x00\x00\x00\x08@\x00\x00\x00\x00\x00\x00\x10@'

In [15]: abs(v1)  # 返回实例的模.
Out[15]: 5.0

In [16]: bool(v1),bool(Vector2d(0,0)) # 如果模为0，返回 False， 非零返回 True
Out[16]: (True, False)
```

**类方法和静态方法** <span id="类方法和静态方法"></span>

- **类方法：** 使用 `@classmethod` 修饰符. 第一个参数传入类本身. 随后跟着参数列表.
- **静态方法：** 使用 `@staticmethod` 修饰符. 不传入 类 本身 或 实例, 直接传参数列表.
- 关于静态方法的使用方法：参考 [The Definitive Guide on How to Use Static, Class or Abstract Methods in Python](https://julien.danjou.info/blog/2013/guide-python-static-class-abstract-methods)

**格式化显示** <span id="格式化显示"></span>

内置的 `format()` 函数和 `str.format()` 方法把各个类型的格式化方式委托给相应的 `.__format__(format_spec)` 方法。`format_spec` 是格式说明符，它是：

- `format(my_obj, format_spec)` 的第二个参数，或者
- `str.format()` 方法的格式字符串，`{}` 里代换字段中冒号后面的部分

更多参考：[Format String Syntax](https://docs.python.org/3/library/string.html#formatspec), 字符串格式化微语言。包含转换标志 `!s`、`!r` 和 `!a`. 中文版：[格式规范迷你语言](http://python.usyiyi.cn/documents/python_352/library/string.html#format-specification-mini-language)

内置类型：`b` 和 `x` 分别表示二进制和十六进制的 `int` 类型，`f` 表示小数形式的 `float` 类型，而 `%` 表示百分数形式.
每个类可以自行决定如何解释 `format_spec` 参数, 通过 `__format__` 特殊方法.
如果类没有定义 `__format__` 方法，从 `object` 继承的方法会返回 `str(my_object)`, 此时传入格式说明符，会抛出 `TypeError` 异常。

```python
In [18]: brl = 1/2.43 # BRL到USD的货币兑换比价

In [19]: brl
Out[19]: 0.4115226337448559

In [20]: format(brl, '0.4f')  # 格式说明符是 '0.4f'。
Out[20]: '0.4115'

In [21]: '1 BRL = {rate:0.2f} USD'.format(rate=brl)  # 格式说明符是 '0.2f'。代换字段中的 'rate' 子串是字段名称，与格式说明符无关，但是它决定把 .format() 的哪个参数传给代换字段。
Out[21]: '1 BRL = 0.41 USD'

# __format__ 第二版， 计算 极坐标
# 在Vector2d类中定义

def angle(self):
       return math.atan2(self.y, self.x)  # 使用 math.atan2() 函数计算角度

def __format__(self, fmt_spec=''):
    if fmt_spec.endswith('p'):  # 以 'p' 结尾，使用极坐标
        fmt_spec = fmt_spec[:-1]  # 删除 'p' 后缀
        coords = (abs(self), self.angle())  # 构建一个元组，表示极坐标：(magnitude, angle)
        outer_fmt = '<{}, {}>'  # 外层格式设为一对尖括号
    else:
        coords = self  # 不以 'p' 结尾，使用 self 的 x 和 y 分量构建直角坐标。
        outer_fmt = '({}, {})'  # 外层格式设为一对圆括号
    components = (format(c, fmt_spec) for c in coords)  # 使用各个分量生成可迭代的对象，构成格式化字符串。
    return outer_fmt.format(*components)  # 把格式化字符串代入外层格式

# 执行
In [4]: format(Vector2d(1,1),'p')
Out[4]: '<1.4142135623730951, 0.7853981633974483>'

In [5]: format(Vector2d(1,1),'.3ep')
Out[5]: '<1.414e+00, 7.854e-01>'

In [6]: format(Vector2d(1,1),'0.5fp')
Out[6]: '<1.41421, 0.78540>'

# 可散列的
# 实例变成可散列的，必须使用 __hash__ 方法（还需要 __eq__ 方法，前面已经实现了）,此外，还要让向量不可变，
# 要想创建可散列的类型，不一定要实现特性，也不一定要保护实例属性。只需正确地实现 __hash__ 和 __eq__ 方法即可。但是，实例的散列值绝不应该变化.
class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)  # 使用两个前导下划线（尾部没有下划线，或者有一个下划线），把属性标记为私有的。
        self.__y = float(y)

    @property  # @property 装饰器把读值方法标记为特性。
    def x(self):  # 读值方法与公开属性同名，都是 x。
        return self.__x

    @property  # 同样的方式处理 y 特性。
    def y(self):
        return self.__y

    def __hash__(self):
        return hash(self.x) ^ hash(self.y) # 使用位运算符异或（^）混合各分量的散列值

    def __iter__(self):
        return (i for i in (self.x, self.y))  # 需要读取 x 和 y 分量的方法可以保持不变，通过 self.x 和 self.y 读取公开特性，而不必读取私有属性

    # ... 其他方法

# 如果定义的类型有标量数值，可能还要实现 __int__ 和 __float__ 方法（分别被 int() 和 float() 构造函数调用），以便在某些情况下用于强制转换类型。
# 此外，还有用于支持内置的 complex() 构造函数的 __complex__ 方法。Vector2d 或许应该提供 __complex__ 方法，

# 完整的代码：
from array import array
import math

class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)

    @property
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y

    def __iter__(self):
        return (i for i in (self.x, self.y))

    def __repr__(self):
       class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(array(self.typecode, self)))

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    def __hash__(self):
        return hash(self.x) ^ hash(self.y)

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def angle(self):
        return math.atan2(self.y, self.x)

    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('p'):
            fmt_spec = fmt_spec[:-1]
            coords = (abs(self), self.angle())
            outer_fmt = '<{}, {}>'
        else:
            coords = self
            outer_fmt = '({}, {})'
        components = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(*components)

    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)
```

**私有属性** <span id="私有属性"></span>

如果实例变量前有两个前导下划线，尾部没有或最多有一个下划线，Python 会把属性名存入实例的 `__dict__` 属性中，而且会在前面加上一个下划线和类名。

因此对 `Dog` 类来说，`__mood` 会变成 `_Dog__mood`, 对 `Beagle` 类来说，会变成 `_Beagle__mood`。这个语言特性叫`名称改写`（name mangling）。

名称改写是一种安全措施，不能保证万无一失：它的目的是避免意外访问，不能防止故意做错事

```python
>>> v1 = Vector2d(3, 4)
>>> v1.__dict__
{'_Vector2d__y': 4.0, '_Vector2d__x': 3.0}
>>> v1._Vector2d__x
3.0
```

更多名称改写风格， 参考：[Paste 的风格指南](http://pythonpaste.org/StyleGuide.html)

Python 解释器不会对使用单个下划线的属性名做特殊处理，不过这是很多 Python 程序员严格遵守的约定，他们不会在类外部访问这种属性。

遵守使用一个下划线标记对象的私有属性很容易，就像遵守使用全大写字母编写常量那样容易。

不过在模块中，顶层名称使用一个前导下划线的话，的确会有影响：对 `from mymod import * ` 来说，`mymod` 中前缀为下划线的名称不会被导入。

然而，依旧可以使用 `from mymod import _privatefunc` 将其导入。Python 教程的 6.1 节“[More on Modules](https://docs.python.org/3/tutorial/modules.html#more-on-modules)”说明了这一点。

**__slots__类属性** <span id="`__slots__`类属性"></span>

如果要处理数百万个属性不多的实例，通过 `__slots__` 类属性，能节省大量内存，方法是让解释器在元组中存储实例属性，而不用字典。 【字典有空间未利用】

继承自超类的 `__slots__` 属性没有效果。Python 只会使用各个类中定义的 `__slots__` 属性。

定义 `__slots__` 的方式: 创建一个类属性，使用 `__slots__` 这个名字，并把它的值设为一个字符串构成的可迭代对象，其中各个元素表示各个实例属性。

```python
class Vector2d:
    __slots__ = ('__x', '__y')  # 目的是告诉解释器：“这个类中的所有实例属性都在这儿了！”
    # 这样，Python 会在各个实例中使用类似元组的结构存储实例变量，从而避免使用消耗内存的 __dict__ 属性。

    typecode = 'd'

    # ...
```

在类中定义 `__slots__` 属性之后，实例不能再有 `__slots__` 中所列名称之外的其他属性。

这只是一个副作用，不是 `__slots__` 存在的真正原因。

不要使用 `__slots__` 属性禁止类的用户新增实例属性。

`__slots__` 是用于优化的，不是为了约束程序员。

然而，“节省的内存也可能被再次吃掉”：如果把 '`__dict__`' 这个名称添加到 `__slots__` 中，实例会在元组中保存各个实例的属性，此外还支持动态创建属性，这些属性存储在常规的 `__dict__` 中。当然，把 '`__dict__`' 添加到 `__slots__` 中可能完全违背了初衷，这取决于各个实例的静态属性和动态属性的数量及其用法。

粗心的优化甚至比提早优化还糟糕。

此外，还有一个实例属性可能需要注意，即 `__weakref__` 属性，为了让对象支持弱引用（参见 8.6 节），必须有这个属性。

用户定义的类中默认就有 `__weakref__` 属性。

可是，如果类中定义了 `__slots__` 属性，而且想把实例作为弱引用的目标，那么要把 '`__weakref__`' 添加到 `__slots__` 中。

**__slots__的问题：**

- 每个子类都要定义 `__slots__` 属性，因为解释器会忽略继承的 `__slots__` 属性。
- 实例只能拥有 `__slots__` 中列出的属性，除非把 '`__dict__`' 加入 `__slots__` 中（这样做就失去了节省内存的功效）。
- 如果不把 '`__weakref__`' 加入 `__slots__`，实例就不能作为弱引用的目标。
