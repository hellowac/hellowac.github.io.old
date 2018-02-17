---
nav: blog
layout: post
title: "算法图解 - 快速排序"
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


**分而治之(divide and conquer，D&C):** 一种著名的递归式问题解决方法。

尝试使用掌握的各种问题解决方法来找出解决方案。

使用D&C解决问题的过程包括两个步骤:

- 找出基线条件，这种条件必须尽可能简单。
- 不断将问题分解（或者说缩小规模），直到符合基线条件。

D&C并非可用于解决问题的算法，而是一种解决问题的思路。 如， 土地分块问题，

**问题：**

给定一个数字数组。要将这些数字相加，并返回结果。使用循环很容易完成任务。但使用递归呢？

- **第一步**：找出基线条件。最简单的数组什么样呢？数组不包含任何元素或只包含一个元素。
- **第二步**：找出递归条件。每次递归调用都必须离空数组更近一步。

提示：编写涉及数组的递归函数时，基线条件通常是数组为空或只包含一个元素。陷入困境时，请检查基线条件是不是这样的。

```python
# 我的实现：
def my_sum(arr):
    if not arr:  # 基线条件
        return 0
    else:  # 递归条件
        return arr.pop() + my_sum(arr)

print my_sum([1,2,3,4,5,6])
21
```

**函数式编程一瞥:**

既然使用循环可轻松地完成任务，为何还要使用递归方式呢？看看函数式编程就明白了！
诸如Haskell等函数式编程语言没有循环，因此你只能使用递归来编写这样的函数。
如果对递归有深入的认识，函数式编程语言学习起来将更容易。如 使用 Haskell 编写sum函数时：

```Haskell
sum [] = 0  # 基线条件
sum (x:xs) = x + (sum xs)  # 递归条件
```

这就像有函数的两个定义。符合基线条件时运行第一个定义，符合递归条件时运行第二个定义。

**练习：**

- **编写一个递归函数来计算列表包含的元素数。**

```python
def my_counter(arr):
    if not arr:
        return 0
    else:
        arr.pop()
        return 1 + my_counter(arr)

print my_counter([1,2,3,4,5,6])

6
```

- **找出列表中最大的数字。**

```python
def my_max(arr):
    if not arr:
        return 0
    else:
        max_num = arr.pop()
        next_max_num = my_max(arr)
        return max_num if max_num > next_max_num else next_max_num

print my_max([1,2,100,3,4,5,6,1002,4])
```

- **还记得第1章介绍的二分查找吗？它也是一种分而治之算法。你能找出二分查找算法的基线条件和递归条件吗？**
  - 基线条件：最左下标大于最右下标 或 匹配到.
  - 递归条件：最左下标小于最右下标 并且 未匹配到.

```python
# 第一版，遍历查找... 不满意.
def my_found(item, arr, index=0):

    out = index == (len(arr)-1)

    if not out and item == arr[index]:
        return index
    elif not out:
        return my_found(item, arr, index+1)
    else:
        return None

arr = [1,2,100,3,4,5,6,1002,4]

print my_found(1002, arr)  # 7

# 第二版, 递归二分查找, 有缺陷，
def my_found(item, arr, high_index, low_index=0):

    mid_index = (low_index+high_index)/2
    mid_item = arr[mid_index]

    if low_index > high_index :
        return None
    elif item > mid_item:
        return my_found(item, arr, high_index, mid_index+1)
    elif item < mid_item:
        return my_found(item, arr, mid_item-1, low_index)
    else:
        return mid_index

arr = [1,2,3,4,5,6,7,8,9]

print my_found(1, arr, len(arr)-1)  # 0
print my_found(9, arr, len(arr)-1)  # 8
# print my_found(0, arr, len(arr)-1)  # 报最大深度错误. 可将条件改为 low_index >= high_index , 可解决. 为None.
# 但同时 最右边界值时就会匹配不到. 上一个即变为None.应该为8的. 如何解决出现的新问题？
print my_found(10, arr, len(arr)-1)  # None

# 第三版, 递归型的二分查找, 解决完bug
def my_found(item, arr, high_index, low_index=0):

    mid_index = (low_index+high_index)/2
    mid_item = arr[mid_index]

    if item > mid_item and mid_index+1 <= high_index:
        return my_found(item, arr, high_index, mid_index+1)
    elif item < mid_item and mid_index-1 >= low_index :
        return my_found(item, arr, mid_item-1, low_index)
    elif item == mid_item:
        return mid_index
    else:
        return None

arr = [1,2,3,4,5,6,7,8,9]

print my_found(1, arr, len(arr)-1)  # 0
print my_found(9, arr, len(arr)-1)  # 8
print my_found(0, arr, len(arr)-1)  # None
print my_found(10, arr, len(arr)-1)  # None
```

**快速排序**

快速排序是一种常用的排序算法，比选择排序快得多。例如，C语言标准库中的函数qsort实现的就是快速排序。快速排序也使用了D&C。

对排序算法来说，最简单的数组什么样呢？就是根本不需要排序的数组。 如：[] 和 \[1\](只包含一个元素的数组)

因此，基线条件为`数组为空`或`只包含一个元素`。在这种情况下，只需原样返回数组——根本就不用排序。

```python
def quicksort(array):
    if len(array) < 2:
        return array
```

当包含多个元素时，只需 `分区`: 选择一个基准值，将 大于它的元素 排如一个数组， 小于它的元素排入一个数组， 然后 `less_arr + 基准值 + great_arr`. 如：

```python
quicksort([15, 10]) + [33] + quicksort([80, 34])
> [10, 15, 33, 35, 80]
```

步骤如下：

- 选择基准值。
- 将数组分成两个子数组：小于基准值的元素和大于基准值的元素。
- 对这两个子数组进行快速排序。

快速排序的python代码:

```python
def my_quicksort(arr):
    """ 快速排序 """

    if len(arr) < 2:
        return arr
    else:
        pivot = arr[0]  # 基准值

        less_arr = [i for i in arr[1:] if i <= pivot]
        great_arr = [i for i in arr[1:] if i > pivot ]

        return my_quicksort(less_arr) + [pivot] + my_quicksort(great_arr)

print my_quicksort([2,10,3,20,32,12,44,98,232])

[2, 3, 10, 12, 20, 32, 44, 98, 232]
```

最后想说,如果所有算法实现都像python代码这么简洁。容易理解。学习算法大概也不吃力了吧.

**再谈大O表示法**

快速排序的独特之处在于，其速度取决于选择的基准值。

还有一种名为合并排序（merge sort）的排序算法，其运行时间为O(n log n)，比选择排序快得多！快速排序的情况比较棘手，在最糟情况下，其运行时间为O(n2)。

快速排序的最糟情况: 数组是经过排序的，那样调用栈的高度将是 O(n)
快速排序的最佳情况: 数组是随机的，那样调用栈的高度是O(log n) [以2为低的n对数] ,【每次调用栈都将数组一分为长度相等的两个子数组. 但每个调用栈都有N个元素，所以需要的时间为O(n), 再乘以高度O(log n) , 那么时间复杂度将是 O(n) x O(log n) = O(n * log n)】

**合并排序:**

即每次都取数组的中间值作为基准值，保证调用栈的高度为O(log n), 那么复杂度则为平均情况: O(n) x O(log n) = O(n * log n) = O(NlogN)

```python
#
def my_quicksort(arr):
    """ 合并排序 """

    if len(arr) < 2:
        return arr
    else:

        pivot = arr.pop(len(arr)/2)  # 基准值, 确保近似的平分数组

        less_arr = [i for i in arr if i <= pivot]
        great_arr = [i for i in arr if i > pivot ]

        return my_unique_quicksort(less_arr) + [pivot] + my_unique_quicksort(great_arr)

print my_unique_quicksort([2,10,3,20,32,12,44,98,232])
```

貌似和快速排序只差了一行代码. ^\_^

**练习**

使用大O表示法时，下面各种操作都需要多长时间？

- 打印数组中每个元素的值。
  - O(n)
- 将数组中每个元素的值都乘以2。
  - O(n)
- 只将数组中第一个元素的值乘以2。
  - O(1)
- 根据数组包含的元素创建一个乘法表，即如果数组为[2, 3, 7, 8, 10]，首先将每个元素 都乘以2，再将每个元素都乘以3，然后将每个元素都乘以7，以此类推。
  - O(n*n) 【n的平方】大概如下所示：

```
2 x 2 ,                  3 x 2 ,                  7 x 2 ,                    8 x 2 ,                   10 x 2
2 x 2 x 3 ,              3 x 2 x 3 ,              7 x 2 x 3  ,               8 x 2 x 3  ,              10 x 2 x 3
2 x 2 x 3 x 7 ,          3 x 2 x 3 x 7 ,          7  x 2 x 3 x 7  ,          8  x 2 x 3 x 7 ,          10 x 2  x 3 x 7
2 x 2 x 3 x 7 x 8 ,      3 x 2 x 3 x 7  x 8 ,     7 x 2  x 3  x 7 x 8 ,      8 x 2  x 3  x 7 x 8 ,     10 x 2 x 3 x 7  x 8
2 x 2 x 3 x 7 x 8 x 10 , 3 x 2 x 3 x 7 x 8 x 10 , 7  x 2 x 3 x 7 x 8  x 10 , 8 x 2 x 3  x 7 x 8 x 10 , 10 x 2 x 3 x 7 x 8 x 10
```

**小结**

- D&C将问题逐步分解。使用D&C处理列表时，基线条件很可能是空数组或只包含一个元素的数组。
- 实现快速排序时，请随机地选择用作基准值的元素。快速排序的平均运行时间为O(n log n)。
- 大O表示法中的常量有时候事关重大，这就是快速排序比合并排序快的原因所在。
- 比较简单查找和二分查找时，常量几乎无关紧要，因为列表很长时，O(log n)的速度比O(n)快得多。
