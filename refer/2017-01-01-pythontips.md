---
nav: refer
layout: post
title: "python - 小技巧"
author: "wangchao"
tags:
  - python
  - '技巧'
  - tips
  - '黑科技'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})
/[{{ site.nav.refer.name }}]({% link refer/index.md %})
/{{ page.title }}

### 2017-03-02 ____future____ 模块

参考官网:[future模块](https://docs.python.org/2/library/__future__.html)

功能模块 | 可选择导入版本 | 强制的版本 | PEP
-------| ------------ | --------|----
nested_scopes |  2.1.0b1 | 2.2 | [PEP 227](https://www.python.org/dev/peps/pep-0227): Statically Nested Scopes
generators | 2.2.0a1 | 2.3 | [PEP 255](https://www.python.org/dev/peps/pep-0255): Simple Generators
`division`   | 2.2.0a2 | 3.0 | [PEP 238](https://www.python.org/dev/peps/pep-0238): Changing the Division Operator
`absolute_import` | 2.5.0a1 | 3.0 | [PEP 328](https://www.python.org/dev/peps/pep-0328): Imports: Multi-Line and Absolute/Relative
with_statement | 2.5.0a1 | 2.6 | [PEP 343](https://www.python.org/dev/peps/pep-0343): The “with” Statement
`print_function` | 2.6.0a2 | 3.0 | [PEP 3105](https://www.python.org/dev/peps/pep-3105): Make print a function
`unicode_literals`  |  2.6.0a2 | 3.0 | [PEP 3112](https://www.python.org/dev/peps/pep-3112): Bytes literals in Python 3000

- division 整除模块
- absolute_import 绝对倒入
- print_function print函数
- unicode_literals Unicode编码

如何使python2代码过渡到python3 : [指南](https://docs.python.org/3/howto/pyporting.html) 和 [python-future](http://python-future.org/imports.html)

### collections

__defaultdict:__

```python
# 问题：
>>> some_dict = {}
>>> some_dict['colours']['favourite'] = "yellow"
# 异常输出：KeyError: 'colours'

#解决⽅案：
>>> import collections
>>> tree = lambda: collections.defaultdict(tree)
>>> some_dict = tree()
>>> some_dict['colours']['favourite'] = "yellow"

# 运⾏正常
# 你可以⽤json.dumps打印出some_dict，例如：
>>> import json
>>> print(json.dumps(some_dict))
## 输出: {"colours": {"favourite": "yellow"}}
```

__counter:__

```python
from collections import Counter
colours = (
('Yasoob', 'Yellow'),
('Ali', 'Blue'),
('Arham', 'Green'),
('Ali', 'Black'),
('Yasoob', 'Red'),
('Ahmed', 'Silver'),
)
favs = Counter(name for name, colour in colours)
print(favs)
# 输出
# {"Arham": 1, "Yasoob": 2, "Ahmed": 1, "Ali": 2}
```

__deque:__

deque提供了⼀个双端队列，你可以从头/尾两端添加或删除元素。要想使⽤它，⾸先我们
要从collections中导⼊deque模块：

```python
from collections import deque
d = deque()
d.append('1')
d.append('2')
d.append('3')
print(len(d))
print(d[0])
# 也可以限制这个列表的⼤⼩，当超出你设定的限制时，数据会从对队列另⼀端被挤出去(pop)
d = deque(maxlen=30)
d = deque([1,2,3,4,5])
d.extendleft([0])
d.extend([6,7,8])
print(d)
```

__一行式:__

__简易Web Server:__ 通过⽹络快速共享⽂件？进⼊到你要共享⽂件的⽬录下并在命令⾏中运⾏下⾯的代码：

```python
# Python 2
python -m SimpleHTTPServer
# Python 3
python -m http.server
```

__漂亮的打印:__ 在Python REPL漂亮的打印出列表和字典。这⾥是相关的代码：

```python
from pprint import pprint

my_dict = {'name': 'Yasoob', 'age': 'undefined', 'personality':'what'}

pprint(my_dict)
# 从⽂件打印出json数据:
>>> cat file.json | python -m json.tool
```

__脚本性能分析:__ 定位脚本中的性能瓶颈时.

```python
>>> python -m cProfile my_script.py
备注：cProfile是⼀个⽐profile更快的实现，因为它是⽤c写的
```

__CSV转换为json:__ 通过使⽤itertools包中的itertools.chain.from_iterable轻松快速的辗
平⼀个列表

```python
import itertools

a_list = [[1, 2], [3, 4], [5, 6]]
print(list(itertools.chain.from_iterable(a_list)))
# Output: [1, 2, 3, 4, 5, 6]
# or
print(list(itertools.chain(*a_list)))
# Output: [1, 2, 3, 4, 5, 6]
```

__⼀⾏式的构造器:__  避免类初始化时⼤量重复的赋值语句 

```python
class A(object):
    def __init__(self, a, b, c, d, e, f):
        self.__dict__.update({k: v for k, v in locals().items()})
```

更多参考:[官网](https://wiki.python.org/moin/Powerful%20Python%20One-Liners)

__for-else语句:__ else从句会在循环正常结束时执⾏

适用场景:

- 第⼀个是当⼀个元素被找到，break被触发。
- 第⼆个场景是循环结束。

```python
# 寻找因子
for n in range(2, 10):
    for x in range(2, n):
        if n % x == 0:
            print( n, 'equals', x, '*', n/x)
            break
    else:
        # loop fell through without finding a factor
        print(n, 'is a prime number')
# 输出
(2, 'is a prime number')
(3, 'is a prime number')
(4, 'equals', 2, '*', 2)
(5, 'is a prime number')
(6, 'equals', 2, '*', 3)
(7, 'is a prime number')
(8, 'equals', 2, '*', 4)
(9, 'equals', 3, '*', 3)
```







