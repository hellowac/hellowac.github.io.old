---
nav: blog
layout: post
title: "流畅的python - 继承"
author: "wangchao"
tags:
  - python
  - '对象'
  - '继承'
  - 'Django'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [继承内置类型](#继承内置类型)
- [多重继承和方法解析顺序](#多重继承和方法解析顺序)
- [多重继承的应用](#多重继承的应用)
- [Django通用视图中的混入](#Django通用视图中的混入)
- [延伸阅读](#延伸阅读)


<span id="继承内置类型"></span>

### 继承内置类型

在 Python 2.2 之前，内置类型（如 list 或 dict）不能子类化.在 Python2.2 之后，内置类型可以子类化了，但 `内置类型（使用 C 语言编写）不会调用用户定义的类覆盖的特殊方法`。

- **dict** 的子类覆盖的 `__getitem__()` 方法不会被内置类型的 `get()` 方法调用。
- 内置类型 **dict** 的 `__init__` 和 `__update__` 方法会忽略我们覆盖的 `__setitem__` 方法
  - `dict.update` 方法也会忽略子类的 `__getitem__` 方法


```python
In [35]: class DopplDict(dict):
    ...:     def __setitem__(self,key,value):
    ...:         super().__setitem__(key,[value]*2)  # 委托给超类
    ...:

In [36]: dd = DopplDict(one=1)

In [37]: dd
Out[37]: {'one': 1}   # 初始方法显然忽略了 自定义的 __setitem__ 方法.

In [38]: dd['two'] = 2  # [] 方法 会调用我们自定义的覆盖的 __setitem__ 方法.

In [39]: dd
Out[39]: {'one': 1, 'two': [2, 2]}

In [40]: dd.update(three=3)  # 继承的 update 也不会调用覆盖的 __setitem__ 方法.

In [41]: dd
Out[41]: {'one': 1, 'three': 3, 'two': [2, 2]}

In [44]: class AnswerDict(dict):
    ...:     def __getitem__(self,key):
    ...:         return 43
    ...:

In [45]: ad = AnswerDict(a='foo')

In [46]: ad['a']  # 自定义的 __getitem__ 方法 只对 [] 调用有效。
Out[46]: 43

In [47]: ad
Out[47]: {'a': 43}

In [48]: d = {}

In [49]: d.update(ad)  # 忽略了自定义的 __getitem__ 方法.

In [50]: d['a']   # 未安装预期更新。
Out[50]: 'foo'

In [51]: d
Out[51]: {'a': 'foo'}
```

**直接子类化内置类型（如 `dict`、`list` 或 `str`）容易出错，因为内置类型的方法通常会忽略用户覆盖的方法。
不要子类化内置类型，用户自己定义的类应该继承 `collections` 模块中的类，例如 `UserDict`、`UserList` 和 `UserString`，这些类做了特殊设计，因此易于扩展。**

```python
>>> import collections
>>>
>>> class DoppelDict2(collections.UserDict):
...     def __setitem__(self, key, value):
...         super().__setitem__(key, [value] * 2)
...
>>> dd = DoppelDict2(one=1)
>>> dd
{'one': [1, 1]}
>>> dd['two'] = 2
>>> dd
{'two': [2, 2], 'one': [1, 1]}
>>> dd.update(three=3)
>>> dd
{'two': [2, 2], 'three': [3, 3], 'one': [1, 1]}
>>>
>>> class AnswerDict2(collections.UserDict):
...     def __getitem__(self, key):
...         return 42
...
>>> ad = AnswerDict2(a='foo')
>>> ad['a']
42
>>> d = {}
>>> d.update(ad)
>>> d['a']
42
>>> d
{'a': 42}
```

<span id="多重继承和方法解析顺序"></span>

### 多重继承和方法解析顺序

任何实现多重继承的语言都要处理潜在的命名冲突，这种冲突由不相关的祖先类实现同名方法引起。这种冲突称为“ **菱形问题** ”

![菱形问题]({% link assets/programingimg/subclass.png %})

```python
class A:
    def ping(self):
        print('ping:', self)


class B(A):
    def pong(self):
        print('pong:', self)


class C(A):
    def pong(self):
        print('PONG:', self)


class D(B, C):

    def ping(self):
        super().ping()
        print('post-ping:', self)

    def pingpong(self):
        self.ping()
        super().ping()  # 使用父类的ping
        self.pong()  # 从 __mro__ 存储的类顺序开始搜索。
        super().pong()  # 显式调用父类的 pong 方法.
        C.pong(self)  # 指定调用某个父类的 方法. 此时应传入当前实例，否则为 为绑定方法. 抛出 unbound error.
```

python中 特殊属性 `__mro__` （Method Resolution Order，MRO）存储了继承的类顺序，并且寻找执行方法时会按照该顺序来查找。

方法解析顺序不仅考虑继承图，还考虑子类声明中列出超类的顺序。 如 `D` 类声明为 `class D(C, B):`，那么 `D` 类的 `__mro__` 属性就会不一样：先搜索 `C` 类，再搜索 `B` 类。

默认从当前类开始按照 `__mro__` 存储的顺序开始搜寻，但可以使用 `super()` 方法 来指定调用父类的方法 和 显式指定 类来调用。

但在python2 中 super() 函数 应该这样调用 `super(D, self).ping()`

```python
>>> D.__mro__
(<class 'diamond.D'>, <class 'diamond.B'>, <class 'diamond.C'>,
<class 'diamond.A'>, <class 'object'>)

# 如标准库中 tkinter 包.
>>> def print_mro(cls):
...     print(', '.join(c.__name__ for c in cls.__mro__))
...
>>> import tkinter
>>> print_mro(tkinter.Text)
Text, Widget, BaseWidget, Misc, Pack, Place, Grid, XView, YView, object
```

![解析顺序]({% link assets/programingimg/subclass2.png %})

<span id="多重继承的应用"></span>

### 多重继承的应用

在 Python 标准库中，最常使用多重继承的是 `collections.abc` 包.

GUI 工具包 [Tkinter](https://docs.python.org/3/library/tkinter.html) 把多重继承用到了极致。

下图是 `Tkinter` 小组件([tkinter.ttk](https://docs.python.org/3/library/tkinter.ttk.html)) 层次结构的一部分 如图：

![多重继承]({% link assets/programingimg/subclass3.png %})

1. Toplevel：表示 Tkinter 应用程序中顶层窗口的类。
2. Widget：窗口中所有可见对象的超类。
3. Button：普通的按钮小组件。
4. Entry：单行可编辑文本字段。
5. Text：多行可编辑文本字段。

在类之间的关系方面有几点要注意:

- `Toplevel` 是所有图形类中唯一没有继承 `Widget` 的，因为它是顶层窗口，行为不像小组件，例如不能依附到窗口或窗体上。`Toplevel` 继承自 `Wm`，后者提供直接访问宿主窗口管理器的函数，例如设置窗口标题和配置窗口边框。
- `Widget` 直接继承自 `BaseWidget`，还继承了 `Pack`、`Place` 和 `Grid`。后三个类是几何管理器，负责在窗口或窗体中排布小组件。各个类封装了不同的布局策略和小组件位置 API。
- `Button` 与大多数小组件一样，只是 `Widget` 的子代，也间接继承 `Misc`，后者为各个小组件提供了大量方法。
- `Entry` 是 `Widget` 和 `XView` 的子类，后者实现横向滚动。
- `Text` 是 `Widget`、`XView` 和 `YView` 的子类，后者提供纵向滚动功能。

处理多继承的一些建议：

- **把接口继承和实现继承区分开**
  - 使用多重继承时，一定要明确一开始为什么创建子类。主要原因可能有：
    - 继承接口，创建子类型，实现“是什么”关系
    - 继承实现，通过重用避免代码重复
  - 其实这两条经常同时出现，不过只要可能，一定要明确意图。通过继承重用代码是实现细节，通常可以换用组合和委托模式。而接口继承则是框架的支柱。
- **使用抽象类显式表示接口**
  - 现代的 Python 中，如果类的作用是定义接口，应该明确把它定义为抽象类。Python 3.4 及以上的版本中，我们要创建` abc.ABC `或其他抽象类的子类。
- **通过混入重用代码**
  - 如果一个类的作用是为多个不相关的子类提供方法实现，从而实现重用，但不体现“是什么”关系，应该把那个类明确地定义为`混入类`（mixin class）。
  - 从概念上讲，混入不定义新类型，只是打包方法，便于重用。混入类绝对不能实例化，而且具体类不能只继承混入类。混入类应该提供某方面的特定行为，只实现少量关系非常紧密的方法。
- **在名称中明确指明混入**
  - 因为在 Python 中没有把类声明为混入的正规方式，所以强烈推荐在名称中加入 `...Mixin `后缀。`Tkinter` 没有采纳这个建议，如果采纳的话，`XView` 会变成 `XViewMixin`，`Pack` 会变成 `PackMixin`，图 12-3 中所有使用 `«mixin»` 标记的类都应该这么做。
- **抽象类可以作为混入，反过来则不成立**
  - 抽象基类可以实现具体方法，因此也可以作为混入使用。不过，抽象基类会定义类型，而混入做不到。此外，抽象基类可以作为其他类的唯一基类，而混入决不能作为唯一的超类，除非继承另一个更具体的混入——真实的代码很少这样做。
  - 抽象基类有个局限是混入没有的：抽象基类中实现的具体方法只能与抽象基类及其超类中的方法协作。这表明，抽象基类中的具体方法只是一种便利措施，因为这些方法所做的一切，用户调用抽象基类中的其他方法也能做到。
- **不要子类化多个具体类**
  - 具体类可以没有，或最多只有一个具体超类. 也就是说，具体类的超类中除了这一个具体超类之外，其余的都是抽象基类或混入。例如，在下述代码中，如果 `Alpha` 是具体类，那么 `Beta` 和 `Gamma` 必须是抽象基类或混入：

```python
class MyConcreteClass(Alpha, Beta, Gamma):
    """这是一个具体类，可以实例化。"""
    # ……更多代码……
```

- **为用户提供聚合类**
  - 如果抽象基类或混入的组合对客户代码非常有用，那就提供一个类，使用易于理解的方式把它们结合起来。Grady Booch 把这种类称为`聚合类`（aggregate class）。
  - `Widget` 类的定义体是空的，但是这个类提供了有用的服务：把四个超类结合在一起，这样需要创建新小组件的用户无需记住全部混入，也不用担心声明 `class` 语句时有没有遵守特定的顺序。`Django` 中的 `ListView` 类是更好的例子，
  - 例如，下面是 `tkinter.Widget` 类的完整代码：

```python
class Widget(BaseWidget, Pack, Place, Grid):
    """Internal class.

    Base class for a widget which can be positioned with the
    geometry managers Pack, Place or Grid."""
    pass
```

- **“优先使用对象组合，而不是类继承”**
  - 这句话引自《设计模式：可复用面向对象软件的基础》一书，熟悉继承之后，就太容易过度使用它了。出于对秩序的诉求，我们喜欢按整洁的层次结构放置物品，程序员更是乐此不疲。
  - 然而，优先使用组合能让设计更灵活。例如，对 `tkinter.Widget` 类来说，它可以不从全部几何管理器中继承方法，而是在小组件实例中维护一个几何管理器引用，然后通过它调用方法。毕竟，小组件“不是”几何管理器，但是可以通过委托使用相关的服务。这样，我们可以放心添加新的几何管理器，不必担心会触动小组件类的层次结构，也不必担心名称冲突。即便是单继承，这个原则也能提升灵活性，因为子类化是一种紧耦合，而且较高的继承树容易倒。
  - 组合和委托可以代替混入，把行为提供给不同的类，但是不能取代接口继承去定义类型层次结构。

<span id="Django通用视图中的混入"></span>

### Django通用视图中的混入

在 `Django` 中，`视图是可调用的对象`，它的参数是 `表示HTTP请求的对象`，返回值是一个 `表示 HTTP 响应的对象`。
我们要关注的是这些`响应对象`。响应可以是简单的`重定向`，没有主体内容，也可以是复杂的内容，如在线商店的目录页面，它使用 HTML 模板渲染，列出多个货品，而且有购买按钮和详情页面链接。

起初，Django 提供的是一系列函数，这叫`通用视图`，实现常见的用例。例如，很多网站都需要展示搜索结果，里面包含很多项目，分成多页，而且各个项目会链接到详细信息页面。在 Django 中，这种需求使用列表视图和详情视图实现，前者用于渲染搜索结果，后者用于生成各个项目的详情页面。

然而，最初的通用视图是函数，不能扩展。如果需求与列表视图相似但不完全一样，那么不得不自己从头实现。

Django 1.3 引入了基于类的视图，而且还通过基类、混入和拿来即用的具体类提供了一些通用视图类。这些基类和混入在 `django.views.generic` 包的 `base` 模块里，如图所示。在这张图中，位于顶部的两个类，`View` 和 `TemplateResponseMixin`，负责完全不同的工作。

*在 [Classy Class-Based Views](http://ccbv.co.uk/) 网站中可以深入研究这些类，可以轻松地浏览各个视图类、查看它们的全部方法（继承的、覆盖的和自己添加的）、查看图表、浏览文档，以及跳转到 [GitHub 中的源码](https://github.com/django/django/tree/master/django/views/generic)。*

![多重继承]({% link assets/programingimg/subclass4.png %})

`View` 是所有视图（可能是个抽象基类）的基类，提供核心功能，如 `dispatch` 方法。这个方法委托具体子类实现的处理方法（handler），如 `get`、`head`、`post` 等，处理不同的 HTTP 动词。`RedirectView` 类只继承 `View`，可以看到，它实现了 `get`、`head`、`post` 等方法。

`View` 的具体子类应该实现处理方法，但它们为什么不在 `View` 接口中呢？原因是：子类只需实现它们想支持的处理方法。`TemplateView` 只用于显示内容，因此它只实现了 `get` 方法。如果把 `HTTP POST `请求发给 `TemplateView`，经继承的 `View.dispatch` 方法检查，它没有 `post` 处理方法，因此会返回 *HTTP 405 Method Not Allowed*（不允许使用的方法）响应。

`TemplateResponseMixin` 提供的功能只针对需要使用模板的视图。例如，`RedirectView` 没有主体内容，因此它不需要模板，也就没有继承这个混入。`TemplateResponseMixin` 为 `TemplateView` 和 `django.views.generic` 包中定义的使用模板渲染的其他视图（例如 `ListView`、`DetailView`，等等）提供行为。下图是 `django.views.generic.list` 模块和部分 `base` 模块的图解。

![多重继承]({% link assets/programingimg/subclass5.png %})

对 `Django` 用户来说，最重要的类是 `ListView`。
这是一个聚合类，不含任何代码（定义体中只有一个文档字符串）。
`ListView` 实例有个 `object_list` 属性，模板会迭代它显示页面的内容，通常是数据库查询返回的多个对象。
生成这个可迭代对象列表的相关功能都由 `MultipleObjectMixin` 提供。
这个混入还提供了复杂的分页逻辑，即在一页中显示部分结果，并提供指向其他页面的链接。

假设你想创建一个使用模板渲染的视图，但是会生成一组 JSON 格式的对象，此时用得到 `BaseListView` 类。这个类提供了易于使用的扩展点，把 `View` 和 `MultipleObjectMixin` 的功能整合在一起，避免了模板机制的开销。

与 `Tkinter` 相比，`Django` 基于类的视图 API 是多重继承更好的示例。尤其是，`Django` 的混入类易于理解：各个混入的目的明确，而且名称的后缀都是 `...Mixin`。

`Django` 用户还没有完全拥抱基于类的视图。很多人确实在使用，但是用法有限，把它们当成黑盒；需要新功能时，很多 `Django` 程序员依然选择编写单块视图函数，负责处理所有事务，而不尝试重用基视图和混入。

学习基于类的视图和根据应用需求扩展它们确实需要一些时间，不过我觉得这是值得的：基于类的视图能避免大量样板代码，便于重用，还能增进团队交流——例如，为模板和传给模板上下文的变量定义标准的名称。基于类的视图把 `Django` 视图带到了正轨上。

<span id="延伸阅读"></span>

### 延伸阅读

- [Lib/_collections_abc.py](https://hg.python.org/cpython/file/3.4/Lib/_collections_abc.py), `collections.abc` 的源码是抽象基类使用多重继承的范例——其中很多还是混入类。
- [Python's super() considered super!](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/), `Raymond Hettinger` 写的文章，从积极的角度解说了 `Python` 的 `super` 和多重继承的运作原理。
  - 这篇文章是对 `James Knight` 的“Python's Super is nifty, but you can't use it”（以前题为“[Python's Super Considered Harmful](https://fuhm.net/super-harmful/)”）一文作出的回应。

- **想想哪些类是真正需要的**
  - 大多数程序员编写应用程序而不开发框架。即便是开发框架的那些人，多数时候也是在编写应用程序。编写应用程序时，我们通常不用设计类的层次结构。我们至多会编写子类、继承抽象基类或框架提供的其他类。作为应用程序开发者，我们极少需要编写作为其他类的超类的类。我们自己编写的类几乎都是末端类（即继承树的叶子）。
  - 如果作为应用程序开发者，你发现自己在构建多层类层次结构，可能是发生了下述事件中的一个或多个。
    - 你在重新发明轮子。去找框架或库，它们提供的组件可以在应用程序中重用。
    - 你使用的框架设计不良。去寻找替代品。
    - 你在过度设计。记住要遵守 [KISS 原则](https://zh.wikipedia.org/zh-hans/KISS%E5%8E%9F%E5%88%99)。
    - 你厌烦了编写应用程序，决定新造一个框架。恭喜，祝你好运！
