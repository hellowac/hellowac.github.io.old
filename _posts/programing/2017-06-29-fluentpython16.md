---
nav: blog
layout: post
title: "流畅的python - 协程"
author: "wangchao"
tags:
  - python
  - 'yield from'
  - '协程'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [yield from](#yieldfrom)
- [yield form的实现](#yieldfrom的实现)
- [yield from 案例](#yieldfrom案例)
- [扩展阅读](#扩展阅读)

**协程底层架构定义:** [PEP 342—Coroutines via Enhanced Generators](https://www.python.org/dev/peps/pep-0342/)

- **协程语法:** [PEP 380—Syntax for Delegating to a Subgenerator](https://www.python.org/dev/peps/pep-0380/)
  - 以前，如果在生成器中给 `return` 语句提供值，会抛出 `SyntaxError` 异常, 现在，生成器可以返回一个值；
  - 新引入了 `yield from` 句法，使用它可以把复杂的生成器重构成小型的嵌套生成器，省去了之前把生成器的工作委托给子生成器所需的大量样板代码。

- **获取协程状态:**  `inspect.getgeneratorstate(...) `
  - `GEN_CREATED` : 等待开始执行。
  - `GEN_RUNNING` : 解释器正在执行.
  - `GEN_SUSPENDED` : 在 `yield` 表达式处暂停。
  - `GEN_CLOSED` : 执行结束。

- **激活(预激)协程:** `next(gen)` 或 `gen.send(None)` 一样的效果.
  - 让协程向前执行到第一个 `yield` 表达式，准备好作为活跃的协程使用。
  - [预激的装饰器](#预激的装饰器)

- **控制协程:**
  - `generator.throw(exc_type[, exc_value[, traceback]])` :
    - 致使生成器在暂停的 `yield` 表达式处抛出指定的异常。如果生成器处理了抛出的异常，代码会向前执行到下一个 `yield` 表达式，而产出的值会成为调用 `generator.throw` 方法得到的返回值。如果生成器没有处理抛出的异常，异常会向上冒泡，传到调用方的上下文中。
  - `generator.close()` :
    - 致使生成器在暂停的 `yield` 表达式处抛出 `GeneratorExit` 异常。如果生成器没有处理这个异常，或者抛出了 `StopIteration` 异常（通常是指运行到结尾），调用方不会报错。如果收到 `GeneratorExit` 异常，生成器一定不能产出值，否则解释器会抛出 `RuntimeError` 异常。生成器抛出的其他异常会向上冒泡，传给调用方。
    - **注意**: 可通过 `throw()` 方法 动态控制协程的行为. 【好高级的样子】
  - 发送哨符让协程退出. 比如 内置常量 `None` 和 `Ellipsis`。 或者其他的自定义的变量.

- **协程返回值:**
  - `return` 表达式的值会赋值给 StopIteration 异常的一个属性, 传回给调用方。
  - **这是 [PEP 380](https://www.python.org/dev/peps/pep-0380/) 定义的方式：**
    - `yield from` 结构会在内部自动捕获 `StopIteration` 异常。这种处理方式与 `for` 循环处理 `StopIteration` 异常的方式一样：循环机制使用用户易于理解的方式处理异常。对 `yield from` 结构来说，解释器不仅会捕获 `StopIteration` 异常，还会把 `value` 属性的值变成 `yield from` 表达式的值。

- **处理协程的特殊装饰器:**
  - [tornado.gen 装饰器](http://tornado.readthedocs.org/en/latest/gen.html)


```python
In [6]: def gen():
   ...:     print('started...')
   ...:     va = yield 'v1'
   ...:     print('recevice {0}'.format(va))
   ...:     va = yield 'v2'
   ...:     print('recevice {0}'.format(va))
   ...:     print('end generator')
   ...:     return 'end gen1', 'end gen2'   # python2 中会报错.
   ...:

In [7]: a = gen()

In [8]: next(a)
started...
Out[8]: 'v1'

In [9]: a.send('s1')
recevice s1
Out[9]: 'v2'

In [11]: a.send('s2')
recevice s2
end generator
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-11-b2472176ced9> in <module>()
----> 1 a.send('s2')

StopIteration: ('end gen1', 'end gen2')   # python3 中 return 语句的返回值在 StopIteration 的 value属性中.

In [12]: def average():
    ...:     total = 0.0
    ...:     count = 0
    ...:     average = None
    ...:     while 1:
    ...:         num = yield average
    ...:         total += num
    ...:         count += 1
    ...:         average = total/count
    ...:

In [20]: averager = average()

In [21]: next(averager)

In [22]: averager.send(12)
Out[22]: 12.0

In [23]: averager.send(14)
Out[23]: 13.0

In [24]: averager.send(20)
Out[24]: 15.333333333333334

In [25]: try:
    ...:     averager.throw(StopIteration('stop averge'))
    ...: except StopIteration as e:
    ...:     print('stoped value is :{0}'.format(e.value))
    ...:
/usr/local/bin/ipython3:2: DeprecationWarning: generator 'average' raised StopIteration

stoped value is :stop averge      # 该异常 原样抛出. 【内部可根据异常类型来控制行为】
```

<span id="预激的装饰器"></span>

```python
from functools import wraps

def coroutin(func):
    @wraps(func)
    def primer(*args, **kwargs):
        gen = func(*args, **kwargs)   # 获取生成器对象.
        next(gen)       # 预激 协程.
        return gen

    return primer
```

<span id="yieldfrom"></span>

**yield from 语句**

- 在生成器 `gen` 中使用 `yield from subgen()` 时，`subgen` 会获得控制权，把产出的值传给 `gen` 的调用方，即调用方可以直接控制 `subgen`。
与此同时，`gen` 会阻塞，等待 `subgen` 终止。
- `yield from x` 表达式对 `x` 对象所做的第一件事是，调用 `iter(x)`，从中获取迭代器。因此，`x` 可以是任何可迭代的对象。 如 cookbook 中示例: [how to flatten a nested sequence](https://github.com/dabeaz/python-cookbook/blob/master/src/4/how_to_flatten_a_nested_sequence/example.py)
- 引入 `yield from` 结构的目的是为了支持实现了 `__next__` 、 `send` 、 `close` 和 `throw` 方法的生成器。所以 `yield from` 的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来，这样二者可以直接发送和产出值，还可以直接传入异常，而不用在位于中间的协程中添加大量处理异常的样板代码。有了这个结构，协程可以通过以前不可能的方式委托职责。
  - **委派生成器:** 包含 `yield from <iterable>` 表达式的生成器函数。
  - **子生成器:** 从 `yield from` 表达式中 `<iterable>` 部分获取的生成器。
  - **调用方:** `PEP 380` 使用“调用方”这个术语指代调用委派生成器的客户端代码。
  - 示例：[Syntax for Delegating to a Subgenerator](https://docs.python.org/3/whatsnew/3.3.html#pep-380)
    - 该示例想表明的关键一点是，如果 `子生成器` 不终止， `委派生成器` 会在 yield from 表达式处永远暂停。
- `yield from` 结构参考: [construct works](http://flupy.org/resources/yield-from.pdf)
- 另一个示例：[coroaverager3](https://github.com/fluentpython/example-code/blob/master/16-coroutine/coroaverager3.py)
- 委派生成器相当于管道，所以可以把任意数量个委派生成器连接在一起：一个委派生成器使用 `yield from` 调用一个子生成器，而那个子生成器本身也是委派生成器，使用 `yield from` 调用另一个子生成器，以此类推。最终，这个链条要以一个只使用 `yield` 表达式的简单生成器结束；不过，也能以任何可迭代的对象结束.

```python
In [3]: def gen():
   ...:     for c in 'AB':
   ...:         yield c
   ...:     for i in range(1,3):
   ...:         yield i
   ...:

In [4]: list(gen())
Out[4]: ['A', 'B', 1, 2]

In [5]: def gen():
   ...:     yield from 'AB'
   ...:     yield from range(1, 3)
   ...:

In [6]: list(gen())
Out[6]: ['A', 'B', 1, 2]
```

<span id="yieldfrom的实现"></span>

## yield from 的实现

- [PEP 380 (yield from a subgenerator) comments](https://mail.python.org/pipermail/python-dev/2009-March/087385.html) 中的一句话:
    - 把迭代器当作生成器使用，相当于把子生成器的定义体内联在 `yield from` 表达式中。此外，子生成器可以执行 `return` 语句，返回一个值，而返回的值会成为 `yield from` 表达式的值。
- [PEP 380 “Proposal”](https://www.python.org/dev/peps/pep-0380/#proposal) 解释的行为:
    - 子生成器产出的值都直接传给委派生成器的调用方（即客户端代码）。
    - 使用 `send()` 方法发给委派生成器的值都直接传给子生成器。如果发送的值是 `None` ，那么会调用子生成器的 `__next__()` 方法。如果发送的值不是 `None` ，那么会调用子生成器的 `send()` 方法; 如果调用的子生成器方法抛出 `StopIteration` 异常，那么委派生成器恢复运行；任何其他子生成器的异常都会向上冒泡，传给委派生成器。
    - 生成器中的 `return expr` 语句会抛出 `StopIteration(expr)` 异常并退出.
    - 传入委派生成器的异常，除了 `GeneratorExit` 之外都传给子生成器的 `throw()` 方法。如果调用 `throw()` 方法时抛出 `StopIteration` 异常，委派生成器恢复运行。`StopIteration` 之外的异常会向上冒泡，传给委派生成器。
        - 我的理解：传递给委派生成器的`GeneratorExit`异常可捕捉，否则抛出并且当前委派生成器的状态为 `GEN_CLOSED` ，但不会传递给子生成器; 如果 子生成器抛出 `StopIteration`异常，委派生成器的`yield from`语句可处理，其他异常需手动捕获，否则向上抛出。
    - 如果把 `GeneratorExit` 异常传入委派生成器，或者在委派生成器上调用 `close()` 方法，那么在子生成器上调用 `close()` 方法，如果它有的话。如果调用 `close()` 方法导致异常抛出，那么异常会向上冒泡，传给委派生成器；否则，委派生成器抛出 `GeneratorExit` 异常。
        - 参考:[pep 380 formal-semantics](https://www.python.org/dev/peps/pep-0380/#formal-semantics)

<span id="yieldfrom案例"></span>

## yield from 案例: 出租车队运营仿真

在仿真领域，`进程`这个术语指代模型中某个实体的活动，与操作系统中的进程无关。
仿真系统中的一个`进程`可以使用操作系统中的一个进程实现，但是通常会使用一个线程或一个协程实现。

```python
import random
import collections
import queue
import argparse
import time

DEFAULT_NUMBER_OF_TAXIS = 3
DEFAULT_END_TIME = 180
SEARCH_DURATION = 5
TRIP_DURATION = 20
DEPARTURE_INTERVAL = 5

Event = collections.namedtuple('Event', 'time proc action')


# 出租车进程
def taxi_process(ident, trips, start_time=0):
    """每次状态改变便会 yield 一个模式事件"""
    time = yield Event(start_time, ident, '离开车库')
    for i in range(trips):  # 跑几班车?
        time = yield Event(time, ident, '客人上车')
        time = yield Event(time, ident, '客人下车')

    yield Event(time, ident, '下班回家')


# 出租车运营模拟
class Simulator:

    def __init__(self, procs_map):
        self.events = queue.PriorityQueue()  # 优先队列, 最先返回值最低的任务.
        self.procs = dict(procs_map)

    def run(self, end_time):
        """调度和展示事件直到规定时间"""

        # 调度每辆出租车的第一个事件
        for _, proc in sorted(self.procs.items()):
            first_event = next(proc)
            self.events.put(first_event)

        # 主调度循环
        sim_time = 0
        while sim_time < end_time:  # 主调度时间小于规定时间
            if self.events.empty():
                print('*** 调度所有事件完成 ***')
                break

            current_event = self.events.get()  # 此时已从队列中取出时间值最小的事件, 【所以已打乱插入时的事件顺序】
            sim_time, proc_id, previous_action = current_event  # 上次调度时间, 事件id, 上一个动作.
            print('出租车:', proc_id, proc_id * '   ', current_event)  # 展示当前处理掉的动作.
            active_proc = self.procs[proc_id]  # 激活的事件
            next_time = sim_time + compute_duration(previous_action)  # 获取上一个动作所需时间并累加到下次调度时间.
            try:
                next_event = active_proc.send(next_time)  # 传入该事件下次动作调度时间.
            except StopIteration:
                del self.procs[proc_id]  # 事件已完成，删除该事件.
            else:
                self.events.put(next_event)  # 将下个动作添加到队列.
        else:
            msg = '*** 规定时间内调度结束: {} 个事件还在等待调度 ***'
            print(msg.format(self.events.qsize()))



def compute_duration(previous_action):
    """使用指数分布计算事件所需时间"""
    if previous_action in ['离开车库', '客人下车']:
        # 新事件等待中
        interval = SEARCH_DURATION
    elif previous_action == '客人上车':
        # 新事件进行中
        interval = TRIP_DURATION
    elif previous_action == '下班回家':
        interval = 1
    else:
        raise ValueError('未定义的上一事件: %s' % previous_action)
    return int(random.expovariate(1/interval)) + 1  # 指数分布计算时间.


def main(end_time=DEFAULT_END_TIME, num_taxis=DEFAULT_NUMBER_OF_TAXIS,
         seed=None):
    """初始化随机生成器, 创建进程和运行调度模拟器"""
    if seed is not None:
        random.seed(seed)  # 改变随机数的种子seed.

    taxis = {i: taxi_process(i, (i+1)*2, i*DEPARTURE_INTERVAL)
             for i in range(num_taxis)}  # 创建指定数量的出租车事件.
    sim = Simulator(taxis)
    sim.run(end_time)


if __name__ == '__main__':

    parser = argparse.ArgumentParser(
                        description='出租车运营仿真器')
    parser.add_argument('-e', '--end-time', type=int,
                        default=DEFAULT_END_TIME,
                        help='仿真器结束时间; 默认 = %s'
                        % DEFAULT_END_TIME)
    parser.add_argument('-t', '--taxis', type=int,
                        default=DEFAULT_NUMBER_OF_TAXIS,
                        help='模拟运营的出租车数量; 默认 = %s'
                        % DEFAULT_NUMBER_OF_TAXIS)
    parser.add_argument('-s', '--seed', type=int, default=None,
                        help='random 随机种子 (仅测试)')

    args = parser.parse_args()
    main(args.end_time, args.taxis, args.seed)

# 测试
$ python3 taxi_sim.py -e 120 -t 3
出租车: 0  Event(time=0, proc=0, action='离开车库')
出租车: 1     Event(time=5, proc=1, action='离开车库')
出租车: 0  Event(time=8, proc=0, action='客人上车')
出租车: 1     Event(time=10, proc=1, action='客人上车')
出租车: 2        Event(time=10, proc=2, action='离开车库')
出租车: 2        Event(time=13, proc=2, action='客人上车')
出租车: 2        Event(time=16, proc=2, action='客人下车')
出租车: 2        Event(time=17, proc=2, action='客人上车')
出租车: 1     Event(time=22, proc=1, action='客人下车')
出租车: 1     Event(time=37, proc=1, action='客人上车')
出租车: 0  Event(time=42, proc=0, action='客人下车')
出租车: 1     Event(time=45, proc=1, action='客人下车')
出租车: 0  Event(time=46, proc=0, action='客人上车')
出租车: 1     Event(time=50, proc=1, action='客人上车')
出租车: 2        Event(time=59, proc=2, action='客人下车')
出租车: 1     Event(time=61, proc=1, action='客人下车')
出租车: 0  Event(time=65, proc=0, action='客人下车')
出租车: 1     Event(time=65, proc=1, action='客人上车')
出租车: 2        Event(time=65, proc=2, action='客人上车')
出租车: 0  Event(time=66, proc=0, action='下班回家')
出租车: 1     Event(time=78, proc=1, action='客人下车')
出租车: 1     Event(time=90, proc=1, action='下班回家')
出租车: 2        Event(time=116, proc=2, action='客人下车')
出租车: 2        Event(time=122, proc=2, action='客人上车')
*** 规定时间内调度结束: 1 个事件还在等待调度 ***
```

- 仿真调度库: [SimPy 的官方文档](https://simpy.readthedocs.org/en/latest/)
    - 不要搞混 [SymPy](http://www.sympy.org/), SymPy 是一个符号数学库，与 DES 无关。

<span id="扩展阅读"></span>

## 扩展阅读

- Beazley `大神` 开设过的课程:
    - [Generator Tricks for Systems Programmers](http://www.dabeaz.com/generators/), 课程, PyCon US 2008 期间
    - [A Curious Course on Coroutines and Concurrency](http://www.dabeaz.com/coroutines/), 课程. PyCon US 2009 期间
        - 三个部分的全部视频链接: [第一部分](http://pyvideo.org/video/213) 、 [第二部分](http://pyvideo.org/video/215) 、 [第三部分](http://pyvideo.org/video/214)
    - [Generators: The Final Frontier](http://www.dabeaz.com/finalgenerator/), 课程。 举了更多并发的例子。最后一部分用协程代替了经典的访问者模式，用于计算算术表达式。
- [Greedy algorithm with coroutines](http://seriously.dontusethiscode.com/2013/05/01/greedy-coroutine.html), James Powell 写的一篇文章， 章中使用协程重写了经典的算法。
    - [ActiveState Code 诀窍数据库](https://code.activestate.com/recipes/) , 协程的 [流行诀窍](https://code.activestate.com/recipes/tags/coroutine/)。
- [yield from 示意图](http://flupy.org/resources/yield-from.pdf)，  Paul Sokolovsky 为 Damien George 开发的超级精简的 [MicroPython](http://micropython.org/)（针对微控制器）解释器实现 `yield from` 结构过程中所做得示意图.
- [使用示例](http://www.cosc.canterbury.ac.nz/greg.ewing/python/yield-from/yield_from.html), Greg Ewing（PEP 380 的作者，为 CPython 实现了 yield from）发表的 `yield from` 使用示例.
    - `asyncio` 库本身和使用这个库的代码大量使用 `yield from`
- [用协程来并发地运行多个函数](http://www.effectivepython.com/2015/03/10/consider-coroutines-to-run-many-functions-concurrently/), 来自 《Effective Python：编写高质量 Python 代码的 59 个有效方法》
    - [生命游戏-源码](https://github.com/bslatkin/effectivepython/blob/master/example_code/item_40.py)， 作者重构后 代码 [生命游戏-重构](https://gist.github.com/ramalho/da5590bc38c973408839)
- [Comparing two CSV files using Python](https://mail.python.org/pipermail/tutor/2015-February/104200.html), Peter Otten 在 Python Tutor 邮件列表中发布的消息
- [Iterables, Iterators, and Generators](http://nbviewer.ipython.org/github/wardi/iterables-iterators-generators/blob/master/Iterables,%20Iterators,%20Generators.ipynb), 教程， Ian Ward 以 iPython Notebook 形式发布， 实现的是剪刀石头布游戏。
- [The difference between yield and yield-from](https://groups.google.com/forum/#!msg/python-tulip/bmphRrryuFk/aB45sEJUomYJ), Guido van Rossum 在 python-tulip Google Group 中发表.
- [087382.html](https://mail.python.org/pipermail/python-dev/2009-March/087382.html), Nick Coghlan 在 Python-Dev 邮件列表中发布的带有大量注释的 `yield from` 扩充实现
- [PEP 492—Coroutines with async and await syntax](https://www.python.org/dev/peps/pep-0492/), 提议为 `Python` 增加两个关键字：`async` 和 `await`。
    - `async` 与其他现有的关键字结合使用，用于定义新的语言结构。
    - 例如，`async def` 用于定义协程，`async for` 用于使用异步迭代器（实现 `__aiter__` 和 `__anext__` 方法，这是协程版的 `__iter__` 和 `__next__` 方法）迭代可迭代的异步对象.
    - 为了避免与即将引入的 `async` 关键字冲突，`asyncio.async()` 函数将在 `Python 3.4.4` 中重命名为 `asyncio.ensure_future()`。`await` 关键字的作用与 `yield from` 结构类似，不过只能在以 `async def` 定义的协程（禁止在使用 `yield` 和 `yield from` 的结构）中使用。
- [Discrete event simulation](https://en.wikipedia.org/wiki/Discrete_event_simulation), 维基百科中入门协程的资料。使用离散事件仿真系统做试验是熟悉协作式多任务的好方法。
- [Writing a Discrete Event Simulation: Ten Easy Lessons](http://www.cs.northwestern.edu/~agupta/_projects/networking/QueueSimulation/mm1.html), Ashish Gupta 写的短篇教程, 说明如何自己动手（不使用特别的库）编写离散事件仿真系统。
