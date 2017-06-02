---
nav: blog
layout: post
title: "流畅的python - 从协议到抽象类"
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

- [python中的接口和协议](#python中的接口和协议)
- [猴子补丁](#猴子补丁)
- [抽象类](#抽象类)
- [实现抽象类](#实现抽象类)
- [标准库中的抽象类](#标准库中的抽象类)
- [定义抽象类](#定义抽象类)
- [子类测试机制](#子类测试机制)
- [扩展阅读](#扩展阅读)

<span id="python中的接口和协议"></span>

### python中的接口和协议

- **接口**
  - `类实现或继承的公开属性`（方法或数据属性），包括特殊方法，如 `__getitem__` 或 `__add__`。
  - 即便“受保护的”属性也只是采用命名约定实现的（单个前导下划线）；私有属性可以轻松地访问，原因也是如此。不要违背这些约定。
  - 补充定义：`对象公开方法的子集，让对象在系统中扮演特定的角色。`
  - 协议是接口，但不是正式的（只由文档和约定定义），因此协议不能像正式接口那样施加限制.
  - 一个类可能只实现部分接口，这是允许的。有时，某些 API 只要求“文件类对象”返回字节序列的 `.read()` 方法。在特定的上下文中可能需要其他文件操作方法，也可能不需要。
  - **序列：**
    - 如果没有 `__iter__` 和 `__contains__` 方法，Python 会调用 `__getitem__` 方法，设法让迭代和 `in` 运算符可用。

<span id="猴子补丁"></span>

### 猴子补丁

**在运行时修改类或模块，而不改动源码。** 猴子补丁很强大，但是打补丁的代码与要打补丁的程序耦合十分紧密，而且往往要处理隐藏和没有文档的部分。

“ **鸭子类型** ”：对象的类型无关紧要，只要实现了特定的协议即可。

```python
>>> from random import shuffle
>>> l = list(range(10))
>>> shuffle(l)  # 就地打乱序列
>>> l
[5, 2, 9, 7, 8, 3, 1, 4, 0, 6]

>>> from random import shuffle
>>> from frenchdeck import FrenchDeck
>>> deck = FrenchDeck()
>>> shuffle(deck)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File ".../python3.3/random.py", line 265, in shuffle
    x[i], x[j] = x[j], x[i]
TypeError: 'FrenchDeck' object does not support item assignment

>>> def set_card(deck, position, card): # 定义一个函数，参数分别为 实例对象，位置，元素
...     deck._cards[position] = card  # 隐含条件， 要知道 deck 对象有一个名为 _cards 的属性，而且 _cards 的值必须是可变序列
...
>>> FrenchDeck.__setitem__ = set_card  # 赋值给 __setitem__ 方法， 实现接口。即 猴子补丁。
>>> shuffle(deck)  # 可以打乱 deck 了，因为 FrenchDeck 实现了可变序列协议所需的方法。
>>> deck[:5]
[Card(rank='3', suit='hearts'), Card(rank='4', suit='diamonds'), Card(rank='4',
suit='clubs'), Card(rank='7', suit='hearts'), Card(rank='9', suit='spades')]

# 强调协议是动态的： random.shuffle 函数不关心参数的类型，只要那个对象实现了部分可变序列协议即可。即便对象一开始没有所需的方法也没关系，后来再提供也行。
```

<span id="抽象类"></span>

### 抽象类

`cls` 的元类是 `abc.ABCMeta`，就可以使用 `isinstance(obj, cls)`, `cls` 就是抽象类。

[`collections.abc` 中有很多有用的抽象类（Python 标准库的 `numbers` 模块中还有一些）。]

**优点：** 可以使用 `register` 类方法在终端用户的代码中把某个类“声明”为一个抽象类的“虚拟”子类. 这大大地打破了严格的强耦合，与面向对象编程人员掌握的知识有很大出入.

```python
# 抽象类的本质就是几个特殊方法

>>> class Struggle:
...     def __len__(self): return 23
...
>>> from collections import abc
# （要使用正确的句法和语义实现，前者要求没有参数，后者要求返回一个非负整数，指明对象的长度；如果不使用规定的句法和语义实现特殊方法，如 __len__，会导致非常严重的问题）
>>> isinstance(Struggle(), abc.Sized)  # 无需注册，abc.Sized 也能把 Struggle 识别为自己的子类，只要实现了特殊方法 __len__ 即可
True
```

如果实现的类体现了 `numbers` 、 `collections.abc` 或其他框架中抽象类的概念，要么继承相应的抽象类（必要时），要么把类注册到相应的抽象类中。
开始开发程序时，不要使用提供注册功能的库或框架，要自己动手注册；如果必须检查参数的类型（这是最常见的），例如检查是不是“序列”，那就这样做：

```python
isinstance(the_arg, collections.abc.Sequence)
```

**不要在生产代码中定义抽象类（或元类）**

```
当然，你还可以自己定义抽象类，但是我不建议高级 Python 程序员之外的人这么做；
同样，我也不建议你自己定义元类……我说的“高级 Python 程序员”是指对 Python 语言的一招一式都了如指掌，即便对这类人来说，抽象类和元类也不是常用工具。
如此“深层次的元编程”，如果可以这么讲的话，适合框架的作者使用，这样便于众多不同的开发团队独立扩展框架……真正需要这么做的“高级 Python 程序员”不超过 1%。
    ——Alex Martelli
```

**兼容性问题：**
  - 在 Python 3.4 中没有能把字符串和元组或其他不可变序列区分开的抽象类，因此必须测试 `str`。
  - 在 Python 2 中，`basestr` 类型可以协助这样的测试。`basestr` 不是抽象类，但它是 `str` 和 `unicode` 的超类；
  - 然而，Python 3 把 `basestr` 去掉了。奇怪的是，Python 3 中有个 `collections.abc.ByteString` 类型，但是它只能检测 `bytes` 和 `bytearray` 类型。

`抽象类`是用于封装框架引入的一般性概念和抽象的，例如“一个序列”和“一个确切的数”。（读者）基本上不需要自己编写新的抽象类，只要正确使用现有的抽象类，就能获得 99.9% 的好处，而不用冒着设计不当导致的巨大风险。

<span id="实现抽象类"></span>

### 实现抽象类

```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck2(collections.MutableSequence):   # 利用现有的抽象基类 ， 序列抽象类
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]

    def __setitem__(self, position, value):  # 支持洗牌, 为 random.shuffle 提供接口.
        self._cards[position] = value

    def __delitem__(self, position):  # 作为子类，必须实现抽象类 MutableSequence 的 抽象方法
        del self._cards[position]

    def insert(self, position, value):  # 作为子类，必须实现抽象类 MutableSequence 的 抽象方法
        self._cards.insert(position, value)
```

导入时（加载并编译模块时），Python 不会检查抽象方法的实现，`在运行时实例化类时才会真正检查`。

因此，如果没有正确实现某个抽象方法，Python 会抛出 TypeError 异常，并把错误消息设为"Can't instantiate abstract class FrenchDeck2 with abstract methods `__delitem__`, insert"。

正是这个原因，即便 FrenchDeck2 类不需要 `__delitem__` 和 `insert` 提供的行为，也要实现，因为 `MutableSequence` 抽象类需要它们。

子类可以覆盖从抽象基类中继承的方法，以更高效的方式重新实现。

参考： [collections.MutableSequence](http://python.usyiyi.cn/documents/python_352/library/collections.abc.html#collections.abc.MutableSequence)

<span id="标准库中的抽象类"></span>

### 标准库中的抽象类

大多数抽象类在 `collections.abc` 模块中定义，不过其他地方也有。

例如，`numbers` 和 `io` 包中有一些抽象基类。但是，`collections.abc` 中的抽象基类最常用。

- `collections.abc`模块中的抽象类

![抽象类]({% link assets/programingimg/abc.png %})

- `Iterable`、`Container` 和 `Sized`:
  - 各个集合应该继承这三个抽象基类，或者至少实现兼容的协议。
  - `Iterable` 通过 `__iter__` 方法支持迭代，`Container` 通过 `__contains__` 方法支持 `in` 运算符，`Sized` 通过 `__len__` 方法支持 `len()` 函数。
- `Sequence`、`Mapping` 和 `Set`:
  - 这三个是主要的不可变集合类型，而且各自都有可变的子类。
- `MappingView` :
  - 在 Python 3 中，映射方法 `.items()`、`.keys()` 和 `.values()` 返回的对象分别是 `ItemsView`、`KeysView` 和 `ValuesView` 的实例。前两个类还从 `Set` 类继承了丰富的接口.
- `Callable` 和 `Hashable` :
  - 这两个抽象基类与集合没有太大的关系，只不过因为` collections.abc` 是标准库中定义抽象基类的第一个模块，而它们又太重要了，因此才把它们放到 `collections.abc` 模块中。
  - 这两个抽象基类的主要作用是为内置函数 `isinstance` 提供支持，以一种安全的方式判断对象能不能调用或散列。
  - 若想检查是否能调用，可以使用内置的 `callable()` 函数；但是没有类似的 `hashable() `函数，因此测试对象是否可散列，最好使用 `isinstance(my_obj, Hashable)`。
- `Iterator`:
  - 注意它是 Iterable 的子类 , 在第十四章讨论。

- **标准库中数值的抽象类**

  [`numbers`](http://python.usyiyi.cn/translate/python_352/library/numbers.html) 包定义的是“数字塔”（即各个抽象基类的层次结构是线性的），其中 Number 是位于最顶端的超类，随后是 Complex 子类，依次往下，最底端是 Integral 类：

- `Number`
- `Complex`
- `Real`
- `Rational`
- `Integral`

因此，如果想检查一个数是不是整数，可以使用 `isinstance(x, numbers.Integral)`，这样代码就能接受 `int`、`bool`（`int` 的子类），或者外部库使用 `numbers` 抽象基类注册的其他类型。
为了满足检查的需要，你或者你的 API 的用户始终可以把兼容的类型注册为 `numbers.Integral `的虚拟子类。

与之类似，如果一个值可能是浮点数类型，可以使用 `isinstance(x, numbers.Real)` 检查。这样代码就能接受 bool、int、float、fractions.Fraction，或者外部库（如 NumPy，它做了相应的注册）提供的非复数类型。

<span id="定义抽象类"></span>

### 定义抽象类

```python
import abc

class Tombola(abc.ABC):  # 自己定义的抽象基类要继承 abc.ABC

    @abc.abstractmethod
    def load(self, iterable):  # 抽象方法使用 @abstractmethod 装饰器标记，而且定义体中通常只有文档字符串
    # 在抽象基类出现之前，抽象方法使用 raise NotImplementedError 语句表明由子类负责实现。
        """从可迭代对象中添加元素。"""

    @abc.abstractmethod
    def pick(self):
        """随机删除元素，然后将其返回。

        如果实例为空，这个方法应该抛出`LookupError`。
        """

    # 可以包含具体方法
    def loaded(self):
        """如果至少有一个元素，返回`True`，否则返回`False`。"""
        return bool(self.inspect())  # 抽象类中的具体方法只能依赖抽象类定义的接口（即只能使用抽象类中的其他具体方法、抽象方法或特性）。

    def inspect(self):
        """返回一个有序元组，由当前元素构成。"""
        items = []
        while True:
            try:
                items.append(self.pick())
            except LookupError:
                break
        self.load(items)
        return tuple(sorted(items))

# 执行：
>>> from tombola import Tombola
>>> class Fake(Tombola):  # 声明为 Tombola 的子类。
...     def pick(self):
...         return 13
...
>>> Fake  # 确认类
<class '__main__.Fake'>
>>> f = Fake()  # Python 认为 Fake 是抽象类，因为它没有实现 load 方法，这是 Tombola 抽象基类声明的抽象方法之一。
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class Fake with abstract methods load
```

抽象方法可以有实现代码。即便实现了，子类也必须覆盖抽象方法，但是在子类中可以使用 `super()` 函数调用抽象方法，为它添加功能，而不是从头开始实现。

`@abstractmethod` 装饰器的用法参见 [abc 模块的文档](http://python.usyiyi.cn/documents/python_352/library/collections.abc.html)。

**抽象类定义方式：**
  - 继承 `abc.ABC` 或其他抽象基类. [python3.4及更新版本]
  - `class Tombola(metaclass=abc.ABCMeta):`  python3旧版.
  - python2 中使用 `__metaclass__` 类属性.

```python
class Tombola(object):  # 这是Python 2！！！
    __metaclass__ = abc.ABCMeta
    # ...

# 装饰器可以折叠, 但堆叠装饰器的顺序通常很重要，@abstractmethod 的文档就特别指出：
# 与其他方法描述符一起使用时，abstractmethod() 应该放在最里层，
# 也就是说，在 @abstractmethod 和 def 语句之间不能有其他装饰器。
# 参考： http://python.usyiyi.cn/translate/python_352/library/abc.html#abc.abstractmethod
class MyABC(abc.ABC):
    @classmethod
    @abc.abstractmethod
    def an_abstract_classmethod(cls, ...):
        pass
```

除了 `@abstractmethod` 之外，`abc` 模块还定义了 `@abstractclassmethod`、`@abstractstaticmethod` 和 `@abstractproperty` 三个装饰器。然而，后三个装饰器从 Python 3.3 起废弃了，因为装饰器可以在 `@abstractmethod` 上堆叠，那三个就显得多余了。

<span id="虚拟子类"></span>

### 虚拟子类

即便不继承，也有办法把一个类注册为抽象类的虚拟子类。

这样做时，保证注册的类忠实地实现了抽象基类定义的接口，而 Python 会相信我们，从而不做检查。如果说谎了，那么常规的运行时异常会被捕获。

注册虚拟子类的方式是在抽象基类上调用 `register` 方法.

这么做之后，注册的类会变成抽象基类的虚拟子类，而且 `issubclass` 和 `isinstance` 等函数都能识别，但是注册的类不会从抽象基类中继承任何方法或属性。

`虚拟子类不会继承注册的抽象基类，而且任何时候都不会检查它是否符合抽象基类的接口，即便在实例化时也不会检查。为了避免运行时错误，虚拟子类要实现所需的全部方法。`

`register` 方法通常作为普通的函数调用，不过也可以作为装饰器使用。

```python
from random import randrange

from tombola import Tombola

@Tombola.register  # 注册为 Tombola 的虚拟子类。
class TomboList(list):  # 扩展 list

    def pick(self):
        if self:  # 从 list 中继承 __bool__ 方法，列表不为空时返回 True。
            position = randrange(len(self))
            return self.pop(position)  # 调用继承自 list 的 self.pop 方法，传入一个随机的元素索引。
        else:
            raise LookupError('pop from empty TomboList')

    load = list.extend  # 与 list.extend 一样

    def loaded(self):
        return bool(self)  # 委托 bool 函数。

    def inspect(self):
        return tuple(sorted(self))

# Tombola.register(TomboList)  # 如果是 Python 3.3 或之前的版本，不能把 .register 当作类装饰器使用，必须使用标准的调用句法。

# 注册之后，可以使用 issubclass 和 isinstance 函数判断 TomboList 是不是 Tombola 的子类：
>>> from tombola import Tombola
>>> from tombolist import TomboList
>>> issubclass(TomboList, Tombola)
True
>>> t = TomboList(range(100))
>>> isinstance(t, Tombola)
True
```

可通过查看[collections_abc模块源码](https://github.com/python/cpython/blob/master/Lib/_collections_abc.py)， 是这样把内置类型 `tuple`、`str`、`range` 和 `memoryview` 注册为 `Sequence` 的虚拟子类的 ：

```python
Sequence.register(tuple)
Sequence.register(str)
Sequence.register(range)
Sequence.register(memoryview)
```

特殊的类属性 `__mro__` 中指定类的继承关系. 即方法解析顺序（Method Resolution Order）这个属性的作用很简单，按顺序列出类及其超类，Python 会按照这个顺序搜索方法。

```python
>>> TomboList.__mro__
(<class 'tombolist.TomboList'>, <class 'list'>, <class 'object'>)
```

<span id="子类测试机制"></span>

### 子类测试机制

测试子类时用到 2 个 类 属性， `__subclasses__()` 和 `_abc_registry` .

- **__subclasses__**
  - 这个方法返回类的直接子类列表，不含虚拟子类。
- **_abc_registry**
  - 只有抽象基类有这个数据属性，其值是一个 `WeakSet` 对象，即抽象类注册的虚拟子类的弱引用。
- **`__subclasshook__`特殊方法.**
  - 可以完全使用不相关的类，只要实现特定的方法即可, 当然，只有提供 `__subclasshook__` 方法的 `抽象类` 才能这么做。
  - 在自己定义的抽象基类中要不要实现 `__subclasshook__` 方法呢？可能不需要。

```python
In [25]: class Struggle:
    ...:     def __len__(self): return 23
    ...:

In [26]: from collections import abc

In [27]: isinstance(Struggle(), abc.Sized)
Out[27]: True

In [28]: issubclass(Struggle,abc.Sized)  # 经 issubclass 函数确认，Struggle 是 abc.Sized 的子类
Out[28]: True

# https://github.com/python/cpython/blob/master/Lib/_collections_abc.py
# 通过查看 Sized 源码可窥探，Sized 的源码：
def _check_methods(C, *methods):
    mro = C.__mro__
    for method in methods:
        for B in mro:
            if method in B.__dict__:
                if B.__dict__[method] is None:
                    return NotImplemented
                break
        else:
            return NotImplemented
    return True

class Sized(metaclass=ABCMeta):

    __slots__ = ()   # 防止被动态赋予属性？

    @abstractmethod
    def __len__(self):
        return 0

    @classmethod
    def __subclasshook__(cls, C):  # C 是否为 cls 的子类.
        if cls is Sized:
            return _check_methods(C, "__len__")  # 表明 C 是否为 cls 的子类.
        return NotImplemented  # 返回 让子类检查。
```

子类检查的细节，参考：[ABCMeta.__subclasscheck__ 方法的源码](https://github.com/python/cpython/blob/master/Lib/abc.py), 提醒：源码中有很多 if 语句和两个递归调用。

```
尽管抽象基类使得类型检查变得更容易了，但不应该在程序中过度使用它。
Python 的核心在于它是一门动态语言，它带来了极大的灵活性。
如果处处都强制实行类型约束，那么会使代码变得更加复杂，而本不应该如此。
我们应该拥抱 Python 的灵活性。

    ——David Beazley 和 Brian Jones
    《Python Cookbook（第 3 版）中文版》
```

<span id="扩展阅读"></span>

### 扩展阅读

- [PEP 3119—Introducing Abstract Base Classes](https://www.python.org/dev/peps/pep-3119), 讲解了抽象基类的基本原理
- [PEP 3141—A Type Hierarchy for Numbers](https://www.python.org/dev/peps/pep-3141/), 提出了 numbers 模块中的抽象基类。
- [Contracts in Python: A Conversation with Guido van Rossum, Part IV](http://www.artima.com/intv/pycontract.html), Bill Venners 对 Guido van Rossum 的采访讨论了动态类型的优缺点。
- [zope.interface 包](http://docs.zope.org/zope.interface/), 提供了一种声明接口的方式：检查对象是否实现了接口，注册提供方，然后查询指定接口的提供方。
  - 一开始，这个包是 `Zope 3` 核心的一部分，不过它可以在 `Zope` 外部使用，而且已经有人这么做了。
  - 这个包为大型 Python 项目（如 `Twisted`、`Pyramid` 和 `Plone`）的组件式架构提供了灵活的基础。
  - [A Python Component Architecture](https://regebro.wordpress.com/2007/11/16/a-python-component-architecture/), Lennart Regebro 写的对 `zope.interface` 包的介绍
  - [A Comprehensive Guide to Zope Component Architecture](http://muthukadan.net/docs/zca.html), Baiju M 还写的一本相关的书。
