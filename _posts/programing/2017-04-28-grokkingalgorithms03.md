---
nav: blog
layout: post
title: "算法图解 - 递归"
author: "wangchao"
tags:
  - python
  - '算法'
  - '递归'
  - '图解算法'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[算法图解](https://book.douban.com/subject/26979890/)


**递归**

递归只是让解决方案更清晰，并没有性能上的优势。

实际上，在有些情况下，使用循环的性能更好。

Leigh Caldwell在Stack Overflow上说的一句话：“如果使用循环，程序的性能可能更高；如果使用递归，程序可能更容易理解。如何选择要看什么对你来说更重要。”

每个递归函数都有两部分：`基线条件`（base case）和`递归条件`（recursive case）。

- **递归条件**:指函数调用自己
- **基线条件**:指函数不再调用自己，从而避免形成无限循环。

```python
def countdown(i):
    print(i)
  if i <= 0:  # 基线条件
    return
  else:  # 递归条件
    countdown(i-1)

# 调用:
countdown(4)

4
3
2
1
0
```

**栈:**

- **调用栈（call stack）:** 用于存储多个函数的变量。
  - 调用另一个函数时，当前函数暂停并处于未完成状态。该函数的所有变量的值都还在内存中。执行完调用函数后，回到当前函数，并从离开的地方开始接着往下执行。

```python
# 阶乘的递归
def fact(x):
    if x == 1:  # 每个fact调用都有自己的x变量。在一个函数调用中不能访问另一个的x变量。
        return 1
    else:
        return x * fact(x-1)
```

用栈虽然很方便，但是也要付出代价：存储详尽的信息可能占用大量的内存。每个函数调用都要占用一定的内存，如果栈很高，就意味着计算机存储了大量函数调用的信息。在这种情况下，有两种选择：

- 重新编写代码，转而使用循环。

- 使用[尾递归](http://www.cnblogs.com/Anker/archive/2013/03/04/2943498.html)。这是一个高级递归主题，不在当前的讨论范围内。另外，并非所有的语言都支持尾递归。

**小结：**

- 递归指的是调用自己的函数。
- 每个递归函数都有两个条件：基线条件和递归条件。
- 栈有两种操作：压入和弹出。
- 所有函数调用都进入调用栈。
- 调用栈可能很长，这将占用大量的内存。
