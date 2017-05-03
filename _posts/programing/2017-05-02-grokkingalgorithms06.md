---
nav: blog
layout: post
title: "算法图解 - 广度优先搜索"
author: "wangchao"
tags:
  - python
  - '算法'
  - '广度优先搜索'
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

**图**

- 图由节点和边组成。一个节点可能与众多节点直接相连，这些节点被称为邻居。
- 图用于模拟不同的东西是如何相连的。


**图算法：广度优先搜索**

能够找出两样东西之间的最短距离，不过最短距离的含义有很多！他可以：

- 编写国际跳棋AI，计算最少走多少步就可获胜；
- 编写拼写检查器，计算最少编辑多少个地方就可将错拼的单词改成正确的单词，如将READED改为READER需要编辑一个地方；
- 根据你的人际关系网络找到关系最近的医生。
- 寻找最短路径问题。

该算法可以回答的问题：

- 第一类问题：从节点A出发，有前往节点B的路径吗？
- 第二类问题：从节点A出发，前往节点B的哪条路径最短？

**查找最短路径**

一度关系胜过二度关系，二度关系胜过三度关系，以此类推。

因此，应先在一度关系中搜索，确定其中没有后，才在二度关系中搜索。广度优先搜索就是这样做的！

在广度优先搜索的执行过程中，搜索范围从起点开始逐渐向外延伸，即先检查一度关系，再检查二度关系。【一度关系在二度关系之前加入查找名单。】

**注意:** 只有按添加顺序查找时，才能实现这样的目的。 【找到的是关系最近的】

**队列：**

队列是一种先进先出（First In First Out，FIFO）的数据结构，而栈是一种后进先出（Last In First Out，LIFO）的数据结构。

队列只支持两种操作：入队和出队。

**实现图：**

每个节点都与邻近节点相连，如果表示类似于“你→Bob”这样的关系呢？知道的散列表结构让能够表示这种关系。

```python
# 表示 you 指向的关系
graph = {}
graph["you"] = ["alice", "bob", "claire"]  # 包含了“你”的所有邻居。
graph["bob"] = ["anuj", "peggy"]           # 包含了“bob”的所有邻居。
graph["alice"] = ["peggy"]                 # 包含了“alice”的所有邻居。
graph["claire"] = ["thom", "jonny"]        # 包含了“claire”的所有邻居。
graph["anuj"] = []
graph["peggy"] = []
graph["thom"] = []
graph["jonny"] = []
```

Anuj、Peggy、Thom和Jonny都没有邻居，这是因为虽然有指向他们的箭头，但没有从他们出发指向其他人的箭头。这被称为有向图（directed graph），其中的关系是单向的。

因此，Anuj是Bob的邻居，但Bob不是Anuj的邻居。

无向图（undirected graph）没有箭头，直接相连的节点互为邻居。

**实现算法**

```python
# 首先，创建一个队列。在Python中，可使用函数deque来创建一个双端队列。
from collections import deque
search_queue = deque()  # 创建一个队列
search_queue += graph["you"]  # 将你的邻居都加入到这个搜索队列中

# 搜索
def search(name):
    search_queue = deque()
    search_queue += graph[name]
    searched = []  # 这个数组用于记录检查过的人，避免陷入无限循环
    while search_queue:  # 只要队列不为空
        person = search_queue.popleft()  # 就取出其中的第一个人
        if person not in searched:     # 仅当这个人没检查过时才检查
            if person_is_seller(person):  # 检查这个人是否是符合搜索条件
                print person + " is a mango seller!"  # 符合搜索条件
                return True
            else:
                search_queue += graph[person]  # 不符合搜索条件，将这个人的朋友都加入搜索队列
                searched.append(person)    # 将这个人标记为检查过
    return False  # 如果到达了这里，就说明队列中没人符合条件

def person_is_seller(name):
    return name.endswith('m')  # 仅仅为了说明.

search("you")
```

**运行时间：**

- 如果在整个人际关系网中搜索符合条件的节点，就意味着将沿每条边前行（记住，边是从一个节点到另一节点的箭头或连接），因此运行时间至少为O(边数)。
- 还使用了一个队列，其中包含要检查的每个节点。将一个节点添加到队列需要的时间是固定的，即为O(1)，因此对每个节点都这样做需要的总时间为O(节点数)。所以，广度优先搜索的运行时间为O(节点数 + 边数)，这通常写作O(V + E)，其中V 为顶点（vertice）数，E 为边数。

**拓扑排序**

如果任务A依赖于任务B，在列表中任务A就必须在任务B后面。使用它可根据图创建一个有序列表。

这种图被称为树。树是一种特殊的图，其中没有往后指的边。

**小结：**

- 广度优先搜索指出是否有从A到B的路径。
- 如果有，广度优先搜索将找出最短路径。
- 面临类似于寻找最短路径的问题时，可尝试使用图来建立模型，再使用广度优先搜索来解决问题。
- 有向图中的边为箭头，箭头的方向指定了关系的方向，例如，rama→adit表示rama欠adit钱。
- 无向图中的边不带箭头，其中的关系是双向的，例如，ross - rachel表示“ross与rachel约会，而rachel也与ross约会”。
- 队列是先进先出（FIFO）的。
- 栈是后进先出（LIFO）的。
- 需要按加入顺序检查搜索列表中的人，否则找到的就不是最短路径，因此`搜索列表必须是队列`。
- 对于检查过的节点，务必不要再去检查，否则可能导致无限循环。
