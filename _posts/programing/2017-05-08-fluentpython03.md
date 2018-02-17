---
nav: blog
layout: post
title: "流畅的python - 字典和集合"
author: "wangchao"
tags:
  - python
  - '数据模型'
  - '映射类型'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [字典的定义形式](#字典的定义形式)
  - [字典的影响](#字典的影响)
- [集合的定义形式](#集合的定义形式)
  - [集合的影响](#集合的影响)
- [散列的实现](#散列的实现)
- 其他散列类型
  - [defaultdict](#defaultdict)
  - [OrderedDict](#OrderedDict)
  - [ChainMap](#ChainMap)
  - [Counter](#Counter)
  - [UserDict](#UserDict)
  - [MappingProxyType](#MappingProxyType)
- [代码片段](#代码片段)

**字典和集合:**

泛映射类型 的抽象类： `collections.abc`模块中的 `Mapping` 和 `MutableMapping` ， 他们两为 `dict` 和其他类似的类型定义接口. 【在 Python 2.6 到 Python 3.2 的版本中，这些类还不属于 `collections.abc` 模块，而是隶属于 `collections` 模块】


![dict_mapping]({% link assets/programingimg/dict_mapping.png %})

**可散列类型：**

- `str` 、 `bytes` 、 `frozenset`【根据定义,只能容纳可散列类型】、`tuple`【包含的所有元素都是可散列类型的情况下，它才是可散列的】
- 如果一个对象是可散列的，那么在这个对象的生命周期中，它的散列值是不变的，而且这个对象需要实现 `__hash__()` 方法。另外可散列对象还要有 `__qe__()` 方法，这样才能跟其他键做比较。如果两个可散列对象是相等的，那么它们的散列值一定是一样的。 参考：[hashable](https://docs.python.org/3/glossary.html#term-hashable)
- 自定义的类型的对象都是可散列的，散列值就是它们的 `id()` 函数的返回值.如果一个对象实现了 `__eq__` 方法，并且在方法中用到了这个对象的内部状态的话，那么只有当所有这些内部状态都是不可变的情况下，这个对象才是可散列的。【没有实现`__eq__` 方法时，自定义对象是可散列的，否则需要保持`__eq__`使用的内部状态不可变，才可散列.】

<span id="字典的定义形式"></span>

- **字典定义形式：**

```python
# 判定某个数据是不是广义上的映射类型
>>> my_dict = {}
>>> isinstance(my_dict, abc.Mapping)
True
# 字典构造
>>> a = dict(one=1, two=2, three=3)
>>> b = {'one': 1, 'two': 2, 'three': 3}
>>> c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
>>> d = dict([('two', 2), ('one', 1), ('three', 3)])
>>> e = dict({'three': 3, 'one': 1, 'two': 2})
>>> a == b == c == d == e
True
>>> DIAL_CODES = [
...         (86, 'China'),
...         (91, 'India'),
...         (1, 'United States'),
...         (62, 'Indonesia'),
...         (55, 'Brazil'),
...         (92, 'Pakistan'),
...         (880, 'Bangladesh'),
...         (234, 'Nigeria'),
...         (7, 'Russia'),
...         (81, 'Japan'),
...     ]
>>> country_code = {country: code for code, country in DIAL_CODES}  # 字典推导
>>> country_code
{'China': 86, 'India': 91, 'Bangladesh': 880, 'United States': 1,
'Pakistan': 92, 'Japan': 81, 'Russia': 7, 'Brazil': 55, 'Nigeria':
234, 'Indonesia': 62}
```

**标准库中常见的字典类型对比：**

`dict` 和 `collections.defaultdict` 和 `collections.OrderedDict`

**方法名** | **dict** | **defaultdict** | **OrderedDict** | **说明**
----------|---------|----------|--------|--------
`d.__contains__(k)` | • | • | • | 检查 `k` 是否在 `d` 中
`d.__copy__()` | | • | | 用于支持 `copy.copy`
`d.__len__()` | • | • | • | 可以用 `len(d)` 的形式得到字典里键值对的数量
`d.__setitem__(k, v)` | • | • | • | 实现 `d[k] = v` 操作，把 `k` 对应的值设为 `v`
`d.__getitem__(k)` | • | • | • | 让字典 `d` 能用`d[k]`的形式返回键`k`对应的值
`d.__delitem__(k)` | • | • | • | `del d[k]`，移除键为 `k` 的元素
`d.__iter__()` | • | • | • | 获取键的迭代器
`d.keys()` | • | • | • | 获取所有的键
`d.values()` | • | • | • | 返回字典里的所有值
`d.copy()` | • | • | • | 浅复制
`d.clear()` | • | • | • | 移除所有元素
`d.update(m, [**kargs])` | • | • | • | `m` 可以是映射或者键值对迭代器，用来更新 `d` 里对应的条目
`d.fromkeys(it, [initial])` | • | • | • | 将迭代器 `it` 里的元素设置为映射里的键，如果有 `initial` 参数，就把它作为这些键对应的值（默认是 `None`）
`d.setdefault(k, [default])` | • | • | • | 若字典里有键k，则把它对应的值设置为 `default` ，然后返回这个值；若无，则让 `d[k] = default`，然后返回 `default`
`d.get(k, [default])` | • | • | • | 返回键 `k` 对应的值，如果字典里没有键 `k` ，则返回 `None` 或者 `default`
`d.pop(k, [defaul]` | • | • | • | 返回键 `k` 所对应的值，然后移除这个键值对。如果没有这个键，返回 `None` 或者 `defaul`
`d.items()` | • | • | • | 返回 `d` 里所有的键值对
`d.popitem()` | • | • | • | 随机返回一个键值对并从字典里移除它<br/>【会移除字典里最先插入的元素（先进先出）；同时这个方法还有一个可选的 last 参数，若为真，则会移除最后插入的元素（后进先出）。】
`d.__missing__(k)` | | • | | 当 `__getitem__` 找不到对应键的时候，这个方法会被调用
`d.default_factory` | | • | | 在 `__missing__` 函数中被调用的函数，用以给未找到的元素设置值.<br/>【并不是一个方法，而是一个可调用对象（callable），它的值在 defaultdict 初始化的时候由用户设定。】
`d.move_to_end(k, [last])` | | | • | 把键为 `k` 的元素移动到最靠前或者最靠后的位置（`last` 的默认值是 `True`）
`d.__reversed__()` | | | • | 返回倒序的键的迭代器

**setdefault用法**

```python
"""创建从一个单词到其出现情况的映射"""

import sys
import re

WORD_RE = re.compile(r'\w+')

index = {}
with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start()+1
            location = (line_no, column_no)
            index.setdefault(word, []).append(location)  # 如果单词不存在，把单词和一个空列表放进映射，然后返回这个空列表.

# 以字母顺序打印出结果
for word in sorted(index, key=str.upper):
    print(word, index[word])
```

<span id="defaultdict"></span>

**collections.defaultdict类型**

在用户创建 defaultdict 对象的时候，就需要给它配置一个为找不到的键创造默认值的方法。

```python
"""创建一个从单词到其出现情况的映射"""

import sys
import re
import collections

WORD_RE = re.compile(r'\w+')

index = collections.defaultdict(list)      # 默认值是list.
with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start()+1
            location = (line_no, column_no)
            index[word].append(location)   # 如果 index 并没有 word 的记录，那么 default_factory 会被调用，为查询不到的键创造一个值。

# 以字母顺序打印出结果
for word in sorted(index, key=str.upper):
    print(word, index[word])
```

如果在创建 `defaultdict` 的时候没有指定 `default_factory` ，查询不存在的键会触发 KeyError。

`defaultdict` 里的 `default_factory` 只会在 `__getitem__` 里被调用，在其他的方法里完全不会发挥作用。 比如: `.get(k)` 方法. 即使没有 `k` . 也还是返回 `None`

其实是特殊方法 `__missing__` 。它会在 `defaultdict` 遇到找不到的键的时候调用 `default_factory` ，而实际上这个特性是所有映射类型都可以选择去支持的。

也就是说，如果有一个类继承了 `dict` ，然后这个继承类提供了 `__missing__` 方法，那么在 `__getitem__` 碰到找不到的键的时候，Python 就会自动调用它，而不是抛出一个 `KeyError` 异常。 【 `__missing__` 方法只会被 `__getitem__` 调用（比如在表达式 `d[k]` 中）, 对  `get` 或者 `__contains__` （in 运算符会用到这个方法）这些方法的使用没有影响】

**自定义的字典**

```python
class StrKeyDict0(dict):  # 继承自 dict。

    def __missing__(self, key):
        if isinstance(key, str):  # 如果找不到的键本身就是字符串，那就抛出 KeyError 异常。
            raise KeyError(key)
        return self[str(key)]  # 如果找不到的键不是字符串，那么把它转换成字符串再进行查找。

    def get(self, key, default=None):
        try:
            return self[key]  # get 方法把查找工作用 self[key] 的形式委托给 __getitem__，这样在宣布查找失败之前，还能通过 __missing__ 再给某个键一个机会。
        except KeyError:
            return default  # 如果抛出 KeyError，那么说明 __missing__ 也失败了，于是返回 default。

    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()  # 先按照传入键的原本的值来查找（我们的映射类型中可能含有非字符串的键），如果没找到，再用 str() 方法把键转换成字符串再查找一次。

# 调用如下：
>>> d = StrKeyDict0([('2', 'two'), ('4', 'four')])
>>> d['2']
'two'
>>> d[4]
'four'
>>> d[1]
Traceback (most recent call last):
...
KeyError: '1'
>>> d.get(1, 'N/A')
'N/A'
>>> 2 in d
True
>>> 1 in d
False
```

**字典的其他类型**

- **collections.OrderedDict** <span id="OrderedDict"></span>
  - 这个类型在添加键的时候会保持顺序，因此键的迭代次序总是一致的。`OrderedDict` 的 `popitem` 方法默认删除并返回的是字典里的最后一个元素，但是如果像 `my_odict.popitem(last=False)` 这样调用它，那么它删除并返回第一个被添加进去的元素。
- **collections.ChainMap** <span id="ChainMap"></span>
  - 该类型可以容纳数个不同的映射对象，然后在进行键查找操作的时候，这些对象会被当作一个整体被逐个查找，直到键被找到为止。[ChainMap](http://python.usyiyi.cn/documents/python_352/library/collections.html#collections.ChainMap)【在给有嵌套作用域的语言做解释器的时候很有用，可以用一个映射对象来代表一个作用域的上下文。】
  - 如： `import builtins`,`pylookup = ChainMap(locals(), globals(), vars(builtins))`
- **collections.Counter** <span id="Counter"></span>
  - 这个映射类型会给键准备一个整数计数器。每次更新一个键的时候都会增加这个计数器。
  - 参考：[counter](http://python.usyiyi.cn/documents/python_352/library/collections.html#collections.Counter)
- **colllections.UserDict** <span id="UserDict"></span>
  - 这个类其实就是把标准 dict 用纯 Python 又实现了一遍。
  - 跟其他的类型不同，UserDict 是让用户继承写子类的。
  - 参考:[UserDict](http://python.usyiyi.cn/documents/python_352/library/collections.html#collections.UserDict)
  - 注意：UserDict 并不是 dict 的子类，但是 UserDict 有一个叫作 data 的属性，是 dict 的实例，这个属性实际上是 UserDict 最终存储数据的地方。
- **将来可能的 collections.TransformDict 类型**
  - <https://www.python.org/dev/peps/pep-0455/>

**不可变的映射类型**

- **types.MappingProxyType** 【只在 Python 3.3 以及更新的版本里才有】 <span id="MappingProxyType"></span>
  - 如果给这个类一个映射，它会返回一个只读的映射视图。虽然是个只读视图，但是它是动态的。这意味着如果对原映射做出了改动，我们通过这个视图可以观察到，但是无法通过这个视图对原映射做出修改。

```python
>>> from types import MappingProxyType
>>> d = {1:'A'}
>>> d_proxy = MappingProxyType(d)
>>> d_proxy
mappingproxy({1: 'A'})
>>> d_proxy[1]  # d 中的内容可以通过 d_proxy 看到
'A'
>>> d_proxy[2] = 'x'  # 但是通过 d_proxy 并不能做任何修改。
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'mappingproxy' object does not support item assignment
>>> d[2] = 'B'
>>> d_proxy  # d_proxy 是动态的，也就是说对 d 所做的任何改动都会反馈到它上面。
mappingproxy({1: 'A', 2: 'B'})
>>> d_proxy[2]
'B'
```

<span id="集合的定义形式"></span>

**集合**

集合的本质是许多唯一对象的聚集。因此，集合可以用于去重：

```python
>>> l = ['spam', 'spam', 'eggs', 'spam']
>>> set(l)
{'eggs', 'spam'}
>>> list(set(l))
['eggs', 'spam']
```

集合中的元素必须是可散列的，`set` 类型本身是不可散列的，但是 `frozenset` 可以。因此可以创建一个包含不同 `frozenset` 的 `set`。

- 集合句法陷阱：
  - 空集合定义：`s = set()` 而不是 `s={}` 【这是字典】
  - 非空集合可以这样定义： `s = {1,2,3,4}`
  - 没有针对 `frozenset` 的特殊字面量句法，只能采用构造方法：`frozenset(range(10))`
- 集合推导：
  - `>>> {chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i),'')}`
- 集合的操作：
  - 交集、并集、差集等等

集合的UML图：

![py_set_uml]({% link assets/programingimg/py_set_uml.png %})

集合的方法：

**方法名** | **set** | **frozenset** | **说明**
-----------|---------|---------------|-------
`s.add(e)` | • |  | 把元素 `e` 添加到 `s` 中
`s.pop()` | • |  | 从 `s` 中移除一个元素并返回它的值，若 `s` 为空，则抛出 `KeyError` 异常
`s.remove(e)` | `•` |  | 从 s 中移除 `e` 元素，若 `e` 元素不存在，则抛出 `KeyError` 异常
`s.clear()` | • |  | 移除掉 `s` 中的所有元素
`s.copy()` | • | • | 对 `s` 浅复制
`s.discard(e)` | • |  | 如果 `s` 里有 `e` 这个元素的话，把它移除
`s.__iter__()` | • | • | 返回 `s` 的迭代器
`s.__len__()` | • | • | `len(s)`


<span id="散列的实现"></span>

**dict和set的背后**

Python 源码 [dictobject.c 模块](http://hg.python.org/cpython/file/tip/Objects/dictobject.c)里有丰富的注释。

散列表其实是一个稀疏数组（总是有空白元素的数组称为稀疏数组）。在一般的数据结构教材中，散列表里的单元通常叫作表元（bucket）。
在 dict 的散列表当中，每个键值对都占用一个表元，每个表元都有两个部分，一个是对键的引用，另一个是对值的引用。
因为所有表元的大小一致，所以可以通过偏移量来读取某个表元。

- 散列值和相等性
  - 如果两个对象在比较的时候是相等的，那它们的散列值必须相等，否则散列表就不能正常运行了。
    - 例如，如果 1 == 1.0 为真，那么 hash(1) == hash(1.0) 也必须为真，但其实这两个数字（整型和浮点）的内部结构是完全不一样的。
- 散列表算法
  - 如：`my_dict[search_key] ` , python 首先利用 `hash(key)` 函数计算 **散列值**。把这个值得最低几位作为偏移量，在散列表里查找表元。
    - 若找到的表元是空的，则抛出 `KeyError` 异常。
    - 若不是空的，则表元里会有一对 `found_key:found_value`。这时候 Python 会检验 `search_key == found_key` 是否为真，如果它们相等的话，就会返回 `found_value`。
    - 如果 `search_key` 和 `found_key` 不匹配的话，这种情况称为 **散列冲突**。
      - 发生这种情况是因为，散列表所做的其实是把随机的元素映射到只有几位的数字上，而散列表本身的索引又只依赖于这个数字的一部分。为了解决散列冲突，算法会在散列值中另外再取几位，然后用特殊的方法处理一下，把新得到的数字再当作索引来寻找表元。 若这次找到的表元是空的，则同样抛出 `KeyError` ；若非空，或者键匹配，则返回这个值；或者又发现了散列冲突，则重复以上的步骤。
    - 添加新元素和更新现有键值的操作几乎跟上面一样。只不过对于前者，在发现空表元的时候会放入一个新元素；对于后者，在找到相对应的表元后，原表里的值对象会被替换成新值。
    - 另外在插入新值时，Python 可能会按照散列表的拥挤程度来决定是否要重新分配内存为它扩容。如果增加了散列表的大小，那散列值所占的位数和用作索引的位数都会随之增加，这样做的目的是为了减少发生散列冲突的概率。
    - 表面上看，这个算法似乎很费事，而实际上就算 `dict` 里有数百万个元素，多数的搜索过程中并不会有冲突发生，平均下来每次搜索可能会有一到两次冲突。在正常情况下，就算是最不走运的键所遇到的冲突的次数用一只手也能数过来。
    - 其他：
      - CPython 的实现细节里有一条是：如果有一个整型对象，而且它能被存进一个机器字中，那么它的散列值就是它本身的值。
      - 在散列冲突的情况下，用 C 语言写的用来打乱散列值位的算法的名字很有意思，叫 `perturb`。详见 CPython 源码里的 [dictobject.c](https://hg.python.org/cpython/file/tip/Objects/dictobject.c)。

![dict_hash_search]({% link assets/programingimg/dict_hash_search.png %})


<span id="字典的影响"></span>

**dict的实现及其导致的结果**

- **键必须是可散列的**,对象有以下要求：
  - 支持 `hash()` 函数，并且通过 `__hash__()` 方法所得到的散列值是不变的。
  - 支持通过 `__eq__()` 方法来检测相等性。
  - 若` a == b` 为真，则 `hash(a) == hash(b)` 也为真。
  - **注意：**
    - 所有由用户自定义的对象默认都是可散列的，因为它们的散列值由 `id()` 来获取，而且它们都是不相等的。
    - 如果实现了一个类的 `__eq__` 方法，并且希望它是可散列的，那么它一定要有个恰当的 `__hash__` 方法，保证在 `a == b` 为真的情况下 `hash(a) == hash(b)` 也必定为真。否则就会破坏恒定的散列表算法，导致由这些对象所组成的字典和集合完全失去可靠性，这个后果是非常可怕的。另一方面，如果一个含有自定义的 `__eq__` 依赖的类处于可变的状态，那就不要在这个类中实现 `__hash__` 方法，因为它的实例是不可散列的。
- **字典在内存上的开销巨大**
  - 由于字典使用了散列表，而散列表又必须是稀疏的，这导致它在空间上的效率低下。
  - 如果你需要存放数量巨大的记录，那么放在由`元组`或是`具名元组`构成的列表中会是比较好的选择；最好不要根据 JSON 的风格，用由字典组成的列表来存放这些记录。用元组取代字典就能节省空间的原因有两个：其一是避免了散列表所耗费的空间，其二是无需把记录中字段的名字在每个元素里都存一遍。【比如从数据库中查询出来的大量的数据】
  - 在用户自定义的类型中，`__slots__` 属性可以改变实例属性的存储方式，由 `dict` 变成 `tuple`
  - 现在讨论的是空间优化。如果你手头有几百万个对象，而机器有几个 GB 的内存，那么空间的优化工作可以等到真正需要的时候再开始计划，因为 **优化往往是可维护性的对立面** 。
- **键查询很快**
  - `dict` 的实现是典型的空间换时间：字典类型有着巨大的内存开销，但它们提供了无视数据量大小的快速访问——只要字典能被装在内存里。
- **键的次序取决于添加顺序**
  - 当往 `dict` 里添加新键而又发生散列冲突的时候，新键可能会被安排存放到另一个位置。于是下面这种情况就会发生：由 `dict([key1, value1), (key2, value2)]` 和 `dict([key2, value2], [key1, value1])` 得到的两个字典，在进行比较的时候，它们是相等的；但是如果在 `key1` 和 `key2` 被添加到字典里的过程中有冲突发生的话，这两个键出现在字典里的顺序是不一样的。
- **往字典里添加新键可能会改变已有键的顺序**
  - 无论何时往字典里添加新的键，Python 解释器都可能做出为字典扩容的决定。扩容导致的结果就是要新建一个更大的散列表，并把字典里已有的元素添加到新表里。这个过程中可能会发生新的散列冲突，导致新散列表中键的次序变化。如： 在迭代一个字典的所有键的过程中同时对字典进行修改，那么这个循环很有可能会跳过一些键——甚至是跳过那些字典中已经有的键。
  - 不要对字典同时进行迭代和修改。如果想扫描并修改一个字典，最好分成两步来进行：首先对字典迭代，以得出需要添加的内容，把这些内容放在一个新字典里；迭代结束之后再对原有字典进行更新。
  - **注意：** 在 Python 3 中，.keys()、.items() 和 .values() 方法返回的都是字典视图。也就是说，这些方法返回的值更像集合，而不是像 Python 2 那样返回列表。视图还有动态的特性，它们可以实时反馈字典的变化。

<span id="集合的影响"></span>

**set的实现以及导致的结果**

`set` 和 `frozenset` 的实现也依赖散列表，但在它们的散列表里存放的只有元素的引用（就像在字典里只存放键而没有相应的值）,在 `set` 加入到 `Python` 之前，我们都是把字典加上无意义的值当作集合来用的。

- **集合里的元素必须是可散列的。**
- **集合很消耗内存。**
- **可以很高效地判断元素是否存在于某个集合。**
- **元素的次序取决于被添加到集合里的次序。**
- **往集合里添加元素，可能会改变集合里已有元素的次序。**

<span id="代码片段"></span>

**代码片段**

```python
# 将string转换为2进制
def bits1(st):
    return ''.join(format(ord(x), 'b') for x in st)

# 将int转换为2进制
def bits(it):
    return format(it, 'b')

# 比较散列值二进制之间的差异
In [12]: k1 = bits(hash('Monty'))
In [14]: k2 = bits(hash('Money'))
In [16]: diff = ('^ '[a==b] for a,b in zip(k1,k2))
In [17]: print(k1);print(k2);print(''.join(diff))
10111010111001101110011110000101110011010010010100110001001101
-1110011110101001000110101110100001010001011101100010010111100
^^  ^  ^  ^^  ^  ^^ ^ ^ ^^^^   ^^^^  ^ ^^  ^^^^   ^   ^^^^   ^

```
