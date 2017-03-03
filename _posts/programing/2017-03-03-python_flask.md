---
nav: blog
layout: post
title: "python - flask小笔记"
author: "wangchao"
tags:
  - python
  - flask
  - bootstrap
category:
  - 'Programing Teach'
show: true
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
[{{ site.nav.blog.subnav.programing.name }}]({% link blog/programing/index.md %})/
{{ page.title }}

参考:[flask-web-developement](https://book.douban.com/subject/26274202/)

- [基本初始化](#基本初始化)
- [注册路由](#注册路由)
- [上下文](#上下文)
- [会话](#会话)
- [钩子](#钩子)
- [响应](#响应)
- [自定义错误页面](#自定义错误页面)
- [url_for](#url_for)
- [flask-script](#flask-script)
- [flask-jinja2](#flask-jinja2)
- [flask-bootstrap](#flask-bootstrap)
- [flask-moment](#flask-moment)

<span id="基本初始化"></span>

#### 基本初始化:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/user/<name>')
def user(name):
    return '<h1>hello,{0}</h1>'.format(name)

if __name__ == '__main__':
    app.run(debug=True)     # 运行于本地5000端口
```

<span id="注册路由"></span>

#### 注册路由:<http://www.pythondoc.com/flask/quickstart.html#id4>

@app.route(rule)

或

app.add_url_rule(rule,endpoint=None,view_func=None,**options) # 参考:<http://www.pythondoc.com/flask/patterns/lazyloading.html>

<span id="上下文"></span>

#### 上下文:

参考:[应用上下文](http://www.pythondoc.com/flask/appcontext.html#app-context) 和 [请求上下文](http://www.pythondoc.com/flask/reqcontext.html)

变量名 | 上下文 | 说　明
------|-------|------
__current_app__ | 应用上下文 | 当前激活程序的程序实例
__g__ | 应用上下文 | 处理请求时用作临时存储的对象。每次请求都会重设这个变量
__request__ | 请求上下文 | 请求对象，封装了客户端发出的 HTTP 请求中的内容
__session__ | 请求上下文 | 用户会话，用于存储请求之间需要“记住”的值的词典

<span id="会话"></span>

#### 会话: <http://www.pythondoc.com/flask/quickstart.html#sessions>

<span id="钩子"></span>

#### 钩子:

 参考: 源码以及 <http://www.pythondoc.com/flask/api.html#flask.Flask.before_first_request>

- __before_first_request__：注册一个函数，在处理第一个请求之前运行。
- __before_request__：注册一个函数，在每次请求之前运行。
- __after_request__：注册一个函数，如果没有未处理的异常抛出，在每次请求之后运行。
- __teardown_request__：注册一个函数，即使有未处理的异常抛出，也在每次请求之后运行。

<span id="响应"></span>

#### 响应:

参考:<http://www.pythondoc.com/flask/quickstart.html#about-responses>

1. 返回HTML文本作为页面响应，默认状态码200
2. 若状态码不是200, 作为第二个返回值 返回。例如: `return '<h1>Bad Request</h1>', 400`
3. 添加header?, 那么 为 字典 作为 第三个返回值 返回. 例如: `return '<h1>Bad Request</h1>', 400 , {'referer':'http://example.com'}`
4. 不返回3个值组成的元组? 可返回 Response 对象， [make_response()](http://www.pythondoc.com/flask/api.html#flask.make_response) 函数 接收3个参数返回 一个 [Response](http://www.pythondoc.com/flask/api.html#id5) 对象
5. 重定向？使用 [redirect()](http://www.pythondoc.com/flask/api.html#flask.redirect) 辅助函数 , 参考:<http://www.pythondoc.com/flask/quickstart.html#id14>
6. 直接错误？使用 [abort()]() 辅助函数 【abort 不会把控制权交还给调用它的函数，而是抛出异常把控制权交给 Web 服
务器】, 参考:<http://www.pythondoc.com/flask/quickstart.html#id14>

<span id="flask-script"></span>

#### flask-script 扩展

使运行支持命令行选项 github地址:[flask-script](https://github.com/smurfix/flask-script)

<span id="flask-jinja2"></span>

#### flask 中 jinja2 模版引擎

参考:[渲染模版](http://www.pythondoc.com/flask/quickstart.html#id7)

官网:[jinja2](http://jinja.pocoo.org/) 和 [github](https://github.com/pallets/jinja)

flask中使用 [render_template()](http://www.pythondoc.com/flask/api.html#flask.render_template) 函数来渲染 jinja2 模版.

- 第一个参数，模版的文件名.
- 随后的参数为键值对，为 模版渲染时的 上下文.

- 变量:
  + {% raw %}__{{ name }}__{% endraw %} 表示一个变量
  + jinja2能识别所有类型的变量,包括`列表`、`字典`和`对象`.
    * {% raw %}A value from a dictionary: __{{ mydict['key'] }}__.{% endraw %}
    * {% raw %}A value from a list: __{{ mylist[3] }}__.{% endraw %}
    * {% raw %}A value from a list, with a variable index: __{{ mylist[myintvar] }}__.{% endraw %}
    * {% raw %}A value from an object's method: __{{ myobj.somemethod() }}__.{% endraw %}
- 变量中的过滤器: {% raw %}__{{ name\|capitalize }}__{% endraw %}
  + __safe__: 渲染值时不转义,【默认转义“\<h1\>Hello\</h1\>”这样的变量】
  + __capitalize__: 把值的首字母转换成大写，其他字母转换成小写
  + __lower__: 把值转换成小写形式
  + __upper__: 把值转换成大写形式
  + __title__: 把值中每个单词的首字母都转换成大写
  + __trim__: 把值的首尾空格去掉
  + __striptags__: 渲染之前把值中所有的 HTML 标签都删掉
  + 更多的过滤器参考官网[filers](http://jinja.pocoo.org/docs/2.9/templates/#builtin-filters).
- 控制结构:
  + 中文参考:[模板设计文档](http://jinja.pocoo.org/docs/2.9/templates/)
  + 英文参考:[Template Designer Documentation](http://jinja.pocoo.org/docs/2.9/templates)
  + if语句:
    * {% raw %}{% if user %} ... {% else %} ... {% endif %}{% endraw %}
  + for语句:
    * {% raw %}{% for comment in comments %} ... {% endfor %}{% endraw %}
- _宏_: macro 【类似于函数】
  + 定义: {% raw %}{% macro 名称(参数) %} ... {% endmacro %}{% endraw %}
  + 单独保存为文件后使用：
    * 导入: {% raw %}{% import 'macros.html' as macros %}{% endraw %}
    * 使用: {% raw %}{{ macros.render_comment(comment) }}{% endraw %}
- _导入_ 其他模版:
  + {% raw %}{% include 'common.html' %}{% endraw %}
- 模版 _继承_:
  + 定义 block: {% raw %}{% block head %} ... {% endblock %} {% endraw %} # block 可嵌套
  + 继承别的模版: {% raw %}{% extends "base.html" %}{% endraw %}
  + 重写继承的 block : {% raw %}{% block head %} ... {% endblock %}{% endraw %}

<span id="flask-bootstrap"></span>

#### flask-bootstrap 前端框架

- 项目地址: [Flask-Bootstrap](https://github.com/mbr/flask-bootstrap)
- 英文文档: [doc](http://pythonhosted.org/Flask-Bootstrap/)
- 中文文档: [rdt.org](http://flask-bootstrap-zh.readthedocs.io/zh/latest/)
- 中文项目地址: [flask-bootstrap-docs-zh](https://github.com/greyli/flask-bootstrap-docs-zh)

- 使用包含所有bootstrap文件的基模版.
  + {% raw %}{% extends "bootstrap/base.html" %}{% endraw %}
  + {% raw %}{{ super() }}{% endraw %} 是个好方法喔，修改而不覆盖 block 中的内容.
- 更多用法参考官方文档.

<span id="自定义错误页面"></span>

#### 自定义错误页面

@app.errorhandler(errno) 装饰器。 参考:[错误](http://www.pythondoc.com/flask/quickstart.html#id14)

例如:

```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500
```

<span id="url_for"></span>

#### 链接

连接不同页面的链接，使用[url_for()](http://www.pythondoc.com/flask/quickstart.html?highlight=url_for#url) 辅助函数，针对一个特定函数来生成URL.

url_for 的 [api](http://www.pythondoc.com/flask/api.html#flask.url_for)

返回绝对地址: 使用 `_external=True`关键字参数,[内部链接可以不用绝对地址，但是外部链接请使用绝对地址]

使用 url_for() 生成动态地址时，将动态部分作为关键字参数传入。例如，url_for
('user', name='john', _external=True) 的返回结果是 http://localhost:5000/user/john。

传入 url_for() 的关键字参数不仅限于动态路由中的参数。函数能将任何额外参数添加到
查询字符串中。例如，url_for('index', page=2) 的返回结果是 /?page=2。

__静态文件:__ static 为特殊路由, 调用`url_for('static', filename='css/styles.css', _external=True)`得到的结果是 _http://localhost:5000/static/css/styles.css。_

这个函数也可以在模版中使用.

<span id="flask-moment"></span>

#### flask-moment 操作时间

项目GitHub地址:<https://github.com/miguelgrinberg/Flask-Moment>

使用的[moment.js](http://momentjs.com/) 文档地址:<http://momentjs.com/docs/>

在模版中引入 moment.js:

```python
{% raw %}
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}

# 传入datetime对象
from datetime import datetime
@app.route('/')
def index():
    return render_template('index.html',current_time=datetime.utcnow())

# 渲染current_time:
# 'L' 到 'LLLL' 分别对应不同的复杂度,
# 'LLL' 根据客户端电脑中的时区和区域设置渲染日期和时间. 
# format还接受自定义的格式说明符.
<p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>

# fromNow() 渲染相对时间戳，而且会随着时间的推移自动刷新显示的时间
<p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>

# 实现多种语言的本地化
{{ moment.lang('es') }}
{% endraw %}
```
Flask-Moment 实现了 moment.js 中的 format()、fromNow()、fromTime()、calendar()、valueOf() 和 unix() 方法. 具体参考[文档](http://momentjs.com/docs/#/displaying/)



