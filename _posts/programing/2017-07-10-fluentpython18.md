---
nav: blog
layout: post
title: "流畅的python - Asyncio"
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

- [线程与协程对比](#threadwithasyncio)
- [显示进度条](#显示进度条)
- [扩展阅读](#扩展阅读)


<span id="threadwithasyncio"></span>

### 线程与协程对比

**线程版**

```python
# spinner_thread.py

# credits: Adapted from Michele Simionato's
# multiprocessing example in the python-list:
# https://mail.python.org/pipermail/python-list/2009-February/538048.html

# BEGIN SPINNER_THREAD
import threading
import itertools
import time
import sys


def spin(msg, done):  # 这个函数会在单独的线程中运行
    write, flush = sys.stdout.write, sys.stdout.flush
    for char in itertools.cycle('|/-\\'):  # 反复不断地生成元素，死循环
        status = char + ' ' + msg
        write(status)
        flush()  # 立即显示
        write('\x08' * len(status))  # 使用退格符（\x08）把光标移回去。
        if done.wait(.1):  # 如果有该事件，则退出
            break
    write(' ' * len(status) + '\x08' * len(status))  # 使用空格清除状态消息，把光标移回开头。


def slow_function():  #  假设这是耗时的计算。
    # pretend waiting a long time for I/O
    time.sleep(3)  # 调用 sleep 函数会阻塞主线程，不过一定要这么做，以便释放 GIL，创建从属线程。
    return 42


def supervisor():  # 设置子线程
    done = threading.Event()  # 线程事件.
    spinner = threading.Thread(target=spin,
                               args=('thinking!', done))
    print('spinner object:', spinner)  # 显示子线程对象。
    spinner.start()  # 启动子线程。
    result = slow_function()  # 阻塞主线程，子线程以动画形式显示旋转指针。
    done.set()  # 设置事件，阻止spin中的for循环
    spinner.join()  # 等待 spinner 线程结束。
    return result


def main():
    result = supervisor()  # 运行 supervisor 函数。
    print('Answer:', result)


if __name__ == '__main__':
    main()
# END SPINNER_THREAD
```

**协程版**

```python
# spinner_asyncio.py

# credits: Example by Luciano Ramalho inspired by
# Michele Simionato's multiprocessing example in the python-list:
# https://mail.python.org/pipermail/python-list/2009-February/538048.html

# BEGIN SPINNER_ASYNCIO
import asyncio
import itertools
import sys


@asyncio.coroutine  # 打算交给 asyncio 处理的协程要使用 @asyncio.coroutine 装饰。这样能在一众普通的函数中把协程凸显出来，也有助于调试
def spin(msg):  # 不需要线程中的事件参数
    write, flush = sys.stdout.write, sys.stdout.flush
    for char in itertools.cycle('|/-\\'):
        status = char + ' ' + msg
        write(status)
        flush()
        write('\x08' * len(status))
        try:
            yield from asyncio.sleep(.1)  # 这样的休眠不会阻塞事件循环。
        except asyncio.CancelledError:  # 发出了取消请求，因此退出循环。
            break
    write(' ' * len(status) + '\x08' * len(status))


@asyncio.coroutine
def slow_function():  # 用休眠假装进行 I/O 操作时，使用 yield from 继续执行事件循环。
    # pretend waiting a long time for I/O
    yield from asyncio.sleep(3)  # 把控制权交给主循环，在休眠结束后恢复这个协程。
    return 42


@asyncio.coroutine
def supervisor():  # supervisor 函数也是协程，因此可以使用 yield from 驱动 slow_function 函数。
    spinner = asyncio.async(spin('thinking!'))  # 排定 spin 协程的运行时间，使用一个 Task 对象包装 spin 协程，并立即返回。
    print('spinner object:', spinner)  # 显示 Task 对象，类似于 <Task pending coro=<spin() running at spinner_ asyncio.py:12>>。
    result = yield from slow_function()  # 驱动 slow_function() 函数
    spinner.cancel()  # Task 对象可以取消；取消后会在协程当前暂停的 yield 处抛出 asyncio.CancelledError 异常。协程可以捕获这个异常，也可以延迟取消，甚至拒绝取消。
    return result


def main():
    loop = asyncio.get_event_loop()  # 获取事件循环的引用。
    result = loop.run_until_complete(supervisor())  # 驱动 supervisor 协程，让它运行完毕；这个协程的返回值是这次调用的返回值。
    loop.close()
    print('Answer:', result)


if __name__ == '__main__':
    main()
# END SPINNER_ASYNCIO
```

两者的区别:

- `asyncio.Task` 对象差不多与 `threading.Thread` 对象等效。“Task 对象像是实现协作式多任务的库（例如 gevent）中的绿色线程（green thread）”。
- `Task` 对象用于驱动协程，`Thread` 对象用于调用可调用的对象。
- `Task` 对象不由自己动手实例化，而是通过把协程传给 `asyncio.async(...)` 函数或 `loop.create_task(...)` 方法获取。
- 获取的 `Task` 对象已经排定了运行时间（例如，由 `asyncio.async` 函数排定）；`Thread` 实例则必须调用 `start` 方法，明确告知让它运行。
- 在线程版 `supervisor` 函数中，`slow_function` 函数是普通的函数，直接由线程调用。在异步版 `supervisor` 函数中，`slow_function` 函数是协程，由 `yield from` 驱动。
- 没有 `API` 能从外部终止线程，因为线程随时可能被中断，导致系统处于无效状态。如果想终止任务，可以使用 `Task.cancel()` 实例方法，在协程内部抛出 `CancelledError` 异常。协程可以在暂停的 `yield` 处捕获这个异常，处理终止请求。
- `supervisor` 协程必须在 `main` 函数中由 `loop.run_until_complete` 方法执行。


后面的内容待以后补充吧..
