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
- [处理文本](#处理文本)
- [unicode字符比较](#unicode字符比较)
- [规范化文本比较](#规范化文本比较)
- [拉丁标音字符转化为ascii](#拉丁标音字符转化)
- [unicode字符排序](#unicode字符排序)
- [正则匹配unicode字符](#正则匹配unicode字符)
- [os模块中处理的字节和字符](#os模块中处理的字节和字符)


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
  - 判断编码类型：
    - [CharDet](https://pypi.python.org/pypi/chardet)
    - 命令行：`$ chardetect 04-text-byte.asciidoc`
  - `BOM`：有用的鬼符
    - 即 **字节序标记** （byte-order mark），指明编码时使用 Intel CPU 的小字节序。
    - 如：`u16 = 'El Niño'.encode('utf_16')` 结果为： `b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'`
      - 其中 `b'\xff\xfe'` 就是BOM, 指明编码时使用 Intel CPU 的小字节序。 转换为列表：`list(u16)` => `[255, 254, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]` , 其中 `E` 编码为 `69 0` 大字节序 CPU 中，编码顺序是相反的, 为: `0 69`
    - UTF-16 有两个变种:
      - **UTF-16LE** : 显式指明使用小字节序; `list('El Niño'.encode('utf_16le'))` -> `[69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]`
      - **UTF-16BE** : 显式指明使用大字节序; `list('El Niño'.encode('utf_16be'))` -> `[0, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111]`
      - 如果使用这两个变种，不会生成 BOM.
    - 如果有 BOM，UTF-16 编解码器会将其过滤掉,提供没有BOM字符的真正文本.
    - 如果没有 BOM, 并且文件使用 UTF-16 编码，而且，那么应该假定它使用的是 UTF-16BE（大字节序）编码；然而，Intel x86 架构用的是小字节序，因此有很多文件用的是不带 BOM 的小字节序 UTF-16 编码。
    - UTF-8 编码的优势： 不管设备使用哪种字节序，生成的字节序列始终一致，因此不需要 BOM。
      - 【但windows中的一些程序依然在UTF-8编码中添加BOM，如：Notepad】
      - 【而且Excel 会根据有没有 BOM 确定文件是不是 UTF-8 编码，否则，它假设内容使用 Windows 代码页（codepage）编码。】
      - 【UTF-8 编码的 U+FEFF 字符是一个三字节序列：`b'\xef\xbb\xbf'`。因此，如果文件以这三个字节开头，有可能是带有 BOM 的 UTF-8 文件。】
  - **python3处理文本文件** <span id="处理文本"></span>
    - 处理最好遵循3步骤：
      - `bytes -> str` : 字节文本解码指定类型，转化为 字符 类型.
      - `100% str` : 百分之百只处理字符类型的文本.
      - `str -> bytes`: 存储时，指定编码类型， 存储为字节文件.
      - 总的来说，就是 **输入 转换 为 `str` 类型 处理, 处理完之后, 输出 指定 编码 类型 存储为字节文件.**
    - **需要在多台设备中或多种场合下运行的代码，一定不能依赖默认编码。打开文件时始终应该明确传入 `encoding=` 参数，因为不同的设备使用的默认编码可能不同，有时隔一天也会发生变化。**
    - 新版 GNU/Linux 或 Mac OS X系统默认使用 UTF-8 编码和解码，但windows 就比较坑了，各种编码都有可能有。 主要根据区域设置:
    - 各种默认值： GNU/Linux 或 Mac OS X系统 比较统一, 但windows下...

```python
import sys, locale

expressions = """
        locale.getpreferredencoding()
        type(my_file)
        my_file.encoding
        sys.stdout.isatty()
        sys.stdout.encoding
        sys.stdin.isatty()
        sys.stdin.encoding
        sys.stderr.isatty()
        sys.stderr.encoding
        sys.getdefaultencoding()
        sys.getfilesystemencoding()
    """

# locale.getpreferredencoding() 最重要， 是打开文件的默认编码,重定向到 sys.stdout/stdin/stderr 文件的默认编码

my_file = open('dummy', 'w')

for expression in expressions.split():
    value = eval(expression)
    print(expression.rjust(30), '->', repr(value))

# python3 default_encodings.py
# GNU/Linux 或 Mac OS X系统 输出结果为 utf-8
# 但 windows 就难说了。
```

- **正确比较而规范化Unicode字符串** <span id="unicode字符比较"></span>
  - 因为 Unicode 有组合字符（`变音符号`和`记号`，如：`é`，打印时作为一个整体），所以字符串比较起来很复杂。
  - 可以使用 `unicodedata.normalize ` 函数提供的 Unicode 规范化, 该函数有四种规范化：
    - `'NFC'`: 使用最少的码位构成等价的字符串. 【w3c推荐的规范化形式】
    - `'NFD'`: 把组合字符分解成基字符和单独的组合字符.
    - `'NFKC'`: 较严格的规范化形式，对“兼容字符”有影响。各个兼容字符会被替换成一个或多个“兼容分解”字符
    - `'NFKD'`: 较严格的规范化形式，对“兼容字符”有影响。各个兼容字符会被替换成一个或多个“兼容分解”字符
  - 使用 `NFKC` 和 `NFKD` 规范化形式时要小心，而且只能在特殊情况中使用，例如搜索和索引，而不能用于持久存储，因为这两种转换会导致数据损失。

```python
>>> from unicodedata import normalize, name
>>> s1 = 'café'  # 把"e"和重音符组合在一起
>>> s2 = 'cafe\u0301'  # 分解成"e"和重音符
>>> len(s1), len(s2)
(4, 5)
>>> len(normalize('NFC', s1)), len(normalize('NFC', s2))  # 最少码位规范化
(4, 4)
>>> len(normalize('NFD', s1)), len(normalize('NFD', s2))  # 基字符和单独的组合字符. 所以长度比较大.
(5, 5)
>>> normalize('NFC', s1) == normalize('NFC', s2)  # 用同一种规范化时，可以比较是否相等. 在中文中貌似没有什么多大用，不过万一将来国际化要用到呢。
True
>>> normalize('NFD', s1) == normalize('NFD', s2)
True

# 电阻的单位欧姆（Ω）会被规范成希腊字母大写的欧米加， 字符在视觉上是一样的，但是比较时并不相等，因此要规范化，防止出现意外。
>>> ohm = '\u2126'
>>> ohm
'Ω'
>>> name(ohm)
'OHM SIGN'
>>> ohm_c = normalize('NFC', ohm)
>>> ohm_c
'Ω'  # 看起来一样. 但比较时不等.
>>> name(ohm_c)
'GREEK CAPITAL LETTER OMEGA'
>>> ohm == ohm_c
False
>>> normalize('NFC', ohm) == normalize('NFC', ohm_c) # 同一种形式规范化.
True
>>> normalize('NFD', ohm) == normalize('NFD', ohm_c)
True

# NFKC 和 NFKD 形式 规范化.
>>> half = '½'
>>> normalize('NFKC', half)  
'1⁄2'  # 兼容分解后得到的是三个字符序列 '1/2'
>>> four_squared = '4²'
>>> normalize('NFKC', four_squared)
'42'  # NFKC 或 NFKD 可能会损失或曲解信息，但是可以为搜索和索引提供便利的中间表述：
      # 用户搜索 '1 / 2 inch' 时，如果还能找到包含 '½ inch' 的文档，那么用户会感到满意。
>>> micro = 'μ' # U+00B5
>>> micro_kc = normalize('NFKC', micro)  # 兼容分解后得到的是小写字母 'μ' U+03BC
>>> micro, micro_kc
('μ', 'μ')
>>> ord(micro), ord(micro_kc)  # 字节序列不一样.
(181, 956)
>>> name(micro), name(micro_kc)  # 名称不一样.
('MICRO SIGN', 'GREEK SMALL LETTER MU')
```

- **大小写折叠**
  - 其实就是把所有文本变成小写，再做些其他转换。这个功能由 `str.casefold()` 方法（Python 3.3 新增）支持。
  - 对于只包含 `latin1` 字符的字符串 `s`，`s.casefold()` 得到的结果与 `s.lower()` 一样，有两个例外:
    - 微符号 'µ' 会变成小写的希腊字母“μ”（在多数字体中二者看起来一样）
    - 德语 Eszett（“sharp s”，ß）会变成“ss”。
  - `str.casefold()` 和 `str.lower()` 得到不同结果的有 116 个码位。Unicode 6.3 命名了 110 122 个字符，这只占 0.11%。
- **规范化文本比较**

```python
"""
Utility functions for normalized Unicode string comparison.

Using Normal Form C, case sensitive:

    >>> s1 = 'café'
    >>> s2 = 'cafe\u0301'
    >>> s1 == s2
    False
    >>> nfc_equal(s1, s2)
    True
    >>> nfc_equal('A', 'a')
    False

Using Normal Form C with case folding:

    >>> s3 = 'Straße'
    >>> s4 = 'strasse'
    >>> s3 == s4
    False
    >>> nfc_equal(s3, s4)
    False
    >>> fold_equal(s3, s4)
    True
    >>> fold_equal(s1, s2)
    True
    >>> fold_equal('A', 'a')
    True

"""

from unicodedata import normalize

def nfc_equal(str1, str2):
    return normalize('NFC', str1) == normalize('NFC', str2)

def fold_equal(str1, str2):
    return (normalize('NFC', str1).casefold() ==
            normalize('NFC', str2).casefold())

# 极端“规范化”：去掉变音符号
# 主要应用于搜索

import unicodedata
import string


def shave_marks(txt):
    """去掉全部变音符号"""
    norm_txt = unicodedata.normalize('NFD', txt)  # 分解成基字符和组合记号。
    shaved = ''.join(c for c in norm_txt
                     if not unicodedata.combining(c))  #过滤掉所有组合记号。
    return unicodedata.normalize('NFC', shaved)  # 重组所有字符。

# 应用：
>>> order = '“Herr Voß: • ½ cup of OEtker™ caffè latte • bowl of açaí.”'
>>> shave_marks(order)
'“Herr Voß: • ½ cup of OEtker™ caffe latte • bowl of acai.”'  # 只替换了“è”“ç”和“í”三个字符。
>>> Greek = 'Zέφupoς, Zéfiro'
>>> shave_marks(Greek)
'Ζεφupoς, Zefiro' #  “έ”和“é”都被替换了

def shave_marks_latin(txt):
    """把拉丁基字符中所有的变音符号删除"""
    norm_txt = unicodedata.normalize('NFD', txt)  # 分解成基字符和组合字符。
    latin_base = True
    keepers = []
    for c in norm_txt:
        if unicodedata.combining(c) and latin_base:  # 基字符为拉丁字母时，跳过组合字符。
            continue  # 忽略拉丁基字符上的变音符号
        keepers.append(c)                            # 否则，保存当前字符。
        # 如果不是组合字符，那就是新的基字符
        if not unicodedata.combining(c):             # 检测新的基字符，判断是不是拉丁字母。
            latin_base = c in string.ascii_letters  # ascii 是 拉丁字符罗？ 为false时，即为标记.
    shaved = ''.join(keepers)
    return unicodedata.normalize('NFC', shaved)      # 重组所有字符
```

参考:[拉丁字母](http://baike.baidu.com/item/%E6%8B%89%E4%B8%81%E5%AD%97%E6%AF%8D/1936851)

<span id="拉丁标音字符转化"></span>

```python
# 把一些西文印刷字符转换成 ASCII 字符

single_map = str.maketrans("""‚ƒ„†ˆ‹‘’“”•–—˜›""",  # 构建字符替换字符的映射表
                           """'f"*^<''""---~>""")

# 构建字符替换字符串的映射表。
multi_map = str.maketrans({
    '€': '<euro>',
    '…': '...',
    'Œ': 'OE',
    '™': '(TM)',
    'œ': 'oe',
    '‰': '<per mille>',
    '‡': '**',
})

multi_map.update(single_map)  # 合并两个映射表。


def dewinize(txt):
    """把Win1252符号替换成ASCII字符或序列"""
    return txt.translate(multi_map)  # 只替换 Microsoft 在 cp1252 中为 latin1 额外添加的字符。

def asciize(txt):
    no_marks = shave_marks_latin(dewinize(txt))     # 去掉变音符号。
    no_marks = no_marks.replace('ß', 'ss')          # 把德语 Eszett 替换成“ss”（这里没有使用大小写折叠，因为我们想保留大小写）。
    return unicodedata.normalize('NFKC', no_marks)  # 使用 NFKC 规范化形式把字符和与之兼容的码位组合起来。

# 应用：转化为ascii字符
>>> order = '“Herr Voß: • ½ cup of OEtker™ caffè latte • bowl of açaí.”'
>>> dewinize(order)
'"Herr Voß: - ½ cup of OEtker(TM) caffè latte - bowl of açaí."'  # 替换弯引号、项目符号和™（商标符号）。
>>> asciize(order)
'"Herr Voss: - 1⁄2 cup of OEtker(TM) caffe latte - bowl of acai."'  # 去掉变音符号，还会替换 'ß'。
```

- **Unicode文本排序** <span id="unicode字符排序"></span>
  - Python 比较任何类型的序列时，会一一比较序列里的各个元素。对字符串来说，比较的是码位。可是在比较非 ASCII 字符时，得到的结果不尽如人意。
  - 不同的区域采用的排序规则有所不同，葡萄牙语等很多语言按照拉丁字母表排序，重音符号和下加符对排序几乎没什么影响。
  - 在 Python 中，非 ASCII 文本的标准排序方式是使用 `locale.strxfrm` 函数，根据 [locale 模块](https://docs.python.org/3/library/locale.html?highlight=strxfrm#locale.strxfrm) 的文档，这 个函数会“ **把字符串转换成适合所在区域进行比较的形式** ”。
  - 使用 `locale.strxfrm` 函数之前，必须先为应用设定合适的区域设置，还要祈祷操作系统支持这项设置。
    - 如：设置为 `pt_BR`; `import locale; locale.setlocale(locale.LC_COLLATE, 'pt_BR.UTF-8')`
  - **注意：**
    - 区域设置是全局的，因此不推荐在库中调用 setlocale 函数。应用或框架应该在进程启动时设定区域设置，而且此后不要再修改。
    - 操作系统必须支持区域设置，否则 setlocale 函数会抛出 locale.Error: unsupported locale setting 异常。
    - 必须知道如何拼写区域名称。 Linux 中文为：`zh_CN.UTF-8` . windows 中参考MSDN的[“Language Identifier Constants and Strings”](https://msdn.microsoft.com/en-us/library/dd318693.aspx)
    - 操作系统的制作者必须正确实现了所设的区域。
  - 替代方案：第三方库：[PyUCA](https://github.com/jtauber/pyuca)
  - PyUCA 没有考虑区域设置。如果想定制排序方式，可以把自定义的排序表路径传给 `Collator()` 构造方法。PyUCA 默认使用项目自带的 `allkeys.txt`，这就是 Unicode 6.3.0 的“[Default Unicode Collation Element Table](http://www.unicode.org/Public/UCA/6.3.0/allkeys.txt)”的副本。

```python
>>> import pyuca
>>> coll = pyuca.Collator()
>>> fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted_fruits = sorted(fruits, key=coll.sort_key)
>>> sorted_fruits
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```

- **Unicode数据库**
  - Unicode 标准提供了一个完整的数据库（许多格式化的文本文件），不仅包括码位与字符名称之间的映射，还有各个字符的元数据，以及字符之间的关系。
    - 例如：记录了字符是否可以打印、是不是字母、是不是数字，或者是不是其他数值符号。
    - 字符串的 `isidentifier`、`isprintable`、`isdecimal` 和 `isnumeric` 等方法就是靠这些信息作判断的。 `str.casefold` 方法也用到了 Unicode 表中的信息。
  - unicodedata 模块中有几个函数用于获取字符的元数据。
    - 例如： 字符在标准中的官方名称是不是组合字符（结合波形符构成的变音符号等），以及符号对应的人类可读数值。
    - 函数： `unicodedata.name() ` 、`unicodedata.numeric()` 以及 字符串的 `.isdecimal()` 和 `.isnumeric()` 方法的用法。
  - 更多unicodedata 库的使用方法， 参考： [官网文档](https://docs.python.org/3/library/unicodedata.html) or [中文版](http://python.usyiyi.cn/documents/python_352/library/unicodedata.html)

```python
import unicodedata
import re

re_digit = re.compile(r'\d')

sample = '1\xbc\xb2\u0969\u136b\u216b\u2466\u2480\u3285'

for char in sample:
    print('U+%04x' % ord(char),                        # U+0000 格式的码位。
          char.center(6),                              # 在长度为 6 的字符串中居中显示字符。
          're_dig' if re_digit.match(char) else '-',   # 如果字符匹配正则表达式 r'\d'，显示 re_dig。
          'isdig' if char.isdigit() else '-',          # 如果 char.isdigit() 返回 True，显示 isdig。
          'isnum' if char.isnumeric() else '-',        # 如果 char.isnumeric() 返回 True，显示 isnum。
          format(unicodedata.numeric(char), '5.2f'),   # 使用长度为 5、小数点后保留 2 位的浮点数显示数值。 这表明，Unicode 知道表示数字的符号的数值。
          unicodedata.name(char),                      # Unicode 标准中字符的名称。
          sep='\t')

# 输出：
# U+0031	  1   	re_dig	isdig	   isnum	 1.00	  DIGIT ONE
# U+00bc	  ¼   	-	      -	       isnum	 0.25	  VULGAR FRACTION ONE QUARTER
# U+00b2	  ²   	-	      isdig	   isnum	 2.00	  SUPERSCRIPT TWO
# U+0969	  ३   	re_dig	isdig	   isnum	 3.00	  DEVANAGARI DIGIT THREE
# U+136b	  ፫   	-	      isdig	   isnum	 3.00	  ETHIOPIC DIGIT THREE
# U+216b	  Ⅻ    -	     -	      isnum	 12.00	 ROMAN NUMERAL TWELVE
# U+2466	  ⑦    -	     isdig	  isnum	  7.00	 CIRCLED DIGIT SEVEN
# U+2480	  ⒀    -	     -	      isnum	 13.00	 PARENTHESIZED NUMBER THIRTEEN
# U+3285	  ㊅    -	     -	      isnum	  6.00	 CIRCLED IDEOGRAPH SIX
```

正则表达式 `r'\d'` 能匹配数字“1”和梵文数字 3，但是不能匹配 `isdigit` 方法判断为数字的其他字符。`re` 模块对 Unicode 的支持并不充分。

PyPI 中有个新开发的 `regex` 模块，它的最终目的是取代` re` 模块，以提供更好的 Unicode 支持。

- **正则匹配unicode字符** <span id="正则匹配unicode字符"></span>
  - 如果使用字节序列构建正则表达式，`\d` 和 `\w` 等模式只能匹配 ASCII 字符；
  - 如果是字符串模式，就能匹配 ASCII 之外的 `Unicode` 数字或字母。
    - 如下面的示例
  - 字符串正则表达式有个 `re.ASCII` 标志，它让 `\w`、`\W`、`\b`、`\B`、`\d`、`\D`、`\s` 和 `\S` 只匹配 `ASCII` 字符。详情参阅 [re 模块的文档](https://docs.python.org/3/library/re.html)或 [中文版](http://python.usyiyi.cn/documents/python_352/library/re.html)。

```python
import re

re_numbers_str = re.compile(r'\d+')     # 字符串类型。
re_words_str = re.compile(r'\w+')
re_numbers_bytes = re.compile(rb'\d+')  # 字节类型.
re_words_bytes = re.compile(rb'\w+')

text_str = ("Ramanujan saw \u0be7\u0bed\u0be8\u0bef"  # Unicode 文本，包括 1729 的泰米尔数字
            " as 1729 = 1³ + 12³ = 9³ + 10³.")        # 字符串在编译时与前一个拼接起来

text_bytes = text_str.encode('utf_8')  # 字节序列只能用字节序列正则表达式搜索。

print('Text', repr(text_str), sep='\n  ')
print('Numbers')
print('  str  :', re_numbers_str.findall(text_str))      # 字符串模式 r'\d+' 能匹配泰米尔数字和 ASCII 数字。
print('  bytes:', re_numbers_bytes.findall(text_bytes))  # 字节序列模式 rb'\d+' 只能匹配 ASCII 字节中的数字。
print('Words')
print('  str  :', re_words_str.findall(text_str))        # 字符串模式 r'\w+' 能匹配字母、上标、泰米尔数字和 ASCII 数字。
print('  bytes:', re_words_bytes.findall(text_bytes))    # 字节序列模式 rb'\w+' 只能匹配 ASCII 字节中的字母和数字。

# 输出:
# Text
#   'Ramanujan saw ௧௭௨௯ as 1729 = 1³ + 12³ = 9³ + 10³.'
# Numbers
#   str  : ['௧௭௨௯', '1729', '1', '12', '9', '10']
#   bytes: [b'1729', b'1', b'12', b'9', b'10']
# Words
#   str  : ['Ramanujan', 'saw', '௧௭௨௯', 'as', '1729', '1³', '12³', '9³', '10³']
#   bytes: [b'Ramanujan', b'saw', b'as', b'1729', b'1', b'12', b'9', b'10']
```

- **双模式字节处理** <span id="os模块中处理的字节和字符"></span>
  - `os` 模块中的所有函数、文件名或路径名参数既能使用字符串，也能使用字节序列。
    - 如果这样的函数使用字符串参数调用，该参数会使用 `sys.getfilesystemencoding()` 得到的编解码器自动编码，然后操作系统会使用相同的编解码器解码。
    - 如果必须处理（也可能是修正）那些无法使用上述方式自动处理的文件名，可以把字节序列参数传给 `os` 模块中的函数，得到字节序列返回值。这一特性允许我们处理任何文件名或路径名，不管里面有多少鬼符.
  - `os` 模块提供了特殊的编码和解码函数。为了便于手动处理字符串或字节序列形式的文件名或路径名.
    - `fsencode(filename)` : 如果 `filename` 是 `str` 类型（此外还可能是 `bytes` 类型），使用 `sys.getfilesystemencoding()` 返回的编解码器把 `filename` 编码成字节序列；否则，返回未经修改的 filename 字节序列。
    - `fsdecode(filename)` : 如果 `filename` 是 `bytes` 类型（此外还可能是 `str` 类型），使用 `sys.getfilesystemencoding()` 返回的编解码器把 `filename` 解码成字符串；否则，返回未经修改的 `filename` 字符串。
    - 在 Unix 衍生平台中，这些函数使用 `surrogateescape` 错误处理方式以避免遇到意外字节序列时卡住。Windows 使用的错误处理方式是 `strict`.

```python
# 把字符串和字节序列参数传给 listdir 函数得到的结果
>>> os.listdir('.')   # 有一个希腊字母 π
['abc.txt', 'digits-of-π.txt']
>>> os.listdir(b'.')  # 参数为字节序列, 返回值中也有一个为字节序列. `b'\xcf\x80'` 是希腊字母 π 的 UTF-8 编码。
[b'abc.txt', b'digits-of-\xcf\x80.txt']
```

**surrogateescape 处理方式**

Python 3.1 引入的 surrogateescape 编解码器错误处理方式是处理意外字节序列或未知编码的一种方式，它的说明参见“[PEP 383 — Non-decodable Bytes in System Character Interfaces](https://www.python.org/dev/peps/pep-0383/)”。

这种错误处理方式会把每个无法解码的字节替换成 `Unicode` 中 `U+DC00` 到 `U+DCFF` 之间的码位（Unicode 标准把这些码位称为“Low Surrogate Area”）， **这些码位是保留的，没有分配字符，供应用程序内部使用** 。编码时，这些码位会转换成被替换的字节值.

```python
# 使用 surrogateescape 错误处理方式

>>> os.listdir('.')  # 列出目录里的文件，有个文件名中包含非 ASCII 字符。
['abc.txt', 'digits-of-π.txt']
>>> os.listdir(b'.')  # 假设我们不知道编码，获取文件名的字节序列形式。
[b'abc.txt', b'digits-of-\xcf\x80.txt']
>>> pi_name_bytes = os.listdir(b'.')[1]  # pi_names_bytes 是包含 π 的文件名。
>>> pi_name_str = pi_name_bytes.decode('ascii', 'surrogateescape')  # 使用'ascii' 编解码器和 'surrogateescape' 错误处理方式把它解码成字符串。
>>> pi_name_str  # 各个非 ASCII 字节替换成代替码位：'\xcf\x80' 变成了'\udccf\udc80'
'digits-of-\udccf\udc80.txt'
>>> pi_name_str.encode('ascii', 'surrogateescape')  # 编码成 ASCII 字节序列：各个代替码位还原成被替换的字节。
b'digits-of-\xcf\x80.txt
```

- **延伸阅读:**
  - [Pragmatic Unicode—or—How Do I Stop the Pain?](http://nedbatchelder.com/text/unipain.html), Ned Batchelder 在 2012 年的 PyCon US 所做的演讲. [幻灯片](http://www.slideshare.net/fischertrav/character-encoding-unicode-how-to-with-dignity-33352863)。 “人类使用文本，计算机使用字节序列。”
  - [Unconfusing Unicode: What Is Unicode?](https://regebro.wordpress.com/2011/03/23/unconfusing-unicode-what-is-unicode/), 提出了“Useful Mental Model of Unicode（UMMU）”。Unicode 是个复杂的标准，Lennart 提出的 UMMU 是个很好的切入点。
  - [Unicode HOWTO](https://docs.python.org/3/howto/unicode.html), Python 文档 从几个不同的角度对本章所涉及的话题做了讨论，从编码历史到句法细节、编解码器、正则表达式、文件名和 Unicode 的 I/O 最佳实践（即 Unicode 三明治），而且各节都给出了大量参考资料链接。
  - [What's New in Python 3.0](https://docs.python.org/3.0/whatsnew/3.0.html#text-vs-data-instead-of-unicode-vs-8-bit), 这篇文章简要列出了新版的 15 点变化，而且附有很多链接。
  - [The Updated Guide to Unicode on Python](http://lucumr.pocoo.org/2013/7/2/the-updated-guide-to-unicode/) 深入分析了 Python 3 中 Unicode 的一些陷阱（Armin 不是很热衷于 Python 3）。
  - 《Python Cookbook（第 3 版）中文版》 的第 2 章“字符串和文本”中有几个诀窍谈到了 Unicode 规范化、清洗文本，以及在字节序列上执行面向文本的操作。第 5 章涵盖文件和 I/O，“5.17 将字节数据写入文本文件”指出，任何文本文件的底层都有一个二进制流，如果需要可以直接访问。之后的“6.11 读写二进制结构的数组”用到了 `struct` 模块。
  - [Python 3 and ASCII Compatible Binary Protocols](http://python-notes.curiousefficiency.org/en/latest/python3/binary_protocols.html) 和 [Processing Text Files in Python 3](http://python-notes.curiousefficiency.org/en/latest/python3/text_file_processing.html) 两篇文章与本章的话题十分相关.
  - [PEP 467—Minor API improvements for binary sequences](https://www.python.org/dev/peps/pep-0467/), Python 3.5 将为二进制序列引入新的构造方法和方法，而且会废弃目前使用的构造方法签名.
  - [PEP 461—Adding % formatting to bytes and bytearray](https://www.python.org/dev/peps/pep-0461/) , Python 3.5 实现.
  - [Standard Encodings](https://docs.python.org/3/library/codecs.html#standard-encodings), 为 `codecs` 模块文档一节的Python 支持的编码列表。
  - [CPython 源码中 /Tools/unicode/listcodecs.py 脚本](https://hg.python.org/cpython/file/6dcc96fa3970/Tools/unicode/listcodecs.py) , 通过编程的方式获得Python 支持的编码列表.
  - [Changing the Python Default Encoding Considered Harmful](http://blog.startifact.com/posts/older/changing-the-python-default-encoding-considered-harmful.html) 和 [sys.setdefaultencoding Is Evil](http://blog.ziade.org/2008/01/08/syssetdefaultencoding-is-evil/), 解释了为什么一定不能修改 `sys.getdefaultencoding()` 获取的编码，即便知道怎么做也不能改。
  - [Programming with Unicode](http://unicodebook.readthedocs.org/index.html), 是一本免费的自出版图书（遵守 CC BY-SA 协议），其中讨论了一般的 Unicode 话题，以及主流操作系统和几门编程语言（包括 Python）中的相关工具和 API。
  - [Case Folding: An Introduction](https://www.w3.org/International/wiki/Case_folding) 和 [Character Model for the World Wide Web: String Matching and Searching](https://www.w3.org/TR/charmod-norm/), 为w3c讨论的规范化相关的概念， 前者是介绍性文章，后者则是以枯燥的标准用语写就的工作草案。`Unicode.org` 网站中的“[Frequently Asked Questions / Normalization](http://www.unicode.org/faq/normalization.html)”更容易理解， Mark Davis 写的“[NFC FAQ](http://www.macchiato.com/unicode/nfc-faq)”也是如此。Mark 是多个 Unicode 算法的作者，在写作本书时，他还在担任 Unicode 联盟的主席。
  - **纯文本:** 只由特定标准的码位序列组成的计算机编码文本，其中不含其他格式化或结构化信息。
  - **在 RAM 中如何表示字符串**: 在内存中，Python 3 使用固定数量的字节存储字符串的各个码位，以便高效访问各个字符或切片。
    - 在 Python 3.3 之前，编译 CPython 时可以配置在内存中使用 16 位或 32 位存储各个码位。16 位是“窄构建”（narrow build），32 位是“宽构建”（wide build）。如果想知道用的是哪个，要查看 `sys.maxunicode` 的值：65535 表示“窄构建”，不能透明地处理 U+FFFF 以上的码位。“宽构建”没有这个限制，但是消耗的内存更多：每个字符占 4 个字节，就算是中文象形文字的码位大多数也只占 2 个字节。这两种构建没有高下之分，应该根据自己的需求选择。
    - 从 Python 3.3 起，创建 `str` 对象时，解释器会检查里面的字符，然后为该字符串选择最经济的内存布局： **如果字符都在 `latin1` 字符集中，那就使用 1 个字节存储每个码位；否则，根据字符串中的具体字符，选择 2 个或 4 个字节存储每个码位**。这是简述，完整细节参阅“[PEP 393—Flexible String Representation](https://www.python.org/dev/peps/pep-0393/)”。
    - 灵活的字符串表述类似于 Python 3 对 int 类型的处理方式：如果一个整数在一个机器字中放得下，那就存储在一个机器字中；否则解释器切换成变长表述，类似于 Python 2 中的 long 类型。
