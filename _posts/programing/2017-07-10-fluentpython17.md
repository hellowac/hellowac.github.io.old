---
nav: blog
layout: post
title: "流畅的python - Future"
author: "wangchao"
tags:
  - python
  - 'Future'
  - '并行'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [concurrent.futures模块](#concurrent_futures)
- [显示进度条](#显示进度条)
- [扩展阅读](#扩展阅读)

<span id="concurrent_futures"></span>

### concurrent.futures模块

类 `ThreadPoolExecutor` 和 类 `ProcessPoolExecutor`.

这两个类实现的接口能分别在不同的线程或进程中执行可调用的对象。

这两个类在内部维护着一个工作线程或进程池，以及要执行的任务队列。

接口抽象的层级很高，无需关心任何实现细节。

**ThreadPoolExecutor.map** 方法：

```python
from concurrent import futures

with futures.ThreadPoolExecutor(worker_num) as executor:  # 最大的线程数量.
    res = executor.map(func, *iterables) # 并发调用 do_something. 返回 迭代器.

len(list(res))  # 如果有线程抛出异常，异常会在这里抛出，与隐式调用 next() 函数从迭代器中获取相应的返回值一样。
```

**Futures**

从 `Python 3.4` 起，标准库`concurrent.futures.Future` 和 `asyncio.Future` 两个库中有名为 `Future` 的类.

两个类都表示可能已经完成或者尚未完成的延迟计算。 【 与 `Twisted` 引擎中的 `Deferred` `类、Tornado` 框架中的 `Future` 类，以及多个 `JavaScript` 库中的 `Promise` 对象类似。】

`Future` 封装待完成的操作，放入队列，状态可以查询, 之后得到结果 或 抛出异常.

- **注意:** 通常情况下自己不应该创建期物，而只能由并发框架（`concurrent.futures` 或 `asyncio` ）实例化。
    - 期物表示终将发生的事情，而确定某件事会发生的唯一方式是执行的时间已经排定。
    - 只有排定把某件事交给 `concurrent.futures.Executor` 子类处理时，才会创建 `concurrent.futures.Future` 实例。
    - 例如，`Executor.submit()` 方法的参数是一个可调用的对象，调用这个方法后会为传入的可调用对象排期，并返回一个 `Future`。
- **注意:** 客户端代码不应该改变期物的状态，并发框架在期物表示的延迟计算结束后会改变期物的状态.

**通用方法:**

- 这两个 `Future` 的 `.done()` 方法 会以 不阻塞 的方式返回一个 布尔值 ， 表明 `Future` 封装的可调用对象是否已经执行。
- 这两个 `Future` 的 `.add_done_callback(callable)` 方法， 在运行结束后会调用指定的可调用对象。
- 这两个 `Future` 的 `.result()` 方法在运行结束后返回可调用对象的结果,或者重新抛出执行可调用的对象时抛出的异常. 可是，如果 `Future` 没有运行结束:
    - 对 `concurrency.futures.Future` 实例来说:
        - 调用 `.result()` 方法会阻塞调用方所在的线程，直到有结果可返回。
        - 此时，`.result()` 方法可以接收可选的 `timeout` 参数，如果在指定的时间内 `Future` 实例没有运行完毕，会抛出 `TimeoutError` 异常。
    - 对 `asyncio.Future` 实例来说:
        - `.result()` 方法不支持 `timeout` 参数. 获取结果最好使用 `yield from` 结构。不过，对 `concurrency.futures.Future` 实例不能这么做。

**与Future有关的方法:**

- `concurrent` 和 `asyncio` 库:
    - `Executor.map` 方法: 返回一个迭代器，迭代器的 `__next__` 方法调用各个期物的 `result` 方法，因此得到的是各个`Future`实例的结果，而非实例本身。
    - [`concurrent.futures.as_completed()`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.as_completed) 函数, 参数是一个 `Future` 实例列表，返回值是一个迭代器，在运行结束后产出 `Future`实例 。
    - `concurrent.futures.Executor.submit(fn, *args, **kwargs)` 方法: 调度一个 可执行对象、函数 就像 `fn(*args, **kwargs)` 一样. 返回一个 Future 实例对象.

```python
from concurrent import futures

with futures.ThreadPoolExecutor(max_workers=3) as executor:  # 创建线程执行器
        to_do = []
        for arg in sorted(args):  # 循环参数列表
            future = executor.submit(callable_or_func, arg)  # 加入线程调度.
            to_do.append(future)  # 添加封装任务的Future实例.
            msg = 'Scheduled for {}: {}'
            print(msg.format(arg, future))  # 显示对应的消息

        results = []
        for future in futures.as_completed(to_do):  # 在Future实例运行结束后产出.
            res = future.result() # 此时不会阻塞，因为 as_completed 只会产出运行结束后的Future实例.
            msg = '{} result: {!r}'
            print(msg.format(future, res))  # 显示对应的结果.
            results.append(res)

len(results)  # 此时也不会阻塞
```

**注意:** `CPython` 解释器本身就不是线程安全的，因此有全局解释器锁（GIL），一次只允许使用一个线程执行 `Python` 字节码。因此，一个 `Python` 进程通常不能同时使用多个 CPU 核心。

编写 `Python` 代码时无法控制 `GIL` ；不过，执行耗时的任务时，可以使用一个内置的函数或一个使用 `C` 语言编写的扩展释放 `GIL` 。其实，有个使用 `C` 语言编写的 `Python` 库能管理 `GIL` ，自行启动操作系统线程，利用全部可用的 `CPU` 核心。这样做会极大地增加库代码的复杂度，因此大多数库的作者都不这么做。

然而，标准库中所有执行阻塞型 `I/O` 操作的函数，在等待操作系统返回结果时都会释放 `GIL` 。这意味着在 `Python` 语言这个层次上可以使用多线程，而 `I/O` 密集型 `Python` 程序能从中受益：一个 `Python` 线程等待网络响应时，阻塞型 `I/O` 函数会释放 `GIL`，再运行一个线程。

`Python`标准库 中的所有阻塞型 `I/O` 函数都会释放 `GIL` ，允许其他线程运行。`time.sleep()` 函数也会释放 `GIL` 。因此，尽管有 `GIL` ， `Python` 线程还是能在 `I/O` 密集型应用中发挥作用。

**ProcessPoolExecutor类**

`concurrent.futures.ProcessPoolExecutor` 类把工作分配给多个 `Python` 进程处理。因此，如果做 `CPU` 密集型处理，使用这个模块能绕开 `GIL` ，利用所有可用的 `CPU` 核心。

`ProcessPoolExecutor` 和 `ThreadPoolExecutor` 类都实现了通用的 `Executor` 接口.

```python
with futures.ProcessPoolExecutor() as executor:  # 这里不需要 `ThreadPoolExecutor.__init__` 的最大工作数量. 那个参数是可选的，而且大多数情况下不使用——默认值是 os.cpu_count() 函数返回的 CPU 数量。
    do_something
```

`ProcessPoolExecutor` 的价值体现在 `CPU` 密集型作业上. 如果使用 `Python` 处理 CPU 密集型工作，应该试试 [PyPy](http://pypy.org/) 。

**Executor.map** 方法:

返回结果的顺序与调用开始的顺序一致, 如果第一个调用生成结果用时 `10` 秒，而其他调用只用 `1` 秒，代码会阻塞 `10` 秒，获取 `map` 方法返回的生成器产出的第一个结果。

在此之后，获取后续结果时不会阻塞，因为后续的调用已经结束。如果必须等到获取所有结果后再处理，这种行为没问题；

不过，通常更可取的方式是，不管提交的顺序，只要有结果就获取。为此，要把 `Executor.submit` 方法和 `futures.as_completed` 函数结合起来使用

`executor.submit` 和 `futures.as_completed` 这个组合比 `executor.map` 更灵活，因为 `submit` 方法能处理不同的可调用对象和参数，而 e`xecutor.map` 只能处理参数不同的同一个可调用对象。

此外，传给 `futures.as_completed` 函数的期物集合可以来自多个 `Executor` 实例，例如一些由 `ThreadPoolExecutor` 实例创建，另一些由 `ProcessPoolExecutor` 实例创建。

<span id="显示进度条"></span>

### 显示进度条

```python
>>> import time
>>> from tqdm import tqdm
>>> for i in tqdm(range(1000)):
...     time.sleep(.01)
...
>>> # -> 进度条会出现在这里 <-
```

`tqdm` 函数的实现方式 :

- 能处理任何可迭代的对象，生成一个迭代器；使用这个迭代器时，显示进度条和完成全部迭代预计的剩余时间。
- 为了计算预计剩余时间， `tqdm` 函数要获取一个能使用 `len` 函数确定大小的可迭代对象，或者在 `第二个参数` 中指定预期的元素数量。

```python
with futures.ThreadPoolExecutor(max_workers=concur_req) as executor:    # 穿件多线程管理器
    to_do_map = {}  # 特征映射，Future可以做键，可见可以被hash.
    for cc in sorted(cc_list):  # 排序
        future = executor.submit(download_one,
                        cc, base_url, verbose)  # 提交任务
        to_do_map[future] = cc  # 存储特征
    done_iter = futures.as_completed(to_do_map) # 等待执行完毕
    if not verbose:
        done_iter = tqdm.tqdm(done_iter, total=len(cc_list))    # 传入iterable 以及预计长度.
```

**总结:**

`concurrent.futures` 是使用线程的最新方式。
`Python 3` 废弃了原来的 `thread` 模块，换成了高级的 `threading` 模块。
如果 `futures.ThreadPoolExecutor` 类对某个作业来说不够灵活，可能要使用 `threading` 模块中的组件（如 `Thread` 、 `Lock` 、 `Semaphore`  等）自行制定方案，比如说使用 [queue](https://docs.python.org/3/library/queue.html) 模块创建线程安全的队列，在线程之间传递数据。
`futures.ThreadPoolExecutor` 类已经封装了这些组件。

对 `CPU` 密集型工作来说，要启动多个进程，规避 `GIL`。
创建多个进程最简单的方式是，使用 `futures.ProcessPoolExecutor` 类。
不过和前面一样，如果使用场景较复杂，需要更高级的工具。
[multiprocessing](https://docs.python.org/3/library/multiprocessing.html) 模块的 `API` 与 `threading` 模块相仿，不过作业交给多个进程处理。
对简单的程序来说，可以用 `multiprocessing` 模块代替 `threading` 模块，少量改动即可。
不过，`multiprocessing` 模块还能解决协作进程遇到的最大挑战：在进程之间传递数据。

<span id="扩展阅读"></span>

### 扩展阅读

- [The Future Is Soon!](http://www.pyvideo.org/video/480/pyconau-2010--the-future-is-soon), `concurrent.futures` 包的贡献者 Brian Quinlan 对 `futures` 包的介绍
- [PEP 3148—futures—execute computations asynchronously](https://www.python.org/dev/peps/pep-3148/), 库的正式介绍文件.
- [Parallel Programming with Python 中文版](https://github.com/Voidly/Parallel-Programming-with-Python), 介绍了`concurrent.futures`、`threading` 和 `multiprocessing` 库 的并发编程.
- [python3 cook book](https://github.com/yidao620c/python3-cookbook), "11.12 理解事件驱动型 I/O", "12.7 创建线程池", "12.8 实现简单的并行编程" 等等均可理解并行编程.
- [https://www.python.org/dev/peps/pep-0371/](https://www.python.org/dev/peps/pep-0371/), `multiprocessing` 模块官方文档.
- [Apache Spark](https://spark.apache.org/)，分布式计算引擎 ， 对于 CPU 密集型和数据密集型并行处理的新工具。 [examples](https://spark.apache.org/examples.html)
- [lelo 库](https://pypi.python.org/pypi/lelo) 和 [python-parallelize 库](https://github.com/npryce/python-parallelize), 它们使用多个进程处理并行任务。
    - `lelo` 包定义了一个 `@parallel` 装饰器，可以应用到任何函数上，把函数变成非阻塞：调用被装饰的函数时，函数在一个新进程中执行。
    -  `python-parallelize` 包提供了一个 `@parallelize` 生成器，能把 `for` 循环分配给多个 `CPU` 执行。这两个包在内部都使用了 `multiprocessing` 模块。
