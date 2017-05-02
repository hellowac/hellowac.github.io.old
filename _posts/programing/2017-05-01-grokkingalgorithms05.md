---
nav: blog
layout: post
title: "算法图解 - 散列"
author: "wangchao"
tags:
  - python
  - '算法'
  - '散列'
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

**散列函数**

无论你给它什么数据，它都返回一个数字。

数字的要求：

- 它必须是一致的。例如，假设你输入apple时得到的是4，那么每次输入apple时，得到的都必须为4。如果不是这样，散列表将毫无用处。
- 它应将不同的输入映射到不同的数字。例如，如果一个散列函数不管输入是什么都返回1，它就不是好的散列函数。最理想的情况是，将不同的输入映射到不同的数字。

用处：

- 散列函数总是将同样的输入映射到相同的索引。
- 散列函数将不同的输入映射到不同的索引。
- 散列函数知道数组有多大，只返回有效的索引。如果数组包含5个元素，散列函数就不会返回无效索引100。

散列表：使用散列函数和数组创建；使用散列函数来确定元素的存储位置。【 也被称为： 散列映射、映射、字典和关联数组。】

**练习**

对于同样的输入，散列表必须返回同样的输出，这一点很重要。如果不是这样的，就无法找到在散列表中添加的元素！

请问下面哪些散列函数是一致的？

- f(x) = 1   # 无论输入是什么，都返回1
  - 不一致。
- f(x) = rand()  # 每次都返回一个随机数
  - 不一致
- f(x) = next_empty_slot() # 返回散列表中下一个空位置的索引
  - 不一致
- f(x) = len(x) # 将字符串的长度用作索引
  - 一致,但有重复。

**散列表适合用于**：

- 模拟映射关系；
- 防止重复；
- 缓存/记住数据，以免服务器再通过处理来生成它们。

**教训：**

- `散列函数很重要`。前面的散列函数将所有的键都映射到一个位置，而最理想的情况是，散列函数将键均匀地映射到散列表的不同位置。
- 如果散列表存储的链表很长，散列表的速度将急剧下降。然而，`如果使用的散列函数很好`，这些链表就不会很长！

平均情况下，散列表的查找（获取给定索引处的值）速度与数组一样快，而插入和删除速度与链表一样快，因此它兼具两者的优点！但在最糟情况下，散列表的各种操作的速度都很慢。因此，在使用散列表时，避开最糟情况至关重要。为此，需要避免冲突。而要避免冲突，需要有：

- 较低的填装因子；
- 良好的散列函数。

**填装因子：** 散列表包含的元素数 / 位置总数. [一个不错的经验规则是：一旦填装因子大于0.7，就调整散列表的长度。]
**良好的散列函数**： 良好的散列函数让数组中的值呈均匀分布。糟糕的散列函数让值扎堆，导致大量的冲突。

**练习：**

散列函数的结果必须是均匀分布的，这很重要。它们的映射范围必须尽可能大。最糟糕的散列函数莫过于将所有输入都映射到散列表的同一个位置。

假设你有四个处理字符串的散列函数。

A. 不管输入是什么，都返回1。

B. 将字符串的长度用作索引。

C. 将字符串的第一个字符用作索引。即将所有以a打头的字符串都映射到散列表的同一个位置，以此类推。

D. 将每个字符都映射到一个素数：a = 2，b = 3，c = 5，d = 7，e = 11，等等。对于给定的字符串，这个散列函数将其中每个字符对应的素数相加，再计算结果除以散列表长度的余数。例如，如果散列表的长度为10，字符串为bag，则索引为(3 + 2 + 17) % 10 = 22 % 10 = 2。

在下面的每个示例中，上述哪个散列函数可`实现均匀分布`？假设散列表的长度为10。

- 将姓名和电话号码分别作为键和值的电话簿，其中联系人姓名为Esther、Ben、Bob和Dan。
  - A: 1,1,1,1
  - B: 6,3,3,3
  - C: 4,1,1,3
  - D: 0,3,9,9
- 电池尺寸到功率的映射，其中电池尺寸为A、AA、AAA和AAAA。
  - A: 1,1,1,1
  - B: 1,2,3,4
  - C: 0,0,0,0
  - D: 2,5,3,9
- 书名到作者的映射，其中书名分别为Maus、Fun 、Home和Watchmen。
  - A: 1,1,1,1
  - B: 1,2,3,4
  - C: 12,5,7,22 【越界了.】
  - D: 4,9,8,0

可见第四种【D】方法比较好的实现了均匀分布. 关于第四种的我的代码实现，参考底部.

**小结：**

- 可以结合散列函数和数组来创建散列表。
- 冲突很糟糕，你应使用可以最大限度减少冲突的散列函数。
- 散列表的查找、插入和删除速度都非常快。
- 散列表适合用于模拟映射关系。
- 一旦填装因子超过0.7，就该调整散列表的长度。
- 散列表可用于缓存数据（例如，在Web服务器上）。
- 散列表非常适合用于防止重复。

**第四种代码实现:**

```python
import string

def get_primer_number_by_index(index):
    """ 常规获取第index个素数【质数】 """
    arr = []
    number = 2
    is_sushu = False

    while index >= 0:

        # 能整除任意一个就不是.
        for i in arr:
            if not number%i:
                is_sushu=False
                break
        else:
            is_sushu = True

        if is_sushu:
            index -= 1
            arr.append(number)

        number += 1

    return arr[index]


def get_primer_number_by_loop(index, init_number=None, arr=[]):
    """ 递归获取第index个素数【质数】, 这里利用了arr只初始化一次的技巧"""
    number = 2 if not init_number else init_number

    if index == -1 :
        return number
    else:
        while True:
            # 能整除任意一个就不是.
            for i in arr:
                if not number%i:
                    break
            else:
                arr.append(number)
                break  # 退出while循环
            number +=1

        return get_primer_number_by_loop(index-1,init_number=number)  # 传当前素数，免得从0开始计算.


def get_primer_number_index(lowercase):
    """ 获取小写字母对应的质数的下标, 不区分大小写, 从0开始 """
    for i,x in enumerate(string.ascii_lowercase,0):
        if x == lowercase:
            return i


def get_sum_by_name(name):
    """ 计算名称的质数和 """
    primer_dict = {}
    for letter in name:
        lowercase_index = get_primer_number_index(letter.lower())
        primer_number = get_primer_number_by_loop(lowercase_index)

        primer_dict[letter] = primer_number

    # print primer_dict
    print '{keys}:{values}'.format(keys=name, values=','.join(map(lambda x : str(x), primer_dict.values())))
    primer_sum =  sum(primer_dict.values())
    # print primer_sum
    return  primer_sum

def my_hash(name, arr_length=10):
    """ 自定义散列函数. """
    name_primer_sum = get_sum_by_name(name)
    return name_primer_sum%arr_length


if __name__ == '__main__':
    lowercase_index = get_primer_number_index('c')
    # lowercase_index = 2
    # print get_primer_number_by_index(lowercase_index)
    # print get_primer_number_by_loop(lowercase_index)
    # name= 'Esther'
    # get_sum_by_name(name)

    names = ['Esther','Ben','Bob','Dan']
    names = ['A','AA','AAA','AAAA']
    names = ['Maus','Fun','Home','Watchmen']
    for name in names:
        # print 'name:',get_primer_number_index(name[0].lower())  # 第三种计算.
        print '{name} hash index:{hash_index}\n'.format(name=name, hash_index=my_hash(name))
```
