---
nav: blog
layout: post
title: "流畅的python - 上下文管理和else语句"
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

- [else语句](#else语句)
- [with上下文管理](#with上下文管理)
- [contextlib工具](#contextlib工具)
- [扩展阅读](#扩展阅读)

<span id="else语句"></span>

## else语句

`else` 子句不仅能在 `if` 语句中使用，还能在 `for` 、 `while` 和 `try` 语句中使用。

- 在 `for` 语句中:
  - 仅当 `for` 循环运行完毕时（即 `for` 循环没有被 `break` 语句中止）才运行 `else` 块。
- 在 while 语句中:
  - 仅当 `while` 循环因为条件为假值而退出时（即 `while` 循环没有被 `break` 语句中止）才运行 `else` 块。
- 在 `try` 语句中:
  - 仅当 `try` 块中没有异常抛出时才运行 `else` 块。官方文档还指出：“ `else` 子句抛出的异常不会由前面的 `except` 子句处理。”

**在所有情况下，如果异常或者 `return` 、 `break` 或 `continue` 语句导致控制权跳到了复合语句的主块之外，else 子句也会被跳过。**

```python
for item in my_list:
    if item.flavor == 'banana':
            break
else:
    raise ValueError('No banana flavor found!')  # 仅当 for循环不被break终端时才抛出异常.

try:
    dangerous_call()
except OSError:
    log('OSError...')
else:
    after_call()  # 仅当 不抛出异常时才执行. 这里抛出的异常不会被 except 捕捉。
```

<span id="with上下文管理"></span>

## with上下文管理

上下文管理器对象存在的目的是管理 `with` 语句，就像迭代器的存在是为了管理 `for` 语句一样。

上下文管理器协议包含 `__enter__` 和 `__exit__` 两个方法。`with` 语句开始运行时，会在上下文管理器对象上调用 `__enter__` 方法。`with` 语句运行结束后，会在上下文管理器对象上调用 `__exit__` 方法，以此扮演 `finally` 子句的角色。

如： 文件对象的上下文管理.

```python
# open() 函数返回 TextIOWrapper 类的实例
>>> with open('mirror.py') as fp:  # fp 绑定到打开的文件上，因为 __enter__ 返回的是 self.
...     src = fp.read(60)  # 从文件对象出去对象.
...
>>> fp  # fp变量仍然可用 【with语句并没有定义新作用域】
<_io.TextIOWrapper name='mirror.py' mode='r' encoding='UTF-8'>
>>> fp.closed, fp.encoding  # 可以读取文件的属性。
(True, 'UTF-8')
>>> fp.read(60)  # 不能在 fp 上执行 I/O 操作，因为在 with 块的末尾，调用 TextIOWrapper.__exit__ 方法把文件关闭了。
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: I/O operation on closed file.
```

`LookingGlass` 上下文管理器类的代码：

```python
class LookingGlass:

    def __enter__(self):  # 调用时除了self，不传入其他参数
        import sys
        self.original_write = sys.stdout.write  #  把原来方法保存到一个事例属性中，以供后面使用
        sys.stdout.write = self.reverse_write  #  打猴子补丁，替换成自己的方法。
        return 'JABBERWOCKY'  # 返回字符串，存入 with语句后面的 as 变量中。

    def reverse_write(self, text):  # 取代标准方法，将文本逆转输出.
        self.original_write(text[::-1])

    def __exit__(self, exc_type, exc_value, traceback):  # 一切正常传入 None,None,None, 否则为异常数据.
        import sys  # 重复导入模块不会消耗很多资源，因为 Python 会缓存导入的模块。
        sys.stdout.write = self.original_write  # 还原 sys.stdout.write 方法.
        if exc_type is ZeroDivisionError:  # 捕捉异常.
            print('Please DO NOT divide by zero!')
            return True  # 返回 True，告诉解释器，异常已经处理了。
        # 返回 None，或者 True 之外的值，with 块中的任何异常都会向上冒泡。
```

传给 `__exit__` 方法的三个参数列举如下:

- `exc_type` : 异常类（例如 `ZeroDivisionError`）。
- `exc_value` : 异常实例。有时会有参数传给异常构造方法，例如错误消息，这些参数可以使用 `exc_value.args` 获取。
- `traceback` : `traceback` 对象。 [ `sys.exc_info()` 就返回这三个参数.]

**调用:**

```python
>>> with LookingGlass() as what:  # 管理器是实例, __enter__ 返回的变量 绑定到 what.
    ...      print('Alice, Kitty and Snowdrop')  # 打印变量做比较.
    ...      print(what)
    ...
pordwonS dna yttiK ,ecilA  # 内容反向
YKCOWREBBAJ
>>> what  # with 语句执行完，可查看变量.[with语句并没有创建作用域]
'JABBERWOCKY'
>>> print('Back to normal.')  # with语句执行完后，内容便不再反向.
Back to normal.

# 在 with 块之外使用 LookingGlass 类
>>> from mirror import LookingGlass
>>> manager = LookingGlass()  # 实例化并审查 manager 实例。
>>> manager
<mirror.LookingGlass object at 0x2a578ac>
>>> monster = manager.__enter__()  # 返回结果存在 monster 中.
>>> monster == 'JABBERWOCKY'  # 反向的打印内容。
eurT
>>> monster
'YKCOWREBBAJ'
>>> manager
>ca875a2x0 ta tcejbo ssalGgnikooL.rorrim<
>>> manager.__exit__(None, None, None)  # 调用 __exit__ ， 还原之前的内容，并模拟参数.
>>> monster
'JABBERWOCKY'  # 不再反向输出内容。
```

标准库中有一些示例:

- [12.6.7.3. Using the connection as a context manager](https://docs.python.org/3/library/sqlite3.html#using-the-connection-as-a-context-manager), 在 sqlite3 模块中用于管理事务
- [17.1.10. Using locks, conditions, and semaphores in the with statement](https://docs.python.org/3/library/threading.html#using-locks-conditions-and-semaphores-in-the-with-statement), 在 threading 模块中用于维护锁、条件和信号.
- [decimal.localcontext 函数的文档](https://docs.python.org/3/library/decimal.html#decimal.localcontext), 为 Decimal 对象的算术运算设置环境.
- [unittest.mock.patch 函数的文档](https://docs.python.org/3/library/unittest.mock.html#patch), 为了测试临时给对象打补丁

<span id="contextlib工具"></span>

## contextlib工具

[29.6 contextlib — Utilities for with-statement contexts](https://docs.python.org/3/library/contextlib.html)

- **closing**：
  - 如果对象提供了 `close()` 方法，但没有实现 `__enter__`/`__exit__` 协议，那么可以使用这个函数构建上下文管理器。
- **suppress**:
  - 构建临时忽略指定异常的上下文管理器。
- **@contextmanager:**
  - 这个装饰器把简单的生成器函数变成上下文管理器，这样就不用创建类去实现管理器协议了。
- **ContextDecorator**
  - 这是个基类，用于定义基于类的上下文管理器。这种上下文管理器也能用于装饰函数，在受管理的上下文中运行整个函数。
- **ExitStack:**
  - 这个上下文管理器能进入多个上下文管理器。 `with` 块结束时， `ExitStack` 按照后进先出的顺序调用栈中各个上下文管理器的 `__exit__` 方法。如果事先不知道 `with` 块要进入多少个上下文管理器，可以使用这个类。例如，同时打开任意一个文件列表中的所有文件。


**@contextmanager** 使用方法:

在使用 `@contextmanager` 装饰的 `生成器` 中， `yield` 语句的作用是把函数的定义体分成两部分： `yield` 语句前面的所有代码在 `with` 块开始时（即解释器调用 `__enter__` 方法时）执行， `yield` 语句后面的代码在 `with` 块结束时（即调用 `__exit__` 方法时）执行。

```python
import contextlib


@contextlib.contextmanager  # 应用 contextmanager 装饰器
def looking_glass():
    import sys
    original_write = sys.stdout.write  # 贮存原来的 sys.stdout.write 方法

    def reverse_write(text):  # 定义自定义的 reverse_write 函数；在闭包中可以访问 original_write。
        original_write(text[::-1])

    sys.stdout.write = reverse_write  # 把 sys.stdout.write 替换成 reverse_write。
    yield 'JABBERWOCKY'  # 产出一个值，这个值会绑定到 with 语句中 as 子句的目标变量上。执行 with 块中的代码时，这个函数会在这一点暂停。
    sys.stdout.write = original_write  # 控制权一旦跳出 with 块，继续执行 yield 语句之后的代码；这里是恢复成原来的 sys.stdout.write 方法。

# 测试
>>> from mirror_gen import looking_glass
>>> with looking_glass() as what:   # 这里 what 的变量就是 yield 生成的值了.
...      print('Alice, Kitty and Snowdrop')
...      print(what)
...
pordwonS dna yttiK ,ecilA
YKCOWREBBAJ
>>> what
'JABBERWOCKY'
```

**其实，`contextlib.contextmanager` 装饰器会把函数包装成实现 __enter__ 和 __exit__ 方法的类。**

类的名称是 `_GeneratorContextManager`, 可以参考源码: [contextlib](https://hg.python.org/cpython/file/3.4/Lib/contextlib.py#l34)

捕捉异常版本:

```python
import contextlib


@contextlib.contextmanager
def looking_glass():
    import sys
    original_write = sys.stdout.write

    def reverse_write(text):
        original_write(text[::-1])

    sys.stdout.write = reverse_write
    msg = ''
    try:
        yield 'JABBERWOCKY'
    except ZeroDivisionError:
        msg = 'Please DO NOT divide by zero!'
    finally:
        sys.stdout.write = original_write
        if msg:
            print(msg)
```

装饰器提供的 `__exit__` 方法假定发给生成器的所有异常都得到处理了，因此应该压制异常. 如果不想让 `@contextmanager` 压制异常，必须在被装饰的函数中显式重新抛出异常。

使用 `@contextmanager` 装饰器时，要把 `yield` 语句放在 `try/finally` 语句中（或者放在 `with` 语句中），这是无法避免的，因为我们永远不知道上下文管理器的用户会在 `with` 块中做什么。

示例：用于原地重写文件的上下文管理器

```python
import csv

with inplace(csvfilename, 'r', newline='') as (infh, outfh):
    reader = csv.reader(infh)
    writer = csv.writer(outfh)

    for row in reader:
        row += ['new', 'columns']
        writer.writerow(row)
```

`inplace` 函数是个上下文管理器，为同一个文件提供了两个句柄（这个示例中的 `infh` 和 `outfh` ），以便同时读写同一个文件。这比标准库中的 [fileinput.input](https://docs.python.org/3/library/fileinput.html#fileinput.input) 函数易于使用。

源码参考： [inplace](http://www.zopatista.com/python/2013/11/26/inplace-file-rewriting/)

`yield` 关键字之前的所有代码都用于设置上下文：先创建备份文件，然后打开并产出 `__enter__` 方法返回的可读和可写文件句柄的引用。

`yield` 关键字之后的 `__exit__` 处理过程把文件句柄关闭；如果什么地方出错了，那么从备份中恢复文件。

<span id="扩展阅读"></span>

## 扩展阅读

- [What Makes Python Awesome?](https://speakerdeck.com/pyconslides/pycon-keynote-python-is-awesome-by-raymond-hettinger?slide=21), with 不仅能管理资源，还能用于去掉常规的设置和清理代码，或者在另一个过程前后执行的操作.
- [Compound statements](https://docs.python.org/3/reference/compound_stmts.html), 说明了 `if` 、 `for` 、`while` 和 `try` 语句的 `else` 子句.
- [Is it a good practice to use try-except-else in Python?](http://stackoverflow.com/questions/16138232/is-it-a-good-practice-to-use-try-except-else-in-python), 关于 `try/except` 语句（有 `else` 子句，或者没有）是否符合 Python 风格
- [上下文管理器的类型](https://docs.python.org/3/library/stdtypes.html#typecontextmanager).
- [With Statement Context Managers](https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers), `__enter__/__exit__` 两个特殊方法的文档.
- [PEP 343—The‘with’Statement](https://www.python.org/dev/peps/pep-0343/), 上下文管理器 pep文档，不过不易读懂，因为大量篇幅都在讲极端情况，以及反对其他提案。
- [Transforming Code into Beautiful, Idiomatic Python](https://speakerdeck.com/pyconslides/transforming-code-into-beautiful-idiomatic-python-by-raymond-hettinger-1?slide=34), 展示了上下文管理器的几个有趣应用.
- [The Python with Statement by Example](http://preshing.com/20110920/the-python-with-statement-by-example/), 举例说明了 pycairo 图形库中的上下文管理器。
- 《Python Cookbook（第 3 版）中文版》
  - "8.3 让对象支持上下文管理协议”一节实现了一个 `LazyConnection` 类，它的实例是上下文管理器，在 with 块中能自动打开和关闭网络连接。
  - “9.22 以简单的方式定义上下文管理器”一节编写了一个用于统计代码运行时间的上下文管理器，还编写了一个使用事务修改 list 对象的上下文管理器：在 with 块中创建 list 实例的副本，所有改动都针对那个副本；仅当 with 块没有抛出异常，正常执行完毕之后，才用副本替代原来的列表。
