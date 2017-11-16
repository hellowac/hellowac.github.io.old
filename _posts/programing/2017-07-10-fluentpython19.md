---
nav: blog
layout: post
title: "流畅的python - Dynamic attributes and properties"
author: "wangchao"
tags:
  - python
  - 'Future'
  - '并行'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)


- [字典的属性访问](#jsontoattrdict)
- [shelve模块](#shelvemodule)
- [property](#property)
- [关于处理属性的特殊属性和函数以及特殊函数](#process_attrs)

<span id="jsontoattrdict"></span>

### json 转化为属性访问方式.

元数据的一部分. [osconfeed.json](https://github.com/fluentpython/example-code/tree/master/19-dyn-attr-prop/oscon/data)

```json
{ "Schedule":
  { "conferences": [{"serial": 115 }],
    "events": [
      { "serial": 34505,
        "name": "Why Schools Don´t Use Open Source to Teach Programming",
        "event_type": "40-minute conference session",
        "time_start": "2014-07-23 11:30:00",
        "time_stop": "2014-07-23 12:10:00",
        "venue_serial": 1462,
        "description": "Aside from the fact that high school programming...",
        "website_url": "http://oscon.com/oscon2014/public/schedule/detail/34505",
        "speakers": [157509],
        "categories": ["Education"] }
    ],
    "speakers": [
      { "serial": 157509,
        "name": "Robert Lefkowitz",
        "photo": null,
        "url": "http://sharewave.com/",
        "position": "CTO",
        "affiliation": "Sharewave",
        "twitter": "sharewaveteam",
        "bio": "Robert ´r0ml´ Lefkowitz is the CTO at Sharewave, a startup..." }
    ],
    "venues": [
      { "serial": 1462,
        "name": "F151",
        "category": "Conference Venues" }
    ]
  }
}
```

加载json文件.

```python
from urllib.request import urlopen
import warnings
import os
import json
from collections import abc
import keyword

URL = 'http://www.oreilly.com/pub/sc/osconfeed'
JSON = 'data/osconfeed.json'


def load():
    if not os.path.exists(JSON):
        msg = 'downloading {} to {}'.format(URL, JSON)
        warnings.warn(msg)  # 如果需要下载，就发出提醒。
        with urlopen(URL) as remote, open(JSON, 'wb') as local:  # 在 with 语句中使用两个上下文管理器（从 Python 2.7 和 Python 3.1 起允许这么做），分别用于读取和保存远程文件。
            local.write(remote.read())

    with open(JSON) as fp:
        return json.load(fp)  # 返回 Python 原生对象。在这个数据源中有这几种数据类型：dict、list、str 和 int


class FrozenJSON:
    """A read-only façade for navigating a JSON-like object
       using attribute notation
    """

    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if keyword.iskeyword(key):  # 是否python关键字.
                key += '_'
            self.__data[key] = value        # 确保传入的是字典（或者是能转换成字典的对象）, 安全起见，创建一个副本。

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)  # 调用 keys 等方法就是通过这种方式处理的。
        else:
            return FrozenJSON.build(self.__data[name])  # 另一个构造方法.

    @classmethod
    def build(cls, obj):   # 类方法.
        if isinstance(obj, abc.Mapping):   # 是否映射或可映射的对象.
            return cls(obj)
        elif isinstance(obj, abc.MutableSequence):      # JSON中比如按是列表.
            return [cls.build(item) for item in obj]    # 递归
        else:  # 既不是字典也不是列表
            return obj
```

改良版:

```python
from collections import abc
from keyword import iskeyword


class FrozenJSON:
    """A read-only façade for navigating a JSON-like object
       using attribute notation
    """

    def __new__(cls, arg):  # 类方法，第一个参数是类本身，余下的参数与 __init__ 方法一样，只不过没有 self。
        if isinstance(arg, abc.Mapping):
            return super().__new__(cls)  # 调用的是 object 基类的 __new__ 方法，把唯一的参数设为 FrozenJSON, 
                                         # 构建FrozenJSON的实例,即实例的 __class__ 属性存储的是 FrozenJSON 类的引用
                                         # 真正的构建操作由解释器调用 C 语言实现的 object.__new__ 方法执行。
        elif isinstance(arg, abc.MutableSequence):  # 和 build 中都一样.
            return [cls(item) for item in arg]
        else:
            return arg

    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if iskeyword(key):
                key += '_'
            self.__data[key] = value

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        else:
            return FrozenJSON(self.__data[name])  # 现在只需调用 FrozenJSON 构造方法
```

<span id="shelvemodule"></span>

### shelve 模块. (调整数据源的结构)

[shelve](https://docs.python.org/3/library/shelve.html) 模块提供 [pickle](https://docs.python.org/3/library/pickle.html) 存储方式.

`shelve.open` 高阶函数返回一个 shelve.Shelf 实例，一个简单的键值对象数据库，背后由 [dbm](https://docs.python.org/3/library/dbm.html) 模块支持:

- `shelve.Shelf` 是 `abc.MutableMapping` 的子类，因此提供了处理映射类型的重要方法。
- `shelve.Shelf` 类还提供了几个管理 I/O 的方法，如 `sync` 和 `close`；它也是一个上下文管理器。
- 只要把新值赋予键，就会保存键和值。
- 键必须是字符串。
- 值必须是 `pickle` 模块能处理的对象。

```python
import warnings
import inspect

import osconfeed

DB_NAME = 'data/schedule2_db'
CONFERENCE = 'conference.115'


class Record:
    def __init__(self, **kwargs):
        self.__dict__.update(kwargs)

    def __eq__(self, other):  # 对测试有重大帮助
        if isinstance(other, Record):
            return self.__dict__ == other.__dict__
        else:
            return NotImplemented


class MissingDatabaseError(RuntimeError):
    """Raised when a database is required but was not set."""  # 自定义的异常通常是标志类，没有定义体。写一个文档字符串，说明异常的用途，比只写一个 pass 语句要好。


class DbRecord(Record):  # 扩展自 Record 类

    __db = None  # 一个打开的 shelve.Shelf 数据库的引用。

    @staticmethod
    def set_db(db):
        DbRecord.__db = db  # __db 属性仍在 DbRecord 类中设置。

    @staticmethod  # 返回值始终是 DbRecord.__db 引用的对象。
    def get_db():
        return DbRecord.__db

    @classmethod  # 类方法，在子类中易于定制它的行为。
    def fetch(cls, ident):
        db = cls.get_db()
        try:
            return db[ident]  # 从数据库中获取 ident 键对应的记录。
        except TypeError:
            if db is None:  # 说明必须设置数据库
                msg = "database not set; call '{}.set_db(my_db)'"
                raise MissingDatabaseError(msg.format(cls.__name__))
            else:  # 重新抛出 TypeError 异常
                raise

    def __repr__(self):
        if hasattr(self, 'serial'):  # 如果记录有 serial 属性，在字符串表示形式中使用。
            cls_name = self.__class__.__name__
            return '<{} serial={!r}>'.format(cls_name, self.serial)
        else:
            return super().__repr__()


class Event(DbRecord):  # 扩展自 DbRecord 类

    @property
    def venue(self):
        key = 'venue.{}'.format(self.venue_serial)
        return self.__class__.fetch(key)   # 不用self.fetch(key)? 防止任何一个事件实例有fetch属性覆盖掉继承的fetch方法.

    @property
    def speakers(self):
        if not hasattr(self, '_speaker_objs'):  # speakers 特性检查记录是否有 _speaker_objs 属性。
            spkr_serials = self.__dict__['speakers']  # 如果没有，直接从 __dict__ 实例属性中获取 'speakers' 属性的值，防止无限递归，因为这个特性的公开名称也是 speakers。
            fetch = self.__class__.fetch  # 获取 fetch 类方法的引用
            self._speaker_objs = [fetch('speaker.{}'.format(key))
                                  for key in spkr_serials]
        return self._speaker_objs  # 返回前面获取的列表。

    def __repr__(self):
        if hasattr(self, 'name'):  # 如果记录有 name 属性，在字符串表示形式中使用。
            cls_name = self.__class__.__name__
            return '<{} {!r}>'.format(cls_name, self.name)
        else:
            return super().__repr__()


def load_db(db):
    raw_data = osconfeed.load()
    warnings.warn('loading ' + DB_NAME)
    for collection, rec_list in raw_data['Schedule'].items():
        record_type = collection[:-1]  
        cls_name = record_type.capitalize()  # 把 record_type 变量的值首字母变成大写, 获取可能的类名。
        cls = globals().get(cls_name, DbRecord)  # 从模块的全局作用域中获取那个名称对应的对象；如果找不到对象，使用 DbRecord。
        if inspect.isclass(cls) and issubclass(cls, DbRecord):  # 如果获取的对象是类，而且是 DbRecord 的子类
            factory = cls  # 把对象赋值给 factory 变量。因此，factory 的值可能是 DbRecord 的任何一个子类，具体的类取决于 record_type 的值。
        else:
            factory = DbRecord  
        for record in rec_list:  # <7>
            key = '{}.{}'.format(record_type, record['serial'])
            record['serial'] = key
            db[key] = factory(**record)  # <8>

```


<span id="property"></span>

### property

内置关键自或函数: <https://docs.python.org/2.7/library/functions.html#property>

内置的 `property` 经常用作装饰器，但它是一个类。
在 `Python` 中，函数和类通常可以互换，因为他们都是可调用的对象，而且没有实例化对象的 `new` 运算符，所以调用构造方法与调用工厂函数没有区别。
此外，只要能返回新的可调用对象，代替被装饰的函数，二者都可以用作装饰器。

- `property`装饰的属性会覆盖实例属性
- 特性都是类属性，但是特性管理的其实是实例属性的存取。
- 如果实例和所属的类有同名数据属性，那么实例属性会覆盖（或称遮盖）类属性——至少通过那个实例读取属性时是这样。
    - 但实例属性不会覆盖类的 `property` 装饰的属性. (管理的就是实例的属性.)

**简单的规则**

`obj.attr` 这样的表达式不会从 `obj` 开始寻找 `attr`，而是从 `obj.__class__` 开始，而且，仅当类中没有名为 `attr` 的`被property装饰的属性`时，`Python` 才会在 `obj` 实例中寻找。

这条规则不仅适用于`被property装饰的属性`，还适用于一整类描述符——覆盖型描述符（overriding descriptor）.

**被`property`装饰的属性的文档**

- 传入 `doc` 参数：`weight = property(get_weight, set_weight, doc='weight in kilograms')`
- 取 读值 函数的 文档. (作为装饰器使用时)

**工厂函数: 返回被 `property` 装饰的属性**

```python
def quantity(storage_name):  # 名称. 采用了闭包.

    def qty_getter(instance):  # 指被装饰的属性的类的实例. (self)
        return instance.__dict__[storage_name]  # 直接从 instance.__dict__ 中获取，为的是跳过特性，防止无限递归。

    def qty_setter(instance, value):  # 设值时的函数.
        if value > 0:
            instance.__dict__[storage_name] = value  # 值直接存到 instance.__dict__ 中，为了跳过特性。
        else:
            raise ValueError('value must be > 0')

    return property(qty_getter, qty_setter)  # 构建一个自定义的property对象，然后将其返回。


class C:
    weight = quantity('weight')  # 被 property 装饰的 weight 属性会覆盖实例的weight属性. 因此对 self.weight 或 nutmeg.weight 的每个引用都由特性函数处理，只有直接存取 __dict__ 属性才能跳过特性的处理逻辑。
```

<span id="process_attrs"></span>

### 关于处理属性的特殊属性和函数以及特殊函数.

**特殊属性**

- `__class__`
    - 对象所属类的引用（即 `obj.__class__` 与 `type(obj)` 的作用相同）。`Python` 的某些特殊方法，例如 `__getattr__`，只在对象的类中寻找，而不在实例中寻找。
- `__dict__`
    - 一个映射，存储对象或类的可写属性。有 `__dict__` 属性的对象，任何时候都能随意设置新属性。如果类有 `__slots__` 属性，它的实例可能没有 `__dict__` 属性。参见下面对 `__slots__` 属性的说明。
- `__slots__`
    - 类可以定义这个这属性，限制实例能有哪些属性。`__slots__` 属性的值是一个字符串组成的元组，指明允许有的属性. 如果 `__slots__` 中没有 `'__dict__'`，那么该类的实例没有 `__dict__` 属性，实例只允许有指定名称的属性。
    - `__slots__` 属性的值虽然可以是一个列表，但是最好始终使用元组，因为处理完类的定义体之后再修改 `__slots__` 列表没有任何作用，所以使用可变的序列容易让人误解。

** 函数 **

- `dir([object])`
    - 列出对象的大多数属性。官方文档说，`dir` 函数的目的是交互式使用，因此没有提供完整的属性列表，只列出一组“重要的”属性名。
    - `dir` 函数能审查有或没有 `__dict__` 属性的对象。
    - `dir` 函数不会列出 `__dict__` 属性本身，但会列出其中的键。
    - `dir` 函数也不会列出类的几个特殊属性，例如 `__mro__`、`__bases__` 和 `__name__`。
    - 如果没有指定可选的 `object` 参数，`dir` 函数会列出当前作用域中的名称。
- `getattr(object, name[, default])`
    - 从 `object` 对象中获取 `name` 字符串对应的属性。
    - 获取的属性可能来自对象所属的类或超类。
    - 如果没有指定的属性，`getattr` 函数抛出 `AttributeError` 异常，或者返回 `default` 参数的值（如果设定了这个参数的话）。
- `hasattr(object, name)`
    - 如果 `object` 对象中存在指定的属性，或者能以某种方式（例如继承）通过 `object` 对象获取指定的属性，返回 `True`。
    - 文档说道：“这个函数的实现方法是调用 `getattr(object, name)` 函数，看看是否抛出 `AttributeError` 异常。”
- `setattr(object, name, value)`
    - 把 `object` 对象指定属性的值设为 `value`，前提是 `object` 对象能接受那个值。这个函数可能会创建一个新属性，或者覆盖现有的属性。
- `vars([object])`
    - 返回 `object` 对象的 `__dict__` 属性；如果实例所属的类定义了 `__slots__` 属性，实例没有 `__dict__` 属性，那么 `vars` 函数不能处理那个实例（相反，`dir` 函数能处理这样的实例）。
    - 如果没有指定参数，那么 `vars()` 函数的作用与 `locals()` 函数一样：返回表示本地作用域的字典。

**处理属性的特殊方法**

使用点号或内置的 `getattr`、`hasattr` 和 `setattr` 函数存取属性都会触发下述列表中相应的特殊方法。
但是，直接通过实例的 `__dict__` 属性读写属性不会触发这些特殊方法——如果需要，通常会使用这种方式跳过特殊方法。

[3.3.9. Special method lookup](https://docs.python.org/3/reference/datamodel.html#special-method-lookup) 说:

````text
对用户自己定义的类来说，如果隐式调用特殊方法，仅当特殊方法在对象所属的类型上定义，而不是在对象的实例字典中定义时，才能确保调用成功。
````

也就是说，要假定特殊方法从类上获取，即便操作目标是实例也是如此。因此，特殊方法不会被同名实例属性遮盖。

- `__delattr__(self, name)`
    - 只要使用 `del` 语句删除属性，就会调用这个方法。例如，`del obj.attr` 语句触发 `Class.__delattr__(obj, 'attr')` 方法。
- `__dir__(self)`:
    - 把对象传给 `dir` 函数时调用，列出属性。例如，`dir(obj)` 触发 `Class.__dir__(obj)` 方法。
- `__getattr__(self, name)`:
    - 仅当获取指定的属性失败，搜索过 `obj`、`Class` 和 `超类` 之后调用。
    - 表达式 `obj.no_such_attr`、`getattr(obj, 'no_such_attr')` 和 `hasattr(obj, 'no_such_attr')` 可能会触发 `Class.__getattr__(obj, 'no_such_attr')` 方法，但是，仅当在 `obj`、`Class` 和 `超类` 中找不到指定的属性时才会触发。
- `__getattribute__(self, name)`
    - 尝试获取指定的属性时总会调用这个方法，不过，寻找的属性是 `特殊属性` 或 `特殊方法` 时除外。
    - 点号与 `getattr` 和 `hasattr` 内置函数会触发这个方法。
    - 调用 `__getattribute__` 方法且抛出 `AttributeError` 异常时，才会调用 `__getattr__` 方法。
    - 为了在获取 `obj` 实例的属性时不导致无限递归，`__getattribute__` 方法的实现要使用 `super().__getattribute__(obj, name)`。
- `__setattr__(self, name, value)`
    - 尝试设置指定的属性时总会调用这个方法。
    - `点号` 和 `setattr` 内置函数会触发这个方法。例如，`obj.attr = 42` 和 `setattr(obj, 'attr', 42)` 都会触发 `Class.__setattr__(obj, ‘attr’, 42)` 方法。