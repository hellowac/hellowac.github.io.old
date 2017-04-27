---
nav: blog
layout: post
title: "python - 小技巧"
author: "wangchao"
tags:
  - python
  - '算法'
  - '选择排序'
  - '图解算法'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[PyTech](http://hyry.dip.jp/tech/forum/index)

**将id转换为对象**

```python
def id_to_object(id_):
   import ctypes
   return ctypes.cast(ctypes.c_void_p(id_), ctypes.py_object).value
```

**平坦化列表，【将多维数组转化为一维】**

```python
import itertools

nested_list = [[1,2],[3,4]]

print list(itertools.chain.from_iterable(nested_list))

# [1, 2, 3, 4]
```

**指定enumerate起始位置,python2.6以后.**

```python
>>> list(enumerate("abc", 2))
[(2, 'a'), (3, 'b'), (4, 'c')]
```

**Queue获取对象，但不取出.**

```python
# Queue没有提供获取队列中的下一个元素，但是不从队列中取出的方法。可通过访问Queue对象的queue属性，获得保存元素的deque对象，从而获得队列中的下一个元素。多线程时须注意排他处理。下面的程序输出PriorityQueue对象的下一个元素：

from Queue import PriorityQueue
a = PriorityQueue()
a.put((10, "a"))
a.put((4, "b"))
a.put((3,"c"))

print a.queue[0]    # (3, "c")
print a.get()       # [(4, "b"),(10, "a")]
print a.queue[0]    # (4, "b")
```
