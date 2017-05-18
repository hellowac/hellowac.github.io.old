---
nav: blog
layout: post
title: "流畅的python - 函数装饰器和闭包"
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

- [闭包](#闭包)
- [装饰器示例](#示例)
- [标准库中的装饰器](#标准库中的装饰器)

- **基础知识**
  - 装饰器是可调用的对象，其参数是另一个函数（被装饰的函数）。装饰器可能会处理被装饰的函数，然后把它返回，或者将其替换成另一个函数或可调用对象。
  - 特性：
    - 能把被装饰的函数替换成其他函数
    - 装饰器在加载模块时立即执行
  - 装饰器执行时间：
    - 在 **导入时** ， 装饰器在被装饰的函数定义之后立即运行。
    - **注意：** 装饰器在导入模块时立即执行，而被装饰的函数只在明确调用时运行。 【导入时和运行时之间的区别】
  - 其他：
    - 装饰器通常在一个模块中定义，然后应用到其他模块中的函数上。
    - 大多数装饰器会在内部定义一个函数，然后将其返回。

使用装饰器改善 [策略模式]({% link _posts/proraming/2017-05-17-fluentpython06.md %})：

```python
promos = []  # 初始化注册列表

def promotion(promo_func):  # 用于注册 策略 的装饰器定义
    promos.append(promo_func)
    return promo_func

@promotion  # 注册一个 策略
def fidelity(order):
    """为积分为1000或以上的顾客提供5%折扣"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

@promotion
def bulk_item(order):
    """单个商品为20个或以上时提供10%折扣"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

@promotion
def large_order(order):
    """订单中的不同商品达到10个或以上时提供7%折扣"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0

def best_promo(order):  # 遍历已注册的策略列表，选择最佳策略
    """选择可用的最佳折扣
    """
    return max(promo(order) for promo in promos)
```

- **变量作用域和闭包以及nonlocal声明：** <span id="闭包"></span>
  - Python 编译函数的定义体时，只要变量赋值了，便认为是局部变量[本地变量]，除非使用关键字（`global`、`nonlocal`）声明为 全局变量 或 自由变量。
  - **自由变量：** 函数定义体中引用、但是不在定义体中定义的非全局变量。
  - **闭包：** 闭包是一种函数，它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍能使用那些绑定。
  - 自由变量 绑定在 返回的内部函数 的 `__closure__` 属性中。 `__closure__` 中的各个元素对应于 内部函数的 `__code__.co_freevars` 属性 中的一个名称。这些元素是 `cell` 对象，有个 `cell_contents` 属性，保存着真正的值。
  - **nonlocal声明：** 把变量标记为自由变量，即使在函数中为变量赋予新值了，也会变成自由变量（而不是局部变量）。如果为 `nonlocal` 声明的变量赋予新值，闭包中保存的绑定会更新。【python3新增 `nonlocal` 声明，python2 中 可参考：[Access to Names in Outer Scopes](https://www.python.org/dev/peps/pep-3104/)】

```python
# 用闭包实现一个 计算平均数 函数.
def make_averager():
    series = []

    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total/len(series)
    return averager
# 应用
In [23]: avg = make_averager()

In [24]: avg(10)
Out[24]: 10.0

In [25]: avg(12)
Out[25]: 11.0

In [26]: avg(14)
Out[26]: 12.0

# 审查
In [27]: avg.__code__.co_varnames  # 局部变量
Out[27]: ('new_value', 'total')

In [28]: avg.__code__.co_freevars  # 自由变量，但这里不适用 nonlocal 声明，一旦定义时赋值，便会变成 局部变量.
Out[28]: ('series',)

In [29]: avg.__closure__  # 闭包
Out[29]: (<cell at 0x1107aaeb8: list object at 0x1109b65c8>,)

In [30]: avg.__closure__[0].cell_contents  # 闭包中 变量 的实际值.
Out[30]: [10, 12, 14]

# 使用 nonlocal 声明
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        nonlocal count, total
        count += 1  # 等于 count = count + 1
        total += new_value  # 等于 total = total + new_value
        return total / count

    return averager

# 应用
In [32]: avg = make_averager()

In [33]: avg.__code__.co_freevars  # 自由变量.
Out[33]: ('count', 'total')

In [34]: avg.__code__.co_varnames  # 本地变量.
Out[34]: ('new_value',)

In [35]: avg.__closure__[0].cell_contents  # 初始的 count 值.
Out[35]: 0

In [36]: avg.__closure__[1].cell_contents  # 初始的 total 值.
Out[36]: 0

In [37]: avg(3)
Out[37]: 3.0

In [38]: avg.__closure__[0].cell_contents  # count 当前值为 1
Out[38]: 1

In [39]: avg.__closure__[1].cell_contents  # total 当前值为 3
Out[39]: 3
```

- **实现一个函数调用计时装饰器** <span id="示例"></span>

```python
import time

def clock(func):
    def clocked(*args):  # 接受被装饰的函数的参数.
        t0 = time.perf_counter()
        result = func(*args) # 调用被装饰的函数. 并且 func 为 自由变量. 指向被装饰的函数.
        elapsed = time.perf_counter() - t0  # 调用花费时间.
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
        return result
    return clocked  # 返回装饰器.

@clock
def snooze(seconds):
    time.sleep(seconds)

# 等价于: snooze = clock(snooze), @ 符号就是语法糖

@clock
def factorial(n):  # 菲波那切数列
    return 1 if n < 2 else n*factorial(n-1)

# 等价于: factorial = clock(factorial), @ 符号就是语法糖

if __name__=='__main__':
    print('*' * 40, 'Calling snooze(.123)')
    snooze(.123)
    print('*' * 40, 'Calling factorial(6)')
    print('6! =', factorial(6))

# 应用：
**************************************** Calling snooze(.123)
[0.12761933s] snooze(0.123) -> None
**************************************** Calling factorial(6)
[0.00000072s] factorial(1) -> 1
[0.00002145s] factorial(2) -> 2
[0.00005648s] factorial(3) -> 6
[0.00007194s] factorial(4) -> 24
[0.00008577s] factorial(5) -> 120
[0.00010014s] factorial(6) -> 720
6! = 720
```

为了便于调试. 需 使用 标准库中的 `functools.wraps` 装饰器， 将 被装饰的函数 的属性复制到返回的 装饰器函数 中. 并且还能正确处理关键字参数.

```python
# 优化后的装饰器定义：
import time
import functools

def clock(func):
    @functools.wraps(func)  # 将被装饰的 func 函数中的属性，复制到 返回的 clocked 装饰器函数 中.
    def clocked(*args, **kwargs):
        t0 = time.time()
        result = func(*args, **kwargs)  # 调用 原函数, 以及 他 的参数.
        elapsed = time.time() - t0
        name = func.__name__
        arg_lst = []
        if args:
            arg_lst.append(', '.join(repr(arg) for arg in args))
        if kwargs:
            pairs = ['%s=%r' % (k, w) for k, w in sorted(kwargs.items())]
            arg_lst.append(', '.join(pairs))
        arg_str = ', '.join(arg_lst)
        print('[%0.8fs] %s(%s) -> %r ' % (elapsed, name, arg_str, result))
        return result
    return clocked
```

- **标准库中的其他装饰器** <span id="标准库中的装饰器"></span>
  - `functools.property` : 装饰成属性.
  - `functools.classmethod` : 装饰成类方法.
  - `functools.staticmethod` : 装饰成静态方法.
  - `functools.lru_cache` : 做备忘.把耗时的函数的结果保存起来，避免传入相同的参数时重复计算.
    - lru: 为 'Least Recently Used' 的 缩写, 表明缓存不会无限制增长，一段时间不用的缓存条目会被扔掉。
    - [参数](http://python.usyiyi.cn/documents/python_352/library/functools.html#functools.lru_cache):
      - **maxsize** : 默认值 `128`, 指定存储多少个调用的结果。缓存满了之后，旧的结果会被扔掉，腾出空间。为了得到最佳性能，`maxsize` 应该设为 `2` 的幂。
      - **typed** : 默认值 `True`, 把不同参数类型得到的结果分开保存，即把通常认为相等的浮点数和整数参数（如 1 和 1.0）区分开。因为 `lru_cache` 使用字典存储结果，而且键根据调用时传入的定位参数和关键字参数创建，所以被 `lru_cache` 装饰的函数，它的所有参数都必须是 `可散列` 的。
  - `functools.singledispatch` : 【3.4新增】将普通函数会变成泛函数（generic function）：根据第一个参数的类型，以不同方式执行相同操作的一组函数。【单分派】【多分派：根据多个参数选择专门的函数】

```python
from clockdeco import clock

@clock
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

if __name__=='__main__':
    print(fibonacci(6))

# 执行:
In [58]: print(fibonacci(6))
[0.00000055s] fibonacci(0) -> 0  # 装饰器输出
[0.00000063s] fibonacci(1) -> 1
[0.00004063s] fibonacci(2) -> 1
[0.00000079s] fibonacci(1) -> 1
[0.00000043s] fibonacci(0) -> 0
[0.00000038s] fibonacci(1) -> 1
[0.00001551s] fibonacci(2) -> 1
[0.00004446s] fibonacci(3) -> 2
[0.00012143s] fibonacci(4) -> 3
[0.00000035s] fibonacci(1) -> 1
[0.00000035s] fibonacci(0) -> 0
[0.00000039s] fibonacci(1) -> 1
[0.00001379s] fibonacci(2) -> 1
[0.00002869s] fibonacci(3) -> 2
[0.00000042s] fibonacci(0) -> 0
[0.00000040s] fibonacci(1) -> 1
[0.00011024s] fibonacci(2) -> 1
[0.00000052s] fibonacci(1) -> 1
[0.00000054s] fibonacci(0) -> 0
[0.00000038s] fibonacci(1) -> 1
[0.00001335s] fibonacci(2) -> 1
[0.00002877s] fibonacci(3) -> 2
[0.00017100s] fibonacci(4) -> 3
[0.00021284s] fibonacci(5) -> 5
[0.00034903s] fibonacci(6) -> 8
8  # 被装饰的函数返回.

# 应用 functools.lru_cache 缓存结果.
@functools.lru_cache()
@clock
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-2) + fibonacci(n-1)

# 执行：
In [61]: print(fibonacci(8))
[0.00000058s] fibonacci(0) -> 0  # 装饰器输出.
[0.00000059s] fibonacci(1) -> 1
[0.00003720s] fibonacci(2) -> 1
[0.00000106s] fibonacci(3) -> 2
[0.00005661s] fibonacci(4) -> 3
[0.00000080s] fibonacci(5) -> 5
[0.00007197s] fibonacci(6) -> 8
[0.00000147s] fibonacci(7) -> 13
[0.00009799s] fibonacci(8) -> 21 # 可见执行时间缩短很大
21

# 应用 functools.singledispatch 装饰器

# 一个web调试函数，根据类型，生成不同的HTML

# 规则如下：
# str：把内部的换行符替换为 '<br>\n'；不使用 <pre>，而是使用 <p>。
# int：以十进制和十六进制显示数字。
# list：输出一个 HTML 列表，根据各个元素的类型进行格式化。

# 效果如下：
# >>> htmlize({1, 2, 3})  # 默认情况下，在 <pre></pre> 中显示 HTML 转义后的对象字符串表示形式。
# '<pre>{1, 2, 3}</pre>'
# >>> htmlize(abs)
# '<pre><built-in function abs></pre>'
# >>> htmlize('Heimlich & Co.\n- a game')  # 为 str 对象显示的也是 HTML 转义后的字符串表示形式，不过放在 <p></p> 中，而且使用 <br> 表示换行。
# '<p>Heimlich & Co.<br>\n- a game</p>'
# >>> htmlize(42)  # int 显示为十进制和十六进制两种形式，放在 <pre></pre> 中。
# '<pre>42 (0x2a)</pre>'
# >>> print(htmlize(['alpha', 66, {3, 2, 1}]))  # 各个列表项目根据各自的类型格式化，整个列表则渲染成 HTML 列表。
# <ul>
# <li><p>alpha</p></li>
# <li><pre>66 (0x42)</pre></li>
# <li><pre>{1, 2, 3}</pre></li>
# </ul>

from functools import singledispatch
from collections import abc
import numbers
import html

# singledispatch 创建一个自定义的 htmlize.register 装饰器，把多个函数绑在一起组成一个泛函数

@singledispatch  # 标记处理 object 类型的基函数
def htmlize(obj):
    content = html.escape(repr(obj))
    return '<pre>{}</pre>'.format(content)

@htmlize.register(str)  # 使用 @«base_function».register(«type») 装饰。
def _(text):            # 专门函数的名称无关紧要；_ 是个不错的选择，简单明了。
    content = html.escape(text).replace('\n', '<br>\n')
    return '<p>{0}</p>'.format(content)

@htmlize.register(numbers.Integral)  # 为每个需要特殊处理的类型注册一个函数。numbers.Integral 是 int 的虚拟超类。
def _(n):
    return '<pre>{0} (0x{0:x})</pre>'.format(n)

@htmlize.register(tuple)  # 叠放多个 register 装饰器，让同一个函数支持不同类型。
@htmlize.register(abc.MutableSequence)
def _(seq):
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
    return '<ul>\n<li>' + inner + '</li>\n</ul>'
```

最佳实践： 只要可能，注册的专门函数应该处理抽象基类（如 `numbers.Integral` 和 `abc.MutableSequence`），不要处理具体实现（如 `int` 和 `list`）。这样，代码支持的兼容类型更广泛。例如，Python 扩展可以子类化 `numbers.Integral`，使用固定的位数实现 `int` 类型。

更多关于 `singledispatch` 的特性， 参考： [PEP 443 — Single-dispatch generic functions](https://www.python.org/dev/peps/pep-0443/)

**优点：** 支持模块化扩展：各个模块可以为它支持的各个类型注册一个专门函数。

**缺陷：** 让代码单元（类或函数）承担的职责太多。

- **叠放装饰器：** 装饰器是函数，因此可以组合起来使用

```python
@d1
@d2
def f():
    print('f')

# 相当于：

def f():
    print('f')

f = d1(d2(f))
```

- **参数化装饰器:**
  - 创建一个装饰器工厂函数，把参数传给它，返回一个装饰器，然后再把它应用到要装饰的函数上。

```python
registry = set()  # 设为set()对象，这样添加和删除函数的速度更快。
def register(active=True):  # 接受一个可选的关键字参数。
    def decorate(func):  # 内部函数是真正的装饰器，参数是一个函数。
        print('running register(active=%s)->decorate(%s)'
              % (active, func))
        if active:   # 是 True 时才注册 func。【闭包】
            registry.add(func)
        else:
            registry.discard(func)  # 如果 active 不为真，而且 func 在 registry 中，那么把它删除。

        return func  # decorate 是装饰器，必须返回一个函数。
    return decorate  # register 是装饰器工厂函数，因此返回 decorate。

@register(active=False)  # 工厂函数必须作为函数调用，并且传入所需的参数。
def f1():
    print('running f1()')

@register()  # 即使不传入参数，必须作为函数调用（@register()），即要返回真正的装饰器 decorate。
def f2():
    print('running f2()')

def f3():
    print('running f3()')

# 相当于：
f1 = register(active=False)(f1)
f2 = register()(f2) # active 默认值 True

# 参数化 clock 装饰器

import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFAULT_FMT):   # 参数化装饰器工厂函数
    def decorate(func):       # 真正的装饰器。
        def clocked(*_args):  # 包装被装饰的函数。
            t0 = time.time()
            _result = func(*_args)  # 被装饰的函数返回的真正结果。
            elapsed = time.time() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)  # 显示被装饰函数参数.
            result = repr(_result)  # 字符串表示形式
            print(fmt.format(**locals()))  # 引用 clocked 的局部变量。
            return _result  # 返回被装饰的函数返回的值
        return clocked  # 返回包装过函数后的装饰器内部函数。
    return decorate  # 返回装饰器.

if __name__ == '__main__':

@clock()
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)

# 执行：
[0.12336302s] snooze(0.123) -> None
[0.12546515s] snooze(0.123) -> None
[0.12632775s] snooze(0.123) -> None

# 应用另一种方式:
@clock('{name}: {elapsed}s')
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)

# 执行:
snooze: 0.12692809104919434s
snooze: 0.12740516662597656s
snooze: 0.12456703186035156s

# 应用另一种方式：
@clock('{name}({args}) dt={elapsed:0.3f}s')
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)

# 执行：
snooze(0.123) dt=0.124s
snooze(0.123) dt=0.124s
snooze(0.123) dt=0.124s
```

[Graham Dumpleton](https://github.com/GrahamDumpleton) 和 [Lennart Regebro](https://github.com/regebro) 认为，装饰器最好通过实现 `__call__` 方法的类实现，不应该像本章的示例那样通过函数实现。 【这个方式实现非平凡的装饰器更好】

更多装饰器参考： [Graham Dumpleton](https://github.com/GrahamDumpleton)的[wrapt](https://github.com/GrahamDumpleton/wrapt) 模块， 【作用是简化装饰器和动态函数包装器的实现，即使多层装饰也支持内省，而且行为正确，既可以应用到方法上，也可以作为描述符使用。】

- **扩展阅读**
  - [博客文章](https://github.com/GrahamDumpleton/wrapt/blob/develop/blog/README.md) 。 深入剖析了如何实现行为良好的装饰器。
  - [decorator](https://github.com/micheles/decorator), 旨在“简化普通程序员使用装饰器的方式，并且通过各种复杂的示例推广装饰器”。可通过 [PyPI](https://pypi.python.org/pypi/decorator) 安装。
  - [Python Decorator Library](https://wiki.python.org/moin/PythonDecoratorLibrary)维基页面, 在 Python 刚添加装饰器这个特性时就创建了，里面有很多示例。由于那个页面是几年前开始编写的，有些技术已经过时了，不过仍是很棒的灵感来源。
  - [PEP 443-Single-dispatch generic functions](http://www.python.org/dev/peps/pep-0443/), 对单分派泛函数的基本原理和细节做了说明。
  - [Five-Minute Multimethods in Python](http://www.artima.com/weblogs/viewpost.jsp?thread=101605), Guido 所作，详细说明了如何使用装饰器实现泛函数（也叫多方法）。他给出的代码支持多分派（即根据多个定位参数进行分派）。Guido 写的多方法代码很棒，但那只是教学示例。如果想使用现代的技术实现多分派泛函数，并支持在生产环境中使用，可以用 Martijn Faassen 开发的 [Reg](http://reg.readthedocs.io/en/latest/) 。Martijn 还是模型驱动型 REST 式 Web 框架 [Morepath](http://morepath.readthedocs.org/en/latest/)的开发者。
  - [Closures in Python](http://effbot.org/zone/closure.htm), 解说了闭包这个术语。
  - [PEP 3104—Access to Names in Outer Scopes](http://www.python.org/dev/peps/pep-3104/), 说明了引入 `nonlocal` 声明的原因：重新绑定既不在本地作用域中也不在全局作用域中的名称。这份 PEP 还概述了其他动态语言（Perl、Ruby、JavaScript，等等）解决这个问题的方式，以及 Python 中可用设计方案的优缺点。
  - [PEP 227—Statically Nested Scopes](http://www.python.org/dev/peps/pep-0227/), 说明了 Python 2.1 引入的词法作用域。词法作用域在这一版里是一种方案，到 Python 2.2 就变成了标准。此外，这份 PEP 还说明了 Python 中闭包的基本原理和实现方式的选择。
