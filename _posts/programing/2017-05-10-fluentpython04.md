---
nav: blog
layout: post
title: "流畅的python - 文本和字节序列"
author: "wangchao"
tags:
  - python
  - 'bytes'
  - '字节'
  - '字节序列'
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[流畅的python](https://book.douban.com/subject/27028517/)

- [字符类型](#字符类型)
  - [说明](#字符说明)
- [编解码异常处理](#编解码异常处理)


- **python中的字符类型** <span id="字符类型"></span>
  - `str` unicode编码，不可变
  - `bytes` 字节编码，不可变
  - `bytearray` 字节数组，可变
  - `memoryview` 内存字节映射, 直接操作内存二进制.
- **字符的标识：**
  - **码位：**  是 0~1 114 111 的数字（十进制），在 Unicode 标准中以 4~6 个十六进制数字表示，而且加前缀“U+”。
    - 如： A：U+0041；
  - **编码：** 字符的具体表述；即在`码位`和`字节序列`之间转换时使用的算法.
    - 在 UTF-8 编码中，A（U+0041）的码位编码成单个字节 `\x41`，而在 UTF-16LE 编码中编码成两个字节 `\x41\x00`。再举个例子，欧元符号（U+20AC）在 UTF-8 编码中是三个字节——`M`，而在 UTF-16LE 中编码成两个字节：`\xac\x20`。
  - 把`码位`转换成`字节序列`的过程是编码；把`字节序列`转换成`码位`的过程是解码。【可读的字符转换为二进制的机器存储数据】

```python
>>> s = 'café'
>>> len(s)  #  4 个 Unicode 字符
4
>>> b = s.encode('utf8')  # 编码成 bytes 对象
>>> b
b'caf\xc3\xa9'  # bytes 类型以 b 开头。
>>> len(b)  # 5 个字节（在 UTF-8 中，“é”的码位编码成两个字节）。
5
>>> b.decode('utf8')  # 把 bytes 对象解码成 str 对象。
'café
```

Python 3 的 `str` 类型基本相当于 Python 2 的 `unicode` 类型；但 Python 3 的 `bytes` 类型却不是把 `str` 类型换个名称那么简单，而且还有关系紧密的 `bytearray` 类型.

- **字节概要** <span id="类型说明"></span>
  - `bytes` 或 `bytearray` 对象的各个元素是介于 0~255（含）之间的整数.
  - 指定下标返回的是一个元素. 【s[i]】
  - 切片返回则是同类型的序列. 【s[:i]】
  - 字节显示规则：
    - 可打印的 ASCII 范围内的字节（从空格到 ~），使用 ASCII 字符本身。
    - `制表符`、`换行符`、`回车符`和 `\` 对应的字节，使用转义序列 `\t`、`\n`、`\r` 和 `\\`。
    - 其他字节的值，使用十六进制转义序列（例如，`\x00` 是空字节）。
  - `str` 类型的 `encode()` 方法 将 编码unicode字符为 `bytes` 类型
  - `bytes` 类型的 `decode` 方法 将 解码bytes字节为 `str` 类型.
  - `bytes` 和 `bytearray` 的参数：
    - 一个 `str` 对象和一个 `encoding` 关键字参数。
    - 一个可迭代对象，提供 0~255 之间的数值。
    - 一个实现了缓冲协议的对象（如 `bytes`、`bytearray`、`memoryview`、`array.array`）；此时，把源对象中的字节序列复制到新建的二进制序列中。
      - 使用缓冲类对象构建二进制序列是一种低层操作，可能涉及类型转换。
  - 使用缓冲类对象创建 `bytes` 或 `bytearray` 对象时，始终复制源对象中的字节序列。 而 `memoryview` 对象允许在二进制数据结构之间共享内存。【更新，提取】
  - 如果经常操作二进制数据，可以参考：[mmap—Memory-mapped file support](https://docs.python.org/3/library/mmap.html)和官网对 [memoryview](https://docs.python.org/3/library/stdtypes.html#memory-views) 以及 [struct](https://docs.python.org/3/library/struct.html) 模块的介绍

```python
>>> import array
>>> numbers = array.array('h', [-2, -1, 0, 1, 2]) # 指定类型代码 h，创建一个短整数（16 位）数组。
>>> octets = bytes(numbers) # 保存组成 numbers 的字节序列的副本到octets中.
>>> octets
b'\xfe\xff\xff\xff\x00\x00\x01\x00\x02\x00' # 表示那 5 个短整数的 10 个字节。
```

```python
# 使用memoryview从二进制序列中提取结构化信息,提取一个 GIF 图像的宽度和高度。
>>> import struct  # 提供了一些函数，把打包的字节序列转换成不同类型字段组成的元组，还有一些函数用于执行反向转换，把元组转换成打包的字节序列.
>>> fmt = '<3s3sHH'  # 结构体的格式：< 是小字节序，3s3s 是两个 3 字节序列，HH 是两个 16 位二进制整数。
>>> with open('filter.gif', 'rb') as fp:
...     img = memoryview(fp.read())  # 创建一个 memoryview 对象
...
>>> header = img[:10]  # 使用它的切片再创建一个 memoryview 对象；这里不会复制字节序列。
>>> bytes(header)  # 转换成字节序列，这只是为了显示；这里复制了 10 字节。
b'GIF89a+\x02\xe6\x00'
>>> struct.unpack(fmt, header)  # 拆包 memoryview 对象，得到一个元组，包含类型、版本、宽度和高度。
(b'GIF', b'89a', 555, 230)
>>> del header  # 删除引用，释放 memoryview 实例所占的内存。
>>> del img
```

- **编解码的类型**
  - `encoding` 参数可以传给 `open()` 、`str.encode()` 、`bytes.decode()` 等参数. 特别的`utf_8` 有好几个别名：`'utf8'`、`'utf-8'` 和 `'U8'`
  - 其他的编码：
    - `latin1`: (即 `iso8859_1`) 一种重要的编码，是其他编码的基础，例如 cp1252 和 Unicode（注意，latin1 与 cp1252 的字节值是一样的，甚至连码位也相同）。
    - `cp1252`: Microsoft 制定的 latin1 超集，添加了有用的符号，例如弯引号和€（欧元）；有些 Windows 应用把它称为“ANSI”，但它并不是 ANSI 标准。
    - `cp437`: IBM PC 最初的字符集，包含框图符号。与后来出现的 latin1 不兼容。
    - `gb2312`: 用于编码简体中文的陈旧标准；这是亚洲语言中使用较广泛的多字节编码之一。
    - `utf-8`: 目前 Web 中最常见的 8 位编码；与 ASCII 兼容（纯 ASCII 文本是有效的 UTF-8 文本）。
    - `utf-16le`: UTF-16 的 16 位编码方案的一种形式；所有 UTF-16 支持通过转义序列（称为“代理对”，surrogate pair）表示超过 U+FFFF 的码位。

- **编解码异常处理** <span id="编解码异常处理"></span>
  - 异常类型：
    - `SyntaxError` 异常： 如果源码的编码与预期不符，加载 Python 模块时还可能抛出。
    - `UnicodeEncodeError` 异常 ： 把字符串转换成二进制序列时
    - `UnicodeDecodeError` 异常 ： 把二进制序列转换成字符串时
  - 处理异常类型：
    - `UnicodeEncodeError` 和 `UnicodeDecodeError`:
      - 传递 `errors` 参数，值为：`ignore` 忽略; `replace` 替换为 '?'，数据损坏了，但是用户知道出了问题， `xmlcharrefreplace` 替换成 XML 实体。
      - 错误处理方式可扩展：
        - 为 `errors` 参数注册额外的字符串，把一个名称和一个错误处理函数传给 `codecs.register_error` 函数。参见 [codecs.register_error](https://docs.python.org/3/library/codecs.html#codecs.register_error) 函数的文档。
    - 使用预期之外的编码加载模块时抛出的SyntaxError:
      - Python 3 默认使用 UTF-8 编码源码，Python 2（从 2.5 开始）则默认使用 ASCII。如果加载的 .py 模块中包含 UTF-8 之外的数据，而且没有声明编码，会得到类似下面的消息：`SyntaxError: Non-UTF-8 code starting with...`
      - GNU/Linux 和 OS X 系统大都使用 UTF-8，因此打开在 Windows 系统中使用 `cp1252` 编码的 .py 文件时可能发生这种情况。注意，这个错误在 Windows 版 Python 中也可能会发生，因为 Python 3 为所有平台设置的默认编码都是 UTF-8。（比如用 notepad 编辑源代码并且不以`utf-8`的格式保存文件）
