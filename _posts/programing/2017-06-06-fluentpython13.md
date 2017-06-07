---
nav: blog
layout: post
title: "流畅的python - 重载运算符"
author: "wangchao"
tags:
  - python
  - '对象'
  - '继承'
  - 'Django'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [python对重载运算符的限制](#python对重载运算符的限制)
- [重载一元运算符](#重载一元运算符)
- [重载二元运算符](#重载二元运算符)
- [重载比较运算符](#重载比较运算符)
- [重载增量运算符](#重载增量运算符)


<span id="python对重载运算符的限制"></span>

### python对重载运算符的限制

Python 施加的一些限制，做了灵活性、可用性和安全性方面的平衡：

- 不能重载内置类型的运算符
- 不能新建运算符，只能重载现有的
- 某些运算符不能重载——`is`、`and`、`or` 和 `not`（不过位运算符 `&`、`|` 和 `~` 可以）

<span id="重载一元运算符"></span>

### 重载一元运算符

[6.5. Unary arithmetic and bitwise operations](https://docs.python.org/3/reference/expressions.html#unary-arithmetic-and-bitwise-operations), 列出了三个一元运算符, 下面是这三个运算符和对应的特殊方法。

- `-`（`__neg__`）
  - 一元取负算术运算符。如果 `x` 是 `-2`，那么 `-x == 2`。
- `+`（`__pos__`）
  - 一元取正算术运算符。通常，`x == +x`，但也有一些例外。(`Decimal` 中精度不一致时.)
- `~`（`__invert__`）
  - 对整数按位取反，定义为 `~x == -(x+1)`。如果 `x` 是 `2`，那么 `~x == -3`。

[Data Model](https://docs.python.org/3/reference/datamodel.html#object.__neg__), 还把内置的 `abs(...)` 函数列为一元运算符。它对应的特殊方法是 `__abs__`.

支持他们，只需实现对应的特殊方法，并且这些特殊方法只有一个参数，就是 `self` 自身.
实现特殊方法时要遵循原则：始终返回一个新对象。（不能修改 `self`，要创建并返回合适类型的新实例。）

```python
    def __abs__(self):
        return math.sqrt(sum(x * x for x in self))

    def __neg__(self):
        return Vector(-x for x in self)  # 构建一个新 Vector 实例，把 self 的每个分量都取反。

    def __pos__(self):
        return Vector(self)  # 构建一个新 Vector 实例，传入 self 的各个分量。
```

<span id="重载二元运算符"></span>

### 重载二元运算符

- `+` (`__add__`)
  - 用于拼接
- `*` (`*`)
  - 用于重复复制
- 更多的参考表格.

```python
    # 在Vector类中定义， 支持所有可迭代对象.
    def __add__(self, other):
        try:
             pairs = itertools.zip_longest(self, other, fillvalue=0.0) # 使用 0.0 填充不足的值.
             return Vector(a + b for a, b in pairs)  # 使用生成器重新生成对象.
         except TypeError:
             return NotImplemented  # 类型不支持时，让python解释器尝试其他方法， __radd__ 或 抛出类型错误.

    #用于支持 方向运算， 如: a + b - >  b.__radd__(a)
    def __radd__(self, other):
        return self + other  # 委托给 __add__

    def __mul__(self, scalar):
        if isinstance(scalar, numbers.Real):  # 检查数值类型， 记得引入 numbers
            return Vector(n * scalar for n in self)
        else:  # 让 Python 尝试在 scalar 操作数上调用 __rmul__ 方法。
            return NotImplemented

    def __rmul__(self, scalar):
        return self * scalar  # 委托给 __mul__ 方法。
```

*实现一元运算符和中缀运算符的特殊方法一定不能修改操作数。使用这些运算符的表达式期待结果是新对象。只有增量赋值表达式可能会修改第一个操作数（`self`）.*

为了支持涉及不同类型的运算，`Python` 为中缀运算符特殊方法提供了特殊的分派机制。对表达式 `a + b` 来说，解释器会执行以下几步操作：

- 如果 `a` 有 `__add__` 方法，而且返回值不是 `NotImplemented`，调用 `a.__add__(b)`，然后返回结果。
- 如果 `a` 没有 `__add__` 方法，或者调用 `__add__` 方法返回 `NotImplemented`，检查 `b` 有没有 `__radd__` 方法，如果有，而且没有返回 `NotImplemented`，调用 `b.__radd__(a)`，然后返回结果。
- 如果 `b` 没有 `__radd__` 方法，或者调用 `__radd__` 方法返回 `NotImplemented`，抛出 `TypeError`，并在错误消息中指明操作数类型不支持。

```
别把 NotImplemented 和 NotImplementedError 搞混了。
前者是特殊的单例值，如果中缀运算符特殊方法不能处理给定的操作数，那么要把它返回（return）给解释器。
而 NotImplementedError 是一种异常，抽象类中的占位方法把它抛出（raise），提醒子类必须覆盖。
```

更多二元运算符:

运算符 | 正向方法 | 反向方法 | 就地方法 | 说明
------|----------|---------|---------|--------
`+` | `__add__` | `__radd__` | `__iadd__` | 加法或拼接
`-` | `__sub__` | `__rsub__` | `__isub__` | 减法
`*` | `__mul__` | `__rmul__` | `__imul__` | 乘法或重复复制
`/` | `__truediv__` | `__rtruediv__` | `__itruediv__` | 除法
`//` | `__floordiv__` | `__rfloordiv__` | `__ifloordiv__` | 整除
`%` | `__mod__` | `__rmod__` | `__imod__` | 取模
`divmod()` | `__divmod__` | `__rdivmod__` | `__idivmod__` | 返回由整除的商和模数组成的元组
`**`，`pow()` | `__pow__` | `__rpow__` | `__ipow__` | 取幂 <br/>(`pow` 的第三个参数 `modulo` 是可选的：`pow(a, b, modulo)`，<br/>直接调用特殊方法时也支持这个参数, 如 `a.__pow__(b, modulo)`。)
`@` | `__matmul__` | `__rmatmul__` | `__imatmul__` | 矩阵乘法 <br/>(Python 3.5 新引入)
`&` | `__and__` | `__rand__` | `__iand__` | 位与
&#124; | `__or__` | `__ror__` | `__ior__` | 位或
`^` | `__xor__` | `__rxor__` | `__ixor__` | 位异或
`<<` | `__lshift__` | `__rlshift__` | `__ilshift__` | 按位左移
`>>` | `__rshift__` | `__rrshift__` | `__irshift__` | 按位右移

<span id="重载比较运算符"></span>

### 重载比较运算符

Python 解释器对众多比较运算符（`==`、`!=`、`>`、`<`、`>=`、`<=`）的处理与前文类似，不过在两个方面有重大区别。

- 正向和反向调用使用的是同一系列方法。例如:
  - 对 `==` 来说，正向和反向调用都是 `__eq__` 方法，只是把参数对调了；
  - 而正向的 `__gt__` 方法调用的是反向的 `__lt__` 方法，并把参数对调。
- 对 `==` 和 `!=` 来说，如果反向调用失败，`Python` 会比较对象的 `ID`，而不抛出 `TypeError` 。

分组 | 中缀运算符 | 正向方法调用 | 反向方法调用 | 后备机制
-----|-----------|-------------|-------------|----------
相等性 | `a == b` | `a.__eq__(b)` | `b.__eq__(a)` | 返回 `id(a) == id(b)`
| `a != b` | `a.__ne__(b)` | `b.__ne__(a)` | 返回 `not (a == b)`
排序 | `a > b` | `a.__gt__(b)` | `b.__lt__(a)` | 抛出 `TypeError`
| `a < b`  | `a.__lt__(b)`  | `b.__gt__(a)` | 抛出 `TypeError`
| `a >= b` | `a.__ge__(b)` | `b.__le__(a)` | 抛出 `TypeError`
| `a <= b` | `a.__le__(b)` | `b.__ge__(a)` | 抛出 `TypeError`

```python
    def __eq__(self, other):
        return (len(self) == len(other) and
                all(a == b for a, b in zip(self, other)))

    # 改进版
    def __eq__(self, other):
        if isinstance(other, Vector):  # 检查类型
            return (len(self) == len(other) and
                    all(a == b for a, b in zip(self, other)))
        else:
            return NotImplemented   # 尝试其他后备机制.
```

`!=` 运算符不用实现它，因为从 object 继承的 `__ne__` 方法的后备行为满足了我们的需求：定义了 `__eq__` 方法，而且它不返回 `NotImplemented`，`__ne__` 会对 `__eq__` 返回的结果取反。

<span id="重载增量运算符"></span>

### 重载增量运算符

- `+=` (`__iadd__`)
  - 就地修改左边的对象. 加
- `*=` (`__imul__`)
  - 就地修改左边的对象. 乘

如果一个类没有实现地运算符，增量赋值运算符只是语法糖：`a += b` 的作用与 `a = a + b` 完全一样。对不可变类型来说，这是预期的行为，而且，如果定义了 `__add__` 方法的话，不用编写额外的代码，`+=` 就能使用。

然而，如果实现了就地运算符方法，例如 `__iadd__`，计算 `a += b` 的结果时会调用就地运算符方法。这种运算符的名称表明，它们会就地修改左操作数，而不会创建新对象作为结果。

```python
>>> v1 = Vector([1, 2, 3])
>>> v1_alias = v1
>>> id(v1)
4302860128
>>> v1 += Vector([4, 5, 6])
>>> v1
Vector([5.0, 7.0, 9.0])
>>> id(v1)
4302859904
>>> v1_alias
Vector([1.0, 2.0, 3.0])
>>> v1 *= 11
>>> v1
Vector([55.0, 77.0, 99.0])
>>> id(v1)
4302858336
```

```python
import itertools  # 把导入标准库的语句放在导入自己编写的模块之前。

from tombola import Tombola
from bingo import BingoCage


class AddableBingoCage(BingoCage):  # 扩展 BingoCage

    def __add__(self, other):
        if isinstance(other, Tombola):  # 判断类型
            return AddableBingoCage(self.inspect() + other.inspect())  # 返回新对象.
        else:
            return NotImplemented

    def __iadd__(self, other):
        if isinstance(other, Tombola):
            other_iterable = other.inspect()  # 尝试使用 other 创建迭代器。
        else:
            try:
                other_iterable = iter(other)
            except TypeError:  # 抛出类型错误
                self_cls = type(self).__name__
                msg = "right operand in += must be {!r} or an iterable"
                raise TypeError(msg.format(self_cls))
        self.load(other_iterable)  # 载入自身实例中.
        return self  # 增量赋值特殊方法必须返回 self
```

编写代码过程中最好坚持 [pep8](https://www.python.org/dev/peps/pep-0008/#imports) 风格

<span id="扩展阅读"></span>

### 扩展阅读

- [Data Model](https://docs.python.org/3/reference/datamodel.html), 参考运算符特殊方法的权威资料.
  - [Implementing the arithmetic operations](https://docs.python.org/3/library/numbers.html#implementing-the-arithmetic-operations), 标准库中 numbers 模块文档也值得一读。
- [functools 模块](https://docs.python.org/3/library/functools.html#functools.total_ordering) 中的 `total_ordering` 函数是个类装饰器, （Python 2.7 及以上版本可用），它能为只定义了几个比较运算符的类自动生成全部比较运算符。
