---
nav: blog
layout: post
title: "流畅的python - 迭代器协议和生成器"
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

- [迭代器](#迭代器)
- [生成器](#生成器)
- [iter函数](#iter函数)
- [Sentence类的5个版本](#Sentence类的5个版本)
  - [不实现迭代器协议也可迭代的版本](#不实现迭代器协议也可迭代的版本)
  - [迭代器和可迭代对象分离](#迭代器和可迭代对象分离)
  - [生成器版本](#生成器版本)
  - [惰性实现](#惰性实现)
  - [生成器表达式](#生成器表达式)
- [等差数列生成器](#等差数列生成器)
- [标准库中的生成器](#标准库中的生成器)
- [yield from语句](#yieldfrom语句)
- [延伸阅读](#延伸阅读)

**惰性获取数据** ： 按需一次获取一个数据项。 【迭代器模式（Iterator pattern）】

生成器实现了迭代器协议，所以所有生成器都是迭代器。

- **迭代器：** 用于从集合中取出元素。
- **生成器：** 用于“凭空”生成元素。

<span id="iter函数"></span>

## iter函数

解释器需要迭代对象 `x` 时，会自动调用 `iter(x)` , `iter函数`会做如下检查:

- 检查对象是否实现了 `__iter__` 方法，如果实现了就调用它，获取一个迭代器。
- 如果没有实现 `__iter__` 方法，但是实现了 `__getitem__` 方法，Python 会创建一个迭代器，尝试按顺序（从索引 `0` 开始）获取元素。
- 如果尝试失败，Python 抛出 `TypeError` 异常，通常会提示“`C object is not iterable`”（C 对象不可迭代），其中 `C` 是目标对象所属的类。

- `iter` 函数的参数:
  - 第一个参数必须是可调用的对象，用于不断调用（没有参数），产出各个值；第二个值是哨符，这是个标记值，当可调用的对象返回这个值时，触发迭代器抛出 `StopIteration` 异常，而不产出哨符。
  - 参考文档: [iter](https://docs.python.org/3/library/functions.html#iter)

```python
In [5]: from random import randint

In [6]: def d4():
   ...:     return randint(1, 4)
   ...:

In [7]: d4_iter = iter(d4, 4)
In [13]: for roll in d4_iter:
    ...:     print(roll)
    ...:
2
3
1
2
2
3

# 逐行读取文件，直到遇到空行或者到达文件末尾为止：
with open('mydata.txt') as fp:
    for line in iter(fp.readline, '\n'):
        process_line(line)
```

<span id="迭代器"></span>

## 迭代器

**可迭代对象**:

- 如果对象实现了能返回迭代器的 `__iter__` 方法，那么对象就是可迭代的。
- 序列都可以迭代；
- 实现了 `__getitem__` 方法，而且其参数是从零开始的索引，这种对象也可以迭代。

**迭代器：**

- 实现了无参数的 `__next__` 方法，返回序列中的下一个元素；
- 如果没有元素了，那么抛出 StopIteration 异常。
- `Python` 中的迭代器还实现了 `__iter__` 方法，因此迭代器也可以迭代。

**可迭代的对象和迭代器之间的关系：** Python 从可迭代的对象中获取迭代器。

**退出迭代器：** `StopIteration` 异常表明迭代器到头了。Python 语言内部会处理 `for` 循环和其他迭代上下文（如列表推导、元组拆包，等等）中的 `StopIteration` 异常。

**检查对象是否可迭代：** 调用 `iter(x)` 函数，如果不可迭代，再处理 `TypeError` 异常。这比使用 `isinstance(x, abc.Iterable)` 更准确，因为 `iter(x)` 函数会考虑到遗留的 `__getitem__` 方法，而 `abc.Iterable `类则不考虑。

**检查对象是否是迭代器：** ： 根据 `Lib/_collections_abc.py` 中的实现逻辑，检查对象 `x` 是否为迭代器最好的方式是调用 `isinstance(x, abc.Iterator)`。得益于 `Iterator.__subclasshook__` 方法，即使对象 x 所属的类不是 `Iterator` 类的真实子类或虚拟子类，也能这样检查。

**标准的产生迭代器的两个方法:**

- **`__next__`** : 返回下一个可用的元素，如果没有元素了，抛出 `StopIteration` 异常。
- **`__iter__`** : 返回迭代器，例如 `self`。
- **注意**: 在 `Python 3` 中，`Iterator` 抽象基类定义的抽象方法是 `it.__next__()`，而在 Python 2 中是 `it.next()`。一如既往，应该避免直接调用特殊方法，使用 `next(it)` 即可，这个内置的函数在 `Python 2` 和 `Python 3` 中都能使用。

**`Iterator` 抽象基类实现：** [Iterator](https://github.com/python/cpython/blob/master/Lib/_collections_abc.py)

```python
class Iterator(Iterable):

    __slots__ = ()

    @abstractmethod
    def __next__(self):
        'Return the next item from the iterator. When exhausted, raise StopIteration'
        raise StopIteration

    def __iter__(self):
        return self  # 返回自身.

    @classmethod
    def __subclasshook__(cls, C):  # 检查子类. 这里检查是否实现了 __iter__ 和 __next__ 方法. 没有考虑到 __getitem__ 但 iter 函数考虑到了。
        if cls is Iterator:
            return _check_methods(C, '__iter__', '__next__')
        return NotImplemented

# 注册虚拟子类
Iterator.register(bytes_iterator)
Iterator.register(bytearray_iterator)
#Iterator.register(callable_iterator)
Iterator.register(dict_keyiterator)
Iterator.register(dict_valueiterator)
Iterator.register(dict_itemiterator)
Iterator.register(list_iterator)
Iterator.register(list_reverseiterator)
Iterator.register(range_iterator)
Iterator.register(longrange_iterator)
Iterator.register(set_iterator)
Iterator.register(str_iterator)
Iterator.register(tuple_iterator)
Iterator.register(zip_iterator)
```

<span id="Sentence类的5个版本"></span>

## Sentence类的5个版本

<span id="不实现迭代器协议也可迭代的版本"></span>

**第一版:**  不实现迭代器协议也可迭代的版本

```python
import re
import reprlib

RE_WORD = re.compile('\w+')

class Sentence:
    """通过索引从文本中提取单词"""

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)   # 匹配正则.

    def __getitem__(self, index):
        return self.words[index]

    def __len__(self):   # 序列协议的一部分. 仅仅为了让对象可迭代，可不实现.
        return len(self.words)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)   # 省略表达字符.
```


<span id="迭代器和可迭代对象分离"></span>

**第二版:** 迭代器和可迭代对象分离

```python
import re
import reprlib

RE_WORD = re.compile('\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):  # 可迭代对象协议的一部分.
        return SentenceIterator(self.words)  # 返回迭代器对象.


class SentenceIterator:

    def __init__(self, words):
        self.words = words
        self.index = 0

    def __next__(self):
        try:
            word = self.words[self.index]  # 获取单词
        except IndexError:
            raise StopIteration()  # 停止迭代.
        self.index += 1
        return word

    def __iter__(self):  # 返回迭代器本身.
        return self
```

构建可迭代的对象和迭代器时经常会出现错误，原因是混淆了二者。要知道，可迭代的对象有个 `__iter__` 方法， **每次都实例化一个新的迭代器**；

而迭代器要实现 `__next__` 方法，返回单个元素，此外还要实现 `__iter__` 方法，返回迭代器本身。

因此，迭代器可以迭代，但是可迭代的对象不是迭代器。【这句话有点混，但是理解了，就是 可迭代对象产生迭代器，本身不可迭代，但返回的迭代器是可迭代的，并且 `__iter__` 返回本身.】

除了 `__iter__` 方法之外，你可能还想在 `Sentence` 类中实现 `__next__` 方法，让 `Sentence` 实例既是可迭代的对象，也是自身的迭代器。可是，这种想法非常糟糕。根据有大量 `Python` 代码审查经验的 `Alex Martelli` 所说，这也是常见的反模式。

根据迭代器的 `适用性`：

- 访问一个聚合对象的内容而无需暴露它的内部表示
- 支持对聚合对象的多种遍历
- 为遍历不同的聚合结构提供一个统一的接口（即支持多态迭代）

为了“支持多种遍历”，必须能从同一个可迭代的实例中获取多个独立的迭代器，而且 `各个迭代器要能维护自身的内部状态` ，因此这一模式正确的实现方式是，每次调用 `iter(my_iterable)` 都新建一个独立的迭代器。这就是为什么这个示例需要定义 `SentenceIterator` 类。

**可迭代的对象一定不能是自身的迭代器。也就是说，可迭代的对象必须实现 `__iter__` 方法，但不能实现 `__next__` 方法。 另一方面，迭代器应该一直可以迭代。迭代器的 `__iter__` 方法应该返回自身。**

<span id="生成器版本"></span>

**第三版**: 生成器版本

```python
import re
import reprlib

RE_WORD = re.compile('\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
        for word in self.words:
            yield word  # 产出当前的 word。
        return  # 可省略return, 不管有没有 return 语句，生成器函数都不会抛出 StopIteration 异常，而是在生成完全部值之后会直接退出。
```

<span id="生成器"></span>

## 生成器

只要 Python 函数的定义体中有 `yield` 关键字，该函数就是生成器函数。调用生成器函数时，会返回一个生成器对象。也就是说，生成器函数是生成器工厂。

在生成器函数名称中 加上 `gen` 前缀或后缀 是个不错的选择。

<span id="惰性实现"></span>

**第四版:** 惰性实现

```python
import re
import reprlib

RE_WORD = re.compile('\w+')


class Sentence:

    def __init__(self, text):
        self.text = text  # 不再需要words.

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
        for match in RE_WORD.finditer(self.text):  # finditer 函数构建一个迭代器，包含 self.text 中匹配 RE_WORD 的单词，产出 MatchObject 实例。
            yield match.group()  # match.group() 方法从 MatchObject 实例中提取匹配正则表达式的具体文本。
```

设计 `Iterator` 接口时考虑到了惰性：`next(my_iterator)` 一次生成一个元素。
懒惰的反义词是急迫，其实，`惰性求值`（lazy evaluation）和 `及早求值`（eager evaluation）是编程语言理论方面的技术术语。

`re.finditer` 函数是 `re.findall` 函数的惰性版本，返回的不是列表，而是一个生成器，按需生成 `re.MatchObject` 实例。
如果有很多匹配，`re.finditer` 函数能节省大量内存。

<span id="生成器表达式"></span>

**第五版：** 生成器表达式

```python
import re
import reprlib

RE_WORD = re.compile('\w+')


class Sentence:

    def __init__(self, text):
      self.text = text

    def __repr__(self):
      return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
      return (match.group() for match in RE_WORD.finditer(self.text))  # 生成器表达式.
      # 这里不是生成器函数了（没有 yield），而是使用生成器表达式构建生成器，然后将其返回。不过，最终的效果一样：调用 __iter__ 方法会得到一个生成器对象。
```

**生成器表达式 （列表推导的惰性版本）**：

不会迫切地构建列表，而是返回一个生成器，按需惰性生成元素。
也就是说，如果列表推导是制造列表的工厂，那么生成器表达式就是制造生成器的工厂。

**推导式:**

**名称** | **例子**
-----|-----
列表推导 | `[l for l in range(11)]` -> `list`
生成器推导 | `(l for l in range(11))` - > `generator`
字典推导 | `{k:v for k,v in enumerate(range(11))}` -> `dict`
集合推导 | `{t for t in range(11)}` -> `set`
元组推导 | `tuple(t for t in range(11))` -> `tuple` 【tuple 嵌套表达式】

**何时使用生成器推导**:

遇到简单的情况时，可以使用生成器表达式，因为这样扫一眼就知道代码的作用.

生成器函数的优势：

如果生成器表达式要分成多行写，倾向于定义生成器函数，以便提高可读性。此外，生成器函数有名称，因此可以重用。

如果函数或构造方法只有一个参数，传入生成器表达式时不用写一对调用函数的括号，再写一对括号围住生成器表达式，只写一对括号就行了。 例如 上面的元祖推导. 和下面的.

```python
def __mul__(self, scalar):
    if isinstance(scalar, numbers.Real):
        return Vector(n * scalar for n in self)  #
    else:
        return NotImplemented
```

<span id="等差数列生成器"></span>

## 等差数列生成器

```python
class ArithmeticProgression:

    def __init__(self, begin, step, end=None):  # end 可选,默认生成无穷数列
        self.begin = begin
        self.step = step
        self.end = end  # None -> 无穷数列

    def __iter__(self):
        result = type(self.begin + self.step)(self.begin)  # 强制转换成加法算式得到的类型
        forever = self.end is None
        index = 0
        while forever or result < self.end:
            yield result  # 生成值
            index += 1
            result = self.begin + self.step * index  # 终止时，该值不会产出.

# 生成器函数形式：
def aritprog_gen(begin, step, end=None):
    result = type(begin + step)(begin)
    forever = end is None
    index = 0
    while forever or result < end:
        yield result
        index += 1
        result = begin + step * index
```

<span id="标准库中的生成器"></span>

## 标准库中的生成器

- `itertools.count` : 返回能生成多个数的生成器.如果不传入参数，会生成从零开始的整数数列。参考：[文档](http://python.usyiyi.cn/documents/python_352/library/itertools.html#itertools.count)
- `itertools.takewhile` : 返回一个使用生成器的生成器.在指定的条件计算结果为 `False` 时停止。参考: [文档](http://python.usyiyi.cn/documents/python_352/library/itertools.html#itertools.takewhile)

可根据这两个函数重写之前的等差数列函数:

```python
import itertools

def aritprog_gen(begin, step, end=None):
    """ 尽管没有yield关键字，所以不是生成器函数，
        但它会返回一个生成器，所以和生成器函数一样 """
    first = type(begin + step)(begin)
    ap_gen = itertools.count(first, step)
    if end is not None:
        ap_gen = itertools.takewhile(lambda n: n < end, ap_gen)
    return ap_gen
```

**其他函数:**

用于过滤:

模块 | 函数 | 说明
-----|------|-------
`itertools` | `compress(it, selector_it)` | 并行处理两个可迭代的对象；如果 `selector_it` 中的元素是真值，产出 `it` 中对应的元素
`itertools` | `dropwhile(predicate, it)` | 处理 `it` ，跳过 `predicate` 的计算结果为真值的元素，然后产出剩下的各个元素（不再进一步检查)
（内置） | `filter(predicate, it)` | 把 `it` 中的各个元素传给 `predicate` ，如果 `predicate(item)` 返回真值，那么产出对应的元素；如果 `predicate` 是 `None`，那么只产出真值元素
`itertools` | `filterfalse(predicate, it)` | 与 `filter` 函数的作用类似，不过 `predicate` 的逻辑是相反的：`predicate` 返回假值时产出对应的元素
`itertools` | `islice(it, stop)` 或 `(it, start, stop[, step])` | 产出 `it` 的切片，作用类似于 `s[:stop]` 或 `s[start:stop:step]`，不过 `it` 可以是任何可迭代的对象，而且这个函数实现的是惰性操作

用于映射:

模块 | 函数 | 说明
-----|------|-------
（内置） | `enumerate(iterable, start=0)` | 产出由两个元素组成的元组，结构是 `(index, item)`，其中 `index` 从 `start` 开始计数，`item` 则从 `iterable` 中获取
（内置） | `map(func, it1, [it2, ..., itN])` | 把 `it` 中的各个元素传给 `func` ，产出结果；如果传入 `N` 个可迭代的对象，那么 `func` 必须能接受 `N` 个参数，而且要并行处理各个可迭代的对象
`itertools` | `accumulate(it, [func])` | 产出累积的总和；如果提供了 `func` ，那么把前两个元素传给它，然后把计算结果和下一个元素传给它，以此类推，最后产出结果
`itertools` | `starmap(func, it)` | 把 `it` 中的各个元素传给 `func`，产出结果；输入的可迭代对象应该产出可迭代的元素 `iit` ，然后以 `func(*iit)` 这种形式调用 `func`

用于合并:

模块 | 函数 | 说明
-----|------|-------
`itertools` |` chain(it1, ..., itN)` | 先产出 `it1` 中的所有元素，然后产出 `it2` 中的所有元素，以此类推，无缝连接在一起
`itertools` | `chain.from_iterable(it)` | 产出 `it` 生成的各个可迭代对象中的元素，一个接一个，无缝连接在一起； `it` 应该产出可迭代的元素，例如可迭代的对象列表
`itertools` | `product(it1, ..., itN, repeat=1)` | 计算笛卡儿积：从输入的各个可迭代对象中获取元素，合并成由 `N` 个元素组成的元组，与嵌套的 `for` 循环效果一样； `repeat` 指明重复处理多少次输入的可迭代对象
（内置） | `zip(it1, ..., itN)` | 并行从输入的各个可迭代对象中获取元素，产出由 `N` 个元素组成的元组，只要有一个可迭代的对象到头了，就默默地停止
`itertools` | `zip_longest(it1, ..., itN, fillvalue=None)` | 并行从输入的各个可迭代对象中获取元素，产出由 `N` 个元素组成的元组，等到最长的可迭代对象到头后才停止，空缺的值使用 `fillvalue` 填充

用于扩展:

模块 | 函数 | 说明
-----|------|-------
`itertools` | `combinations(it, out_len)` | 把 `it` 产出的 `out_len` 个元素组合在一起，然后产出
`itertools` | `combinations_with_replacement(it, out_len)` | 把 `it` 产出的 `out_len` 个元素组合在一起，然后产出，包含相同元素的组合
`itertools` | `count(start=0, step=1)` | 从 `start` 开始不断产出数字，按 `step` 指定的步幅增加
`itertools` | `cycle(it)` | 从 `it` 中产出各个元素，存储各个元素的副本，然后按顺序重复不断地产出各个元素
`itertools` | `permutations(it, out_len=None)` | 把 `out_len` 个 `it` 产出的元素排列在一起，然后产出这些排列；`out_len` 的默认值等于 `len(list(it))`
`itertools` | `repeat(item, [times])` | 重复不断地产出指定的元素，除非提供 `times` ，指定次数

用于重新排列元素:

模块 | 函数 | 说明
-----|------|-------
`itertools` | `groupby(it,key=None)` | 产出由两个元素组成的元素，形式为 (`key`, `group`)，其中 `key` 是分组标准，`group` 是生成器，用于产出分组里的元素
（内置） | `reversed(seq)` | 从后向前，倒序产出 `seq` 中的元素；`seq` 必须是序列，或者是实现了 `__reversed__` 特殊方法的对象
`itertools` | `tee(it, n=2)` | 产出一个由 `n` 个生成器组成的元组，每个生成器用于单独产出输入的可迭代对象中的元素

<span id="yieldfrom语句"></span>

归约函数:

模块 | 函数 | 说明
-----|------|-------
（内置） | `all(it)` | `it` 中的所有元素都为真值时返回 `True`，否则返回 `False`；`all([])` 返回 `True`
（内置） | `any(it)` | 只要 `it` 中有元素为真值就返回 `True`，否则返回 `False`；`any([])` 返回 `False`
（内置） | `max(it, [key=,] [default=])` | 返回 `it` 中值最大的元素；`key` 是排序函数，与 `sorted` 函数中的一样；如果可迭代的对象为空，返回 `default` .
（内置） | `min(it, [key=,] [default=])` | 返回 `it` 中值最小的元素；`key` 是排序函数，与 `sorted` 函数中的一样；如果可迭代的对象为空，返回 `default` .
（内置） | `sum(it, start=0)` | `it` 中所有元素的总和，如果提供可选的 `start` ，会把它加上（计算浮点数的加法时，可以使用 `math.fsum` 函数提高精度）
`functools` | `reduce(func, it, [initial])` | 把前两个元素传给 `func`，然后把计算结果和第三个元素传给 `func`，以此类推，返回最后的结果；如果提供了 `initial` ，把它当作第一个元素传入

内置函数 `sorted` 接受一个可迭代的对象，返回不同的值, 列表.

**注意:** `sorted` 和这些归约函数只能处理最终会停止的可迭代对象。否则，这些函数会一直收集元素，永远无法返回结果。

<span id="yieldfrom语句"></span>

## yield from语句

pep 参考: [PEP 380 — Syntax for Delegating to a Subgenerator](https://www.python.org/dev/peps/pep-0380/)

惰性读取另一个生成器中值.

```python
def chain(*iterables):
    for i in iterables:
        yield from i

# 示例：
In [2]: s = 'ABC'

In [3]: t = range(3)

In [4]: list(chain(s,t))
Out[4]: ['A', 'B', 'C', 0, 1, 2]
```

<span id="延伸阅读"></span>

## 延伸阅读

- [Yield expressions](https://docs.python.org/3/reference/expressions.html#yieldexpr), 从技术层面深入说明了生成器.
- [PEP 255—Simple Generators](https://www.python.org/dev/peps/pep-0255/), 定义生成器函数的 PEP.
- [itertools 模块的文档](https://docs.python.org/3/library/itertools.html), 包含大量示例.
- [Itertools Recipes](https://docs.python.org/3/library/itertools.html#itertools-recipes), 说明如何使用 `itertools` 模块中的现有函数实现额外的高性能函数。
- [PEP 380: Syntax for Delegating to a Subgenerator](https://docs.python.org/3/whatsnew/3.3.html#pep-380-syntax-for-delegating-to-a-subgenerator), 说明了 `yield from` 句法。
