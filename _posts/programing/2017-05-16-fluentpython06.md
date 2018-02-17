---
nav: blog
layout: post
title: "流畅的python - 函数与设计模式"
author: "wangchao"
tags:
  - python
  - 'function'
  - '函数'
  - '内置函数'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

**策略模式**

- **上下文：** 把一些计算委托给实现不同算法的可互换组件，它提供服务。
- **策略：** 实现算法的不同组件的共同接口。
- **具体策略**： “策略”的具体子类。

```python
# 策略模式，计算订单折扣：
# 规则：同时只能享受一个折扣.
# 1. 有 1000 或以上积分的顾客，每个订单享 5% 折扣。
# 2. 同一订单中，单个商品的数量达到 20 个或以上，享 10% 折扣。
# 3. 订单中的不同商品达到 10 个或以上，享 7% 折扣。

from abc import ABC, abstractmethod
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')

class LineItem:

    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.price * self.quantity

class Order:  # 上下文

    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total

    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount

    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

class Promotion(ABC) : # 策略：抽象基类
    """  定义为抽象基类 """

    # 明确表明所用的模式
    @abstractmethod
    def discount(self, order):
        """返回折扣金额（正值）"""


class FidelityPromo(Promotion):  # 第一个具体策略
    """为积分为1000或以上的顾客提供5%折扣"""

    def discount(self, order):
        return order.total() * .05 if order.customer.fidelity >= 1000 else 0


class BulkItemPromo(Promotion):  # 第二个具体策略
    """单个商品为20个或以上时提供10%折扣"""

    def discount(self, order):
        discount = 0
        for item in order.cart:
            if item.quantity >= 20:
                discount += item.total() * .1
        return discount


class LargeOrderPromo(Promotion):  # 第三个具体策略
    """订单中的不同商品达到10个或以上时提供7%折扣"""

    def discount(self, order):
        distinct_items = {item.product for item in order.cart}
        if len(distinct_items) >= 10:
            return order.total() * .07
        return 0

# 应用：
In [5]: joe = Customer('John Doe', 0)  # joe 的积分是 0。

In [6]: ann = Customer('Ann Smith', 1100) # ann 的积分是 1100。

In [7]: cart = [LineItem('banana',4,.5),  # 三个商品的购物车.
   ...:         LineItem('apple',10,1.5),
   ...:         LineItem('watermellon',5,5.0)]

In [8]: Order(joe,cart,FidelityPromo())  # 积分没有1000，不提供折扣.
Out[8]: <Order total: 42.00 due: 42.00>

In [9]: Order(ann,cart,FidelityPromo())  # 积分有1000，提供折扣.
Out[9]: <Order total: 42.00 due: 39.90>

In [10]: banana_cart = [LineItem('banana',30,.5), # 30斤香蕉，10斤苹果。
    ...:                LineItem('apple',10,1.5)]

In [11]: Order(joe,banana_cart,BulkItemPromo())  # 商品超过20个，为joe提供折扣
Out[11]: <Order total: 30.00 due: 28.50>

In [12]: long_order = [LineItem(str(item_code),1,1.0) for item_code in range(10)] # 是个不同的商品，提供折扣.

In [13]: Order(joe,long_order,LargeOrderPromo())  # 是个不同的商品，提供折扣.
Out[13]: <Order total: 10.00 due: 9.30>

In [14]: Order(joe,cart,LargeOrderPromo())
Out[14]: <Order total: 42.00 due: 42.00>
```

使用函数实现, 没必要在新建订单时实例化新的促销对象，函数拿来即用。

```python
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')


class LineItem:

    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.price * self.quantity


class Order:  # 上下文

    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total

    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion(self)  # 计算折扣只需调用 self.promotion(), 而不是上面的 self.promotion.discount()
        return self.total() - discount

    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

# 没有抽象类。

def fidelity_promo(order):  #  各个策略都是函数。
    """为积分为1000或以上的顾客提供5%折扣"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

def bulk_item_promo(order):
    """单个商品为20个或以上时提供10%折扣"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

def large_order_promo(order):
    """订单中的不同商品达到10个或以上时提供7%折扣"""
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0
```

**“策略对象通常是很好的享元（`flyweight`）。”**
  - **享元：** “享元是可共享的对象，可以同时在多个上下文中使用。”
  - 共享是推荐的做法，这样不必在每个新的上下文中使用相同的策略时不断新建具体策略对象，从而减少消耗。

- **元策略：** 选择最佳的策略。

```python
promos = [fidelity_promo, bulk_item_promo, large_order_promo]  # 列出实现的各个策略。

def best_promo(order):  # 传入 order参数。
    """选择可用的最佳折扣
    """
    return max(promo(order) for promo in promos)  # 使用生成器表达式把 order 传给 promos 列表中的各个函数，返回折扣额度最大的那个函数。
```

- **找出模块中的全部策略**
  - **globals():**  返回一个字典，表示当前的全局符号表。这个符号表始终针对当前模块（对函数或方法来说，是指定义它们的模块，而不是调用它们的模块）。

```python
promos = [globals()[name] for name in globals()  # 迭代 globals() 返回字典中的各个 name
            if name.endswith('_promo')  # 只选择以 _promo 结尾的名称。
            and name != 'best_promo']   # 过滤掉 best_promo 自身，防止无限递归。

def best_promo(order):
    """选择可用的最佳折扣
    """
    return max(promo(order) for promo in promos)  # best_promo 内部的代码没有变化。

# 另一种方法：在一个单独的模块中保存所有策略函数，把 best_promo 排除在外。
```

```python
# 利用 inspect 模块 构建策略列表.
# 不管怎么命名策略函数，唯一重要的是，promotions 模块只能包含计算订单折扣的函数.这是对代码的隐性假设。
import inspect

promos = [func for name, func in
                inspect.getmembers(promotions, inspect.isfunction)]  # 单独的 promotions 模块

def best_promo(order):
    """选择可用的最佳折扣
    """
    return max(promo(order) for promo in promos)
```

- `inspect.getmembers` 函数用于获取对象（这里是 `promotions` 模块）的属性，第二个参数是可选的判断条件（一个布尔值函数）. 使用的是 `inspect.isfunction`, 只获取模块中的函数。

动态收集策略函数更为显式的一种方案是使用简单的装饰器。

- **命令模式：** 通过把函数作为参数传递而简化。
  - 目的是解耦调用操作的对象（调用者）和提供实现的对象（接收者）
  - 做法是，在二者之间放一个 `Command` 对象，让它实现只有一个方法（`execute`）的接口，调用接收者中的方法执行所需的操作。这样，调用者无需了解接收者的接口，而且不同的接收者可以适应不同的 `Command` 子类。调用者有一个具体的命令，通过调用 `execute` 方法执行。
  - 在python中，可以不为调用者提供一个 `Command` 实例，而是给它一个函数。此时，调用者不用调用 `command.execute()`，直接调用 `command() `即可. 可以是实现了 `__call__` 方法这样的类。这样，`Command` 的实例就是可调用对象，各自维护着一个函数列表，供以后调用。

```python
# MacroCommand 的各个实例都在内部存储着命令列表

class MacroCommand:
    """一个执行一组命令的命令"""

    def __init__(self, commands):
        self.commands = list(commands) # 构建一个列表，这样能确保参数是可迭代对象

    def __call__(self):
        for command in self.commands: # 调用 MacroCommand 实例时，self.commands 中的各个命令依序执行。
            command()
```

- **延伸阅读**
  - 在设计模式方面，Python 程序员的阅读选择没有其他语言多。
  - 两个设计原则 :
    - 对接口编程，而不是对实现编程
    - 优先使用对象组合，而不是类继承
  - 以为设计模式在任何语言中都有用 ? No. 一些特殊的面向对象语言可以直接支持某些模式
