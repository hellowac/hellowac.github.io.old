---
nav: blog
layout: post
title: "流畅的python - python数据模型"
author: "wangchao"
tags:
  - python
  - '数据模型'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

**隐式迭代：** 如果一个集合类型没有实现 `__contains__` 方法，那么 `in` 运算符就会按顺序做一次迭代搜索.

**数组下标方式取值的特殊方法：** `__getitem__`

**特殊方法的存在是为了被 Python 解释器调用的，自己并不需要调用它们。**

**字符格式化中的`%r`:** 用来获取对象各个属性的标准字符串表示形式.

`__repr__` 所返回的字符串应该准确、无歧义，并且尽可能表达出如何用代码创建出这个被打印的对象.

`__repr__` 和 `__str__` 的区别在于，后者是在 str() 函数被使用，或是在用 print 函数打印一个对象的时候才被调用的，并且它返回的字符串对终端用户更友好。

如果只想实现这两个特殊方法中的一个，`__repr__` 是更好的选择，因为如果一个对象没有 `__str__` 函数，而 Python 又需要调用它的时候，解释器会用 `__repr__` 作为替代。

两者的区别在 stackoverflow上的回答：[Difference between __str__ and __repr__ in Python](http://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python)

默认情况，自定义的类的实例总被认为是真的，除非这个类对 `__bool__` 或者 `__len__` 函数有自己的实现。【`__boo__`的优先级高于`__len__`】

**特殊方法一览：[Data Model](https://docs.python.org/3/reference/datamodel.html)** 列出了 83 个特殊方法的名字，其中 47 个用于实现算术运算、位运算和比较操作。

特殊方法 | 作用
--------|--------
`__repr__` | 把一个对象用字符串的形式表达出来.
`__abs__` | 如果输入是整数或者浮点数，它返回的是输入值的绝对值；<br/> 如果输入是复数（complex number），那么返回这个复数的模。
`__add__` | 加‘+’号操作符.
`__mul__` | 乘‘\*’号操作符.
`__bool__`  | 该对象的bool值.

**其他：跟运算符无关的特殊方法**

**类别** | **方法名**
--------|--------
字符串/字节序列表示形式 |  `__repr__`、`__str__`、`__format__`、<br/>`__bytes__`
数值转换 | `__abs__`、`__bool__`、`__complex__`、<br/>`__int__`、`__float__`、`__hash__`、<br/>`__index__`
集合模拟 | `__len__`、`__getitem__`、`__setitem__`、<br/>`__delitem__`、`__contains__`
迭代枚举 | `__iter__`、`__reversed__`、`__next__`
可调用模拟 | `__call__`
上下文管理 | `__enter__`、`__exit__`
实例创建和销毁 | `__new__`、`__init__`、`__del__`
属性管理 | `__getattr__`、`__getattribute__`、`__setattr__`、<br/>`__delattr__`、`__dir__`
属性描述符 | `__get__`、`__set__`、`__delete__`
跟类相关的服务 | `__prepare__`、`__instancecheck__`、`__subclasscheck__`

**其他：跟运算符无关的特殊方法**

**类别** | **方法名**
--------|--------
一元运算符 | `__neg__` -、`__pos__` +、`__abs__` abs()
众多比较运算符 | `__lt__` <、`__le__` <=、`__eq__` ==、<br/>`__ne__` !=、`__gt__` >、`__ge__` >=
算术运算符 | `__add__` +、`__sub__` -、`__mul__` *、<br/>`__truediv__` /、`__floordiv__` //、`__mod__` %、<br/>`__divmod__` divmod()、`__pow__` ** 或pow()、<br/>`__round__` round()
反向算术运算符 | `__radd__`、`__rsub__`、`__rmul__`、<br/>`__rtruediv__`、`__rfloordiv__`、`__rmod__`、<br/>`__rdivmod__`、`__rpow__`
增量赋值算术运算符 | `__iadd__`、`__isub__`、`__imul__`、<br/>`__itruediv__`、`__ifloordiv__`、`__imod__`、<br/>`__ipow__`
位运算符 | `__invert__` ~、`__lshift__` <<、`__rshift__` >>、<br/>`__and__` &、`__or__` &#124;、`__xor__` ^
反向位运算符 | `__rlshift__`、`__rrshift__`、`__rand__`、<br/>`__rxor__`、`__ror__`
增量赋值位运算符 | `__ilshift__`、`__irshift__`、`__iand__`、<br/>`__ixor__`、`__ior__`
