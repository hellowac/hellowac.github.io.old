---
nav: blog
layout: post
title: "算法图解 - 算法简介"
author: "wangchao"
tags:
  - python
  - '算法'
  - '二分查找'
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

- [二分查找](#二分查找)

<span id="二分查找"></span>

### 二分查找

```python
def binary_search(arr, item):
  """ 二分查找算法 """
    low_index = 0
    high_index = len(arr) - 1 # 下标从0开始.

    while low_index <= high_index:
      mid_index = (low_index + high_index) / 2
      guss = arr[mid_index]

      if guss == item:
        return mid_index

      if guss < item:     # 猜小了
        low_index = mid_index + 1
      else:   # 猜大了
        high_index = high_index - 1

    return None

# 练习：
my_list = [1, 3, 5, 7, 9]
print binary_search(my_list, 3)  # 1
print binary_search(my_list, -1)  # None
```

#### 练习：

- 假设有一个包含128个名字的有序列表，你要使用二分查找在其中查找一个名字，请 问最多需要几步才能找到？
  - log 以2 为低 的 128 的对数.
- 上面列表的长度翻倍后，最多需要几步？
  - 加1.

**优缺点：**

- 二分查找的速度更快
- 简单查找算法编写起来更容易，出现bug的可能性更小。

**运行时间表示：**

- **线性时间：** 最多需要猜测的次数与列表长度相同, O(n)
- **对数时间(或log时间):** 二分查找的运行时间, O(log n)

**大O表示法指出了算法有多快**
**大O表示法指出了最糟情况下的运行时间**

**一些常见的大O运行时间：**

- `O(log n)`，也叫对数时间，这样的算法包括二分查找。
- `O(n)`，也叫线性时间，这样的算法包括简单查找。
- `O(n * log n)`，这样的算法包括将介绍的快速排序——一种速度较快的排序算法。
- `O(n*n)`，这样的算法包括将介绍的选择排序——一种速度较慢的排序算法。
- `O(n!)`，这样的算法包括将介绍的旅行商问题的解决方案——一种非常慢的算法。 [阶乘.]

**小总结**：

- 算法的速度指的并非时间，而是操作数的增速。
- 谈论算法的速度时，说的是随着输入的增加，其运行时间将以什么样的速度增加。
- 算法的运行时间用大O表示法表示。
- O(log n)比O(n)快，当需要搜索的元素越多时，前者比后者快得越多。

**小练习:**

- 在电话簿中根据名字查找电话号码。
  - O(log n)
- 在电话簿中根据电话号码找人。（提示：你必须查找整个电话簿。）
  - O(n)
- 阅读电话簿中每个人的电话号码。
  - O(n2)
- 阅读电话簿中姓名以A打头的人的电话号码。这个问题比较棘手，它涉及第4章的概 念。答案可能让你感到惊讶！
  - O(n)
