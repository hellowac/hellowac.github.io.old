---
nav: blog
layout: post
title: "流畅的python - 从协议到抽象基类"
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
- [抽象基类](#抽象基类)


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

<span id="抽象基类"></span>

### 抽象基类

`cls` 的元类是 `abc.ABCMeta`，就可以使用 `isinstance(obj, cls)`, `cls` 就是抽象基类。

[`collections.abc` 中有很多有用的抽象类（Python 标准库的 `numbers` 模块中还有一些）。]

**优点：** 可以使用 `register` 类方法在终端用户的代码中把某个类“声明”为一个抽象基类的“虚拟”子类. 这大大地打破了严格的强耦合，与面向对象编程人员掌握的知识有很大出入.

```python
# 抽象基类的本质就是几个特殊方法

>>> class Struggle:
...     def __len__(self): return 23
...
>>> from collections import abc
# （要使用正确的句法和语义实现，前者要求没有参数，后者要求返回一个非负整数，指明对象的长度；如果不使用规定的句法和语义实现特殊方法，如 __len__，会导致非常严重的问题）
>>> isinstance(Struggle(), abc.Sized)  # 无需注册，abc.Sized 也能把 Struggle 识别为自己的子类，只要实现了特殊方法 __len__ 即可
True
```

如果实现的类体现了 `numbers` 、 `collections.abc` 或其他框架中抽象基类的概念，要么继承相应的抽象基类（必要时），要么把类注册到相应的抽象基类中。
开始开发程序时，不要使用提供注册功能的库或框架，要自己动手注册；如果必须检查参数的类型（这是最常见的），例如检查是不是“序列”，那就这样做：

```python
isinstance(the_arg, collections.abc.Sequence)
```

**不要在生产代码中定义抽象基类（或元类）**

```
当然，你还可以自己定义抽象基类，但是我不建议高级 Python 程序员之外的人这么做；
同样，我也不建议你自己定义元类……我说的“高级 Python 程序员”是指对 Python 语言的一招一式都了如指掌，
即便对这类人来说，抽象基类和元类也不是常用工具。如此“深层次的元编程”，
如果可以这么讲的话，适合框架的作者使用，这样便于众多不同的开发团队独立扩展框架……真正需要这么做的“高级 Python 程序员”不超过 1%。
    ——Alex Martelli
```

**兼容性问题：**
  - 在 Python 3.4 中没有能把字符串和元组或其他不可变序列区分开的抽象基类，因此必须测试 `str`。
  - 在 Python 2 中，`basestr` 类型可以协助这样的测试。`basestr` 不是抽象基类，但它是 `str` 和 `unicode` 的超类；
  - 然而，Python 3 把 `basestr` 去掉了。奇怪的是，Python 3 中有个 `collections.abc.ByteString` 类型，但是它只能检测 `bytes` 和 `bytearray` 类型。

`抽象基类`是用于封装框架引入的一般性概念和抽象的，例如“一个序列”和“一个确切的数”。（读者）基本上不需要自己编写新的抽象基类，只要正确使用现有的抽象基类，就能获得 99.9% 的好处，而不用冒着设计不当导致的巨大风险。
