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
- [flask-wtf](#flask-wtf)
  + [支持的字段](#支持的字段)
  + [支持的验证函数](#支持的验证函数)
  + [渲染表单](#渲染表单)
  + [视图中处理表单](#视图中处理表单)
  + [重定向和会话](#重定向和会话)
  + [flash消息](#flash消息)
- [flask-SQLAlchemy](#flask-SQLAlchemy)
  + [数据库配置](#数据库配置)
  + [定义模型](#定义模型)
  + [常用的列类型](#常用的列类型)
  + [常用的列选项](#常用的列选项)
  + [关系](#关系)
  + [常用的关系选项](#常用的关系选项)
  + [数据库操作](#数据库操作)
  + [数据库查询](#数据库查询)
  + [查询过滤器](#查询过滤器)
  + [执行查询](#执行查询)
  + [视图中操作数据库](#视图中操作数据库)
  + [集成开发环境到shell](#集成开发环境到shell)
- [flask-migrate](#flask-migrate)
- [flask-mail](#flask-mail)
- [falsk-项目结构](#falsk-项目结构)

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

- Flask 在 _分发请求之前_ 激活（或推送）程序和请求上下文【在单独处理请求的线程中】，_请求处理完成后_ 再将其删除。【从处理请求的线程中.】
- 程序上下文被推送后，就可以在线程中使用 current_app 和 g 变量。
- 类似地，请求上下文被推送后，就可以使用 request 和 session 变量。
- 如果使用这些变量时我们没有激活程序上下文或请求上下文，就会导致错误。

```python
>>> from hello import app
>>> from flask import current_app
>>> current_app.name
Traceback (most recent call last):
  ...
    raise RuntimeError(_app_ctx_err_msg)
RuntimeError: Working outside of application context.
>>> app_ctx = app.app_context()
>>> app_ctx.push()
>>> current_app.name
'hello'
>>> current_app is app
False
>>> current_app.name is app.name
True
>>> id(current_app)
4340571168
>>> id(app)
4403849232
>>> id(current_app.name)
4324354640
>>> id(app.name)
4324354640
```

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

<span id="flask-wtf"></span>

#### flask-wtf

GitHub 项目地址: [flask-wtf](https://github.com/lepture/flask-wtf)

flask-wtf文档地址: [flask-wtf-doc](https://flask-wtf.readthedocs.io/en/stable/)

flask-wtf文档中文翻译项目地址: [flask-wtf-doc-zh](https://github.com/pretdo/Flask-WTF-cn)

wtf文档地址: [WTF](https://wtforms.readthedocs.io/en/latest/)

```python
# CSRF防跨站请求，设置wtf使用的密匙
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
# app.config 字典可用来存储框架、扩展和程序本身的配置变量。
# SECRET_KEY 配置变量是通用密钥，可在 Flask 和多个第三方扩展中使用。(保持私密性)

from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required

class NameForm(Form):
    # 可选参数 validators 指定一个由验证函数组成的列表，在接受用户提交的数据之前验证数据
    name = StringField('What is your name?', validators=[Required()])
    submit = SubmitField('Submit')
```

- 使用 Flask-WTF 时，每个 Web 表单都由一个继承自 Form 的类表示。
- 这个类定义表单中的一组字段，每个字段都用对象表示。
- 字段对象可附属一个或多个验证函数。验证函数用来验证用户提交的输入值是否符合要求。

<span id="支持的字段"></span>

__WTF支持的字段:__

基础类: [The Field base class](https://wtforms.readthedocs.io/en/latest/fields.html#the-field-base-class)

__字段类型__ | __说明__
-------|--------
[StringField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.StringField) | 文本字段
[TextAreaField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.TextAreaField) | 多行文本字段
[PasswordField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.PasswordField) | 密码文本字段
[HiddenField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.HiddenField) | 隐藏文本字段
[DateField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.DateField) | 文本字段，值为 datetime.date 格式
[DateTimeField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.DateTimeField) | 文本字段，值为 datetime.datetime 格式
[IntegerField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.IntegerField) | 文本字段，值为整数
[DecimalField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.DecimalField) | 文本字段，值为 decimal.Decimal
[FloatField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.FloatField) | 文本字段，值为浮点数
[BooleanField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.BooleanField) | 复选框，值为 True 和 False
[RadioField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.RadioField) | 一组单选框
[SelectField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.SelectField) | 下拉列表
[SelectMultipleField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.SelectMultipleField) | 下拉列表，可选择多个值
[FileField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.FileField) | 文件上传字段
[MultipleFileField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.MultipleFileField) | A FileField that allows choosing multiple files.
[SubmitField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.SubmitField) | 表单提交按钮
[FormField](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.FormField) | 把表单作为字段嵌入另一个表单
[FieldList](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.FieldList) | 一组指定类型的字段
[Flags](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.Flags) | Holds a set of boolean flags as attributes.
[Label](https://wtforms.readthedocs.io/en/latest/fields.html#wtforms.fields.Label) | Additional Helper 

你还可以自定义字段: [custom-fields](https://wtforms.readthedocs.io/en/latest/fields.html#custom-fields)

<span id="支持的验证函数"></span>

__WTF支持的验证函数:__

__验证函数__ | __说明__
--------|-------
[Email](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.Email) | 验证电子邮件地址
[EqualTo](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.EqualTo) | 比较两个字段的值；常用于要求输入两次密码进行确认的情况
[IPAddress](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.IPAddress) | 验证 IPv4 网络地址
[MacAddress](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.MacAddress) | Validates a MAC address.
[UUID](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.UUID) | Validates a UUID.
[Length](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.Length) | 验证输入字符串的长度
[NumberRange](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.NumberRange) | 验证输入的值在数字范围内
[Optional](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.Optional) | 无输入值时跳过其他验证函数
[DataRequired](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.DataRequired) | Checks the field’s data is ‘truthy’ otherwise stops the validation chain.
[InputRequired](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.InputRequired) | Validates that input was provided for this field.  <br />参考:[Difference between DataRequired and InputRequired](http://stackoverflow.com/questions/23982917/flask-wtforms-difference-between-datarequired-and-inputrequired)
[Regexp](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.Regexp) | 使用正则表达式验证输入值
[URL](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.URL) | 验证 URL
[AnyOf](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.AnyOf) | 确保输入值在可选值列表中
[NoneOf](https://wtforms.readthedocs.io/en/latest/validators.html#wtforms.validators.NoneOf) | 确保输入值不在可选值列表中

你还可以自定义验证函数: [Custom validators](https://wtforms.readthedocs.io/en/latest/validators.html#custom-validators) 和 [Setting flags on the field with validators](https://wtforms.readthedocs.io/en/latest/validators.html#setting-flags-on-the-field-with-validators)

<span id="渲染表单"></span>

__渲染表单：__

```python
{% raw %}
<form method="POST">
 {{ form.hidden_tag() }}
 {{ form.name.label }}  {{ form.name(id='my-text-field') }} # 为字段指定 id 或 class 属性，然后定义 CSS 样式
 {{ form.submit() }}
</form>

# 利用bootstrap的wtf快速渲染:
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}    # wtf.quick_form() 函数的参数为 Flask-WTF 表单对象，使用 Bootstrap 的默认样式渲染传入的表单

# 一个基本的表单:
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}
{% block title %}Flasky{% endblock %}
{% block page_content %}
<div class="page-header">
 <h1>Hello, {% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
</div>
{{ wtf.quick_form(form) }}
{% endblock %}
{% endraw %}
```

<span id="视图中处理表单"></span>

__在视图中处理表单：__

```python
# 如果没指定 methods 参数，默认注册为 GET 请求的处理程序.
@app.route('/', methods=['GET', 'POST'])
def index():
    name = None
    form = NameForm()
    if form.validate_on_submit():   # 返回值决定是重新渲染表单还是处理表单提交的数据.
      name = form.name.data
      form.name.data = ''
    return render_template('index.html', form=form, name=name)
```

<span id="重定向和会话"></span>

__重定向和会话：session__

```python
from flask import Flask, render_template, session, redirect, url_for

@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
        if form.validate_on_submit():
            session['name'] = form.name.data      # 在多次次请求之间记住输入的值
            return redirect(url_for('index'))     # 参数是重定向的 URL,推荐使用 url_for() 生成 URL，因为这个函数使用 URL 映射生成 URL，从而保证 URL 和定义的路由兼容，而且修改路由名字后依然可用。
            # url_for() 函数的第一个且唯一必须指定的参数是端点名，即路由的内部名字。默认情况下，路由的端点是相应视图函数的名字
        return render_template('index.html', form=form, name=session.get('name'))   # 直接从会话中读取 name 参数的值,不存在的键,get() 会返回默认值 None
```

<span id="flash消息"></span>

__flash消息:__

让用户知道状态发生了变化。参考: [flash-doc](http://www.pythondoc.com/flask/patterns/flashing.html)

```python
from flask import Flask, render_template, session, redirect, url_for, flash

@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            # 调用 flash() 函数，发给客户端的下一个响应中显示一个消息。
            # 仅调用 flash() 函数并不能把消息显示出来，模板还要渲染这些消息.
            flash('Looks like you have changed your name!') 
            session['name'] = form.name.data
            return redirect(url_for('index'))
    return render_template('index.html',
        form = form, name = session.get('name'))

# 渲染消息
{% raw %}
{% block content %}
<div class="container">
    {% for message in get_flashed_messages() %} # Flask 把 get_flashed_messages() 函数开放给模板，用来获取并渲染消息。
    # 在下次调用时不会再次返回，因此 Flash 消息只显示一次，然后就消失了.
        <div class="alert alert-warning">
            <button type="button" class="close" data-dismiss="alert">&times;</button>
            {{ message }}
        </div>
        {% endfor %}
    {% block page_content %}{% endblock %}
</div>
{% endblock %}
{% endraw%}
```


<span id="flask-SQLAlchemy"></span>

#### flask-SQLAlchemy

GitHub 项目地址: [flask-sqlalchemy](https://github.com/mitsuhiko/flask-sqlalchemy)

文档地址: [flask-sqlalchemy-doc](http://flask-sqlalchemy.pocoo.org/2.1/)

SQLAlchemy 文档地址: [sqlalchemy-doc](http://docs.sqlalchemy.org/en/latest/)

flask-sqlalchemy 中文文档翻译项目: [flask-sqlalchemy-doc-cn](https://github.com/pretdo/Flask-SQLAlchemy-cn)

<span id="数据库配置"></span>

__数据库URL和配置:__

__数据库引擎__ | __URL__
---------|-----
MySQL | mysql://username:password@hostname/database
Postgres | postgresql://username:password@hostname/database
SQLite（Unix） | sqlite:////absolute/path/to/database
SQLite（Windows） | sqlite:///c:/absolute/path/to/database

flask-SQLAlchemy需要的配置:

- SQLALCHEMY_DATABASE_URI 键: 使用的数据库 URL
- SQLALCHEMY_COMMIT_ON_TEARDOWN 键: 设为 True时，每次请求结束后都会自动提交数据库中的变动.
- 更多选项参考:[Configuration](http://flask-sqlalchemy.pocoo.org/2.1/config/)

```python
from flask.ext.sqlalchemy import SQLAlchemy
basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] ='sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
db = SQLAlchemy(app)
```

<span id="定义模型"></span>

__定义模型:__

参考另一个笔记:[SQLAlchemy]({% link _posts/programing/2017-02-25-python08.md %}#引入SQLAlchemy)

```python
class Role(db.Model):
    # 定义在数据库中使用的表名
    __tablename__ = 'roles'

    # db.Column 类构造函数的第一个参数是数据库列和模型属性的类型
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    
    def __repr__(self):
        return '<Role %r>' % self.name

class User(db.Model):
    # 定义在数据库中使用的表名
    __tablename__ = 'users'

    # db.Column 类构造函数的第一个参数是数据库列和模型属性的类型
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    
    # 返回一个具有可读性的字符串表示模型，可在调试和测试时使用
    def __repr__(self):
        return '<User %r>' % self.username
```

Flask-SQLAlchemy 要求每个模型都要定义主键，这一列经常命名为 id

Column类型文档:[Column-doc](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column)

<span id="常用的列类型"></span>

__常用的SQLAlchemy列类型:__

__类型名__ | __Python类型__ | __说明__
----------|---------------|----------
[Integer](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Integer) | int | 普通整数，一般是 32 位
[SmallInteger](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.SmallInteger) | int | 取值范围小的整数，一般是 16 位
[BigInteger](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.BigInteger) | int 或 long | 不限制精度的整数
[Float](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Float) | float | 浮点数
[Numeric](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Numeric) | decimal.Decimal | 定点数
[String](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.String) | str | 变长字符串
[Text](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Text) | str | 变长字符串，对较长或不限长度的字符串做了优化
[Unicode](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Unicode) | unicode | 变长 Unicode 字符串
[UnicodeText](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.UnicodeText) | unicode | 变长 Unicode 字符串，对较长或不限长度的字符串做了优化
[Boolean](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Boolean) | bool | 布尔值
[Date](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Date) | datetime.date | 日期
[Time](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Time) | datetime.time | 时间
[DateTime](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.DATETIME) | datetime.datetime | 日期和时间
[Interval](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Interval) | datetime.timedelta | 时间间隔
[Enum](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.Enum) | str | 一组字符串
[PickleType](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.PickleType) | 任何 Python 对象 | 自动使用 Pickle 序列化
[LargeBinary](http://docs.sqlalchemy.org/en/latest/core/type_basics.html#sqlalchemy.types.LargeBinary) | str | 二进制文件

更多类型参考：[type_basics](http://docs.sqlalchemy.org/en/latest/core/type_basics.html)

<span id="常用的列选项"></span>

__常使用的SQLAlchemy列选项:__

__选项名__ | __说明__
----------|----------
[primary_key](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column.params.primary_key) | 如果设为 True，这列就是表的主键
[unique](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column.params.unique) | 如果设为 True，这列不允许出现重复的值
[index](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column.params.index) | 如果设为 True，为这列创建索引，提升查询效率
[nullable](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column.params.nullable) | 如果设为 True，这列允许使用空值；如果设为 False，这列不允许使用空值
[default](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column.params.default) | 为这列定义默认值

更多列选项参考: [Column-params](http://docs.sqlalchemy.org/en/latest/core/metadata.html#sqlalchemy.schema.Column.__init__)

<span id="关系"></span>

__关系:__

使用关系把不同表中的行联系起来.

参考官网: [Relationship Configuration](http://docs.sqlalchemy.org/en/latest/orm/relationships.html)

一对多关系: [one-to-many](http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html#one-to-many)

```python
# 角色到用户的一对多关系，因为一个角色可属于多个用户，而每个用户都只能有一个角色
class Role(db.Model):
    # ...
    # 代表关系的面向对象视角,
    # 对于一个 Role 类的实例，其 users 属性将返回与角色相关联的用户组成的列表。
    # db.relationship() 的第一个参数表明这个关系的另一端是哪个模型。
    # 如果模型类尚未定义，可使用字符串形式指定。
    # backref 参数向 User 模型中添加一个 role 属性，从而定义反向关系。
    # 这一属性可替代 role_id 访问 Role 模型，此时获取的是模型对象，而不是外键的值。
    users = db.relationship('User', backref='role')
    
class User(db.Model):
    # ...
    # 定义为外键，建立起关系,
    # db.ForeignKey() 的参数 'roles.id' 表明，这列的值是 roles 表中行的 id 值
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```

- db.relationship() 都能自行找到关系中的外键，但有时却无法决定把哪一列作为外键。
- 例如，如果 User 模型中有两个或以上的列定义为 Role 模型的外键，SQLAlchemy 就不知道该使用哪列。
- 如果无法决定外键，你就要为 db.relationship() 提供额外参数，从而确定所用外键。

<span id="常用的关系选项"></span>

__常用的SQLAlchemy关系选项:__

__选项名__ | __说明__
----------|---------
[backref](http://docs.sqlalchemy.org/en/latest/orm/relationship_api.html#sqlalchemy.orm.relationship.params.backref) | 在关系的另一个模型中添加反向引用
[primaryjoin](http://docs.sqlalchemy.org/en/latest/orm/relationship_api.html#sqlalchemy.orm.relationship.params.primaryjoin) | 明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定lazy 指定如何加载相关记录。<br />可选值有 `select`（首次访问时按需加载）、`immediate`（源对象加载后就加载）、`joined` （加载记录，但使用联结）、<br />`subquery`（立即加载，但使用子查询），`noload`（永不加载）和 `dynamic`（不加载记录，但提供加载记录的查询）
[uselist](http://docs.sqlalchemy.org/en/latest/orm/relationship_api.html#sqlalchemy.orm.relationship.params.uselist) | 如果设为 Fales，不使用列表，而使用标量值
[order_by](http://docs.sqlalchemy.org/en/latest/orm/relationship_api.html#sqlalchemy.orm.relationship.params.order_by) | 指定关系中记录的排序方式
[secondary](http://docs.sqlalchemy.org/en/latest/orm/relationship_api.html#sqlalchemy.orm.relationship.params.secondary) | 指定多对多关系中关系表的名字
[secondaryjoin](http://docs.sqlalchemy.org/en/latest/orm/relationship_api.html#sqlalchemy.orm.relationship.params.secondaryjoin) | SQLAlchemy 无法自行决定时，指定多对多关系中的二级联结条件| 

其他关系:

- 多对一关系: [many-to-one](http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html#many-to-one)
- 一对一关系: [one-to-one](http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html#one-to-one)
- 多对多关系: [many-to-many](http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html#many-to-many)

<span id="数据库操作"></span>

__数据库操作:__

```python
# 创建表
(venv) $ python hello.py shell
>>> from hello import db
>>> db.drop_all()     #先删除旧表再重新创建，副作用，把数据库中原有的数据都销毁了.
>>> db.create_all()   #如果表已经存在于库中，那么db.create_all()不会重新创建或者更新这个表。

# 插入行
>>> from hello import Role, User
>>> admin_role = Role(name='Admin')     #使用关键字参数指定的模型属性初始值
>>> mod_role = Role(name='Moderator')
>>> user_role = Role(name='User')
>>> user_john = User(username='john', role=admin_role)  #使用关键字参数指定的模型属性初始值
>>> user_susan = User(username='susan', role=user_role) # role 属性也可使用，虽然它不是真正的数据库列，但却是一对多关系的高级表示。
>>> user_david = User(username='david', role=user_role) # 新建对象的 id属性并没有明确设定，因为主键是由 Flask-SQLAlchemy 管理的。插入数据库后就有了。

# 数据库会话[事务]，增加数据
>>> db.session.add(admin_role)
>>> db.session.add(mod_role)
>>> db.session.add(user_role)
>>> db.session.add(user_john)
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
# 或者这样增加:
>>> db.session.add_all([admin_role, mod_role, user_role,
... user_john, user_susan, user_david])
>>> db.session.commit()   # 提交会话:
>>> db.session.rollback() # 数据库库会话回滚，还原数据库状态.

# 修改行
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)    # 也能更新模型
>>> db.session.commit()

# 删除行
>>> db.session.delete(mod_role)   #删除与插入和更新一样，提交数据库会话后才会执行。
>>> db.session.commit() 
```

<span id="数据库查询"></span>

__数据库查询:__

参考官网文档: [Query-doc](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query)

```python
# Flask-SQLAlchemy 为每个模型类都提供了 query 对象
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>]
>>> User.query.all()
[<User u'john'>, <User u'susan'>, <User u'david'>]

# 过滤器可以配置 query 对象进行更精确的数据库查询
>>> User.query.filter_by(role=user_role).all()
[<User u'susan'>, <User u'david'>]

# 查看 SQLAlchemy 为查询生成的原生 SQL 查询语句
>>> str(User.query.filter_by(role=user_role))
'SELECT users.id AS users_id, users.username AS users_username,
users.role_id AS users_role_id FROM users WHERE :param_1 = users.role_id'

# 重新打开shell，从数据库中查询对象
>>> user_role = Role.query.filter_by(name='User').first()
# filter_by() 等过滤器在 query 对象上调用，返回一个更精确的 query 对象。多个过滤器可以一起调用，直到获得所需结果。
```

<span id="查询过滤器"></span>

__查询过滤器:__

__过滤器__ | __说　　明__
------|---------
[filter()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.filter) | 把过滤器添加到原查询上，返回一个新查询
[filter_by()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.filter_by) | 把等值过滤器添加到原查询上，返回一个新查询
[limit()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.limit) | 使用指定的值限制原查询返回的结果数量，返回一个新查询
[offset()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.offset) | 偏移原查询返回的结果，返回一个新查询
[order_by()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.order_by) | 根据指定条件对原查询结果进行排序，返回一个新查询
[group_by()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.group_by) | 根据指定条件对原查询结果进行分组，返回一个新查询

更多过滤器参考: [Query API](http://docs.sqlalchemy.org/en/latest/orm/query.html#)

<span id="执行查询"></span>

__执行查询函数:__

__方　法__ | __说　　明__
----------|-------------
[all()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.all) | 以列表形式返回查询的所有结果
[first()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.first) | 返回查询的第一个结果，如果没有结果，则返回 None
[first_or_404()](http://flask-sqlalchemy.pocoo.org/2.1/queries/#queries-in-views) | 返回查询的第一个结果，如果没有结果，则终止请求，返回 404 错误响应
[get()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.get) | 返回指定主键对应的行，如果没有对应的行，则返回 None
[get_or_404()](http://flask-sqlalchemy.pocoo.org/2.1/queries/#queries-in-views) | 返回指定主键对应的行，如果没找到指定的主键，则终止请求，返回 404 错误响应
[count()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.count) | 返回查询结果的数量
[paginate()](http://flask-sqlalchemy.pocoo.org/2.1/api/?highlight=paginate#flask.ext.sqlalchemy.BaseQuery.paginate) | 返回一个 Paginate 对象，它包含指定范围内的结果

```python
# 分别从关系的两端查询角色和用户之间的一对多关系
>>> users = user_role.users     #隐含的查询会调用 all() 返回一个用户列表,query 对象是隐藏的，因此无法指定更精确的查询过滤器.
>>> users
[<User u'susan'>, <User u'david'>]
>>> users[0].role
<Role u'User'>

# 修改关系的设置，加入了 lazy = 'dynamic' 参数，禁止自动执行查询。
class Role(db.Model):
    # ...
    users = db.relationship('User', backref='role', lazy='dynamic')
    # ...

# 样配置关系之后，user_role.users 会返回一个尚未执行的查询，因此可以在其上添加过滤器：
>>> user_role.users.order_by(User.username).all()
[<User u'david'>, <User u'susan'>]
>>> user_role.users.count()
2
```

<span id="视图中操作数据库"></span>

__视图函数中操作数据库:__

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        # 使用 filter_by() 查询过滤器在数据库中查找提交的名字
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username = form.name.data)
            db.session.add(user)
            session['known'] = False
        else:
            session['known'] = True
        session['name'] = form.name.data
        form.name.data = ''
        return redirect(url_for('index'))
    return render_template('index.html',
        form = form, name = session.get('name'),
        known = session.get('known', False))    #模板使用 known 参数
```

<span id="集成开发环境到shell"></span>

__集成开发环境到shell:__

```python
from flask.ext.script import Shell
def make_shell_context():
    # 注册了程序、数据库实例以及模型，这些对象将直接导入 shell
    return dict(app=app, db=db, User=User, Role=Role)

# 为 shell 命令注册一个 make_context 回调函数
manager.add_command("shell", Shell(make_context=make_shell_context))

# 打开Shell
$ python hello.py shell
>>> app
<Flask 'app'>
>>> db
<SQLAlchemy engine='sqlite:////home/flask/flasky/data.sqlite'>
>>> User
<class 'app.User'>
```

<span id="flask-migrate"></span>

#### flask-migrate

数据库迁移框架,数据库迁移框架能跟踪数据库模式的变化，然后增量式的把变化应用到数据库中

GitHub地址: [flask-migrate](https://github.com/miguelgrinberg/Flask-Migrate)
文档地址: [flask-migrate-doc](http://flask-migrate.readthedocs.io/en/latest/)

相关数据库迁移框架: [alembic-doc](http://alembic.zzzcomputing.com/en/latest/) 和 [GitHub地址](https://github.com/zzzeek/alembic)  和 [Bitbucket地址](https://bitbucket.org/zzzeek/alembic)

SQLAlchemy 的主力开发人员编写了一个迁移框架，称为 Alembic。除了直接使用 Alembic 之外，Flask 程序还可使用 Flask-Migrate扩展。这个扩展对 Alembic 做了轻量级包装，并集成到 Flask-Script 中，所有操作都通过 Flask-Script 命令完成。

```python
# 配置flask-migrate
from flask.ext.migrate import Migrate, MigrateCommand
# ...
migrate = Migrate(app, db)
manager.add_command('db', MigrateCommand)   #MigrateCommand 类，可附加到 FlaskScript 的 manager 对象上

# init 子命令创建迁移仓库
(venv) $ python hello.py db init
 Creating directory /home/flask/flasky/migrations...done
 Creating directory /home/flask/flasky/migrations/versions...done
 Generating /home/flask/flasky/migrations/alembic.ini...done
 Generating /home/flask/flasky/migrations/env.py...done
 Generating /home/flask/flasky/migrations/env.pyc...done
 Generating /home/flask/flasky/migrations/README...done
 Generating /home/flask/flasky/migrations/script.py.mako...done
 Please edit configuration/connection/logging settings in
 '/home/flask/flasky/migrations/alembic.ini' before proceeding.

# 会创建 migrations 文件夹，所有迁移脚本都存放其中。
```

__创建迁移脚本:__

- 在 Alembic 中，数据库迁移用迁移脚本表示。
- 脚本中有两个函数，分别是 upgrade() 和downgrade()。
- upgrade() 函数把迁移中的改动应用到数据库中，
- downgrade() 函数则将改动删除。
- Alembic 具有添加和删除改动的能力，因此数据库可重设到修改历史的任意一点。

- 可以使用 `revision` 命令手动创建 Alembic 迁移，也可使用 `migrate` 命令自动创建。
- 手动创建的迁移只是一个骨架，_upgrade()_ 和 _downgrade()_ 函数都是空的，开发者要使用Alembic 提供的 _Operations_ 对象指令实现具体操作。
- 自动创建的迁移会根据模型定义和数据库当前状态之间的差异生成 _upgrade()_ 和 _downgrade()_ 函数的内容。

__自动创建的迁移不一定总是正确的，有可能会漏掉一些细节。自动生成迁移脚本后一定要进行检查。__

```python
(venv) $ python hello.py db migrate -m "initial migration"
INFO [alembic.migration] Context impl SQLiteImpl.
INFO [alembic.migration] Will assume non-transactional DDL.
INFO [alembic.autogenerate] Detected added table 'roles'
INFO [alembic.autogenerate] Detected added table 'users'
INFO [alembic.autogenerate.compare] Detected added index
'ix_users_username' on '['username']'
 Generating /home/flask/flasky/migrations/versions/1bc
 594146bb5_initial_migration.py...done

# 更新数据库, 使用 db upgrade 命令把迁移应用到数据库中
(venv) $ python hello.py db upgrade
INFO [alembic.migration] Context impl SQLiteImpl.
INFO [alembic.migration] Will assume non-transactional DDL.
INFO [alembic.migration] Running upgrade None -> 1bc594146bb5, initial migration
```

数据库的设计和使用是很重要的话题，甚至有整本的书对其进行介绍,把这些介绍视作一个概览.

#### flask-mail

GitHub项目地址: [flask-mail](#https://github.com/mattupstate/flask-mail)

文档地址: [flask-mail-doc](#https://pythonhosted.org/Flask-Mail/)

flask-mail需要的配置:

__配 置__ | __默认值__ | __说 明__
-------|-------|---------
MAIL_SERVER | localhost | 电子邮件服务器的主机名或 IP 地址
MAIL_PORT | 25 | 电子邮件服务器的端口
MAIL_USE_TLS | False | 启用传输层安全（Transport Layer Security，TLS）协议
MAIL_USE_SSL | False | 启用安全套接层（Secure Sockets Layer，SSL）协议
MAIL_USERNAME | None | 邮件账户的用户名
MAIL_PASSWORD | None | 邮件账户的密码

初始化flask-mail:

```python
from flask.ext.mail import Mail
mail = Mail(app)

# 使用:
(venv) $ python hello.py shell
>>> from flask.ext.mail import Message
>>> from hello import mail
>>> msg = Message('test subject', sender='you@example.com',
... recipients=['you@example.com'])
>>> msg.body = 'text body'
>>> msg.html = '<b>HTML</b> body'
>>> with app.app_context():         #send() 函数使用 current_app，因此要在激活的程序上下文中执行。
... mail.send(msg)
...

# 集成发送电子邮件功能
from flask.ext.mail import Message

app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Flasky]'
app.config['FLASKY_MAIL_SENDER'] = 'Flasky Admin <flasky@example.com>'

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
        sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)   #使用 Jinja2 模板渲染邮件正文，灵活性极高
    msg.html = render_template(template + '.html', **kwargs)  #使用 Jinja2 模板渲染邮件正文，灵活性极高
    mail.send(msg)

# 在视图中扩展mail
# ...
app.config['FLASKY_ADMIN'] = os.environ.get('FLASKY_ADMIN')
# ...
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            session['known'] = False
            #每当表单接收新名字时，程序都会给管理员发送一封电子邮件
            if app.config['FLASKY_ADMIN']:
                send_email(app.config['FLASKY_ADMIN'], 'New User',
                           'mail/new_user', user=user)    # 两个模板文件都保存在 templates 文件夹下的 mail 子文件夹中，以便和普通模板区分开来
        else:
            session['known'] = True
        session['name'] = form.name.data
        form.name.data = ''
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'),
                            known=session.get('known', False))
```

__异步发送邮件:__

```python
from threading import Thread

def send_async_email(app, msg):
    # 在不同线程中执行 mail.send() 函数时，
    # 程序上下文要使用 app.app_context() 人工创建。
    # 会用到current_app中的配置.
    with app.app_context():
        mail.send(msg)

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
                sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)

    thr = Thread(target=send_async_email, args=[app, msg])      # 线程异步发送.
    thr.start()

    return thr
```

<span id="falsk-项目结构"></span>

#### falsk-项目结构

```shell
项目文件夹
.
├── app                       # flask程序目录
│   ├── static/               # 静态文件目录【flask默认】
│   ├── templates/            # 模版文件目录【flask默认】
│   ├── main/                 # 主程序包
│   │   ├── __init__.py       # 包声明
│   │   ├── errors.py         # 错误处理
│   │   ├── forms.py          # wtf表单类
│   │   └── views.py          # 视图层
│   ├── __init__.py           # 包声明
│   └── models.py             # 数据库声明
├── migrations/               # 数据库迁移仓库
├── venv/                     # pyton虚拟环境
├── tests/                    # 测试目录
│    ├── __init__.py          # 包声明
│    └── test_*.py            # 单元测试文件
├── requirements.txt          # 所有依赖包
├── config.py                 # 存储配置
├── manage.py                 # 用于启动程序以及其他的程序任务
├── LICENSE                   # 版权声明
└── README.md                 # 项目介绍
```

```python
#使用程序工厂函数
from flask import Flask
from flask_bootstrap import Bootstrap
from flask_mail import Mail
from flask_moment import Moment
from flask_sqlalchemy import SQLAlchemy
from config import config

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()


def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)

    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)

    # 使用蓝本注册
    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)

    return app

# 蓝本中错误处理
# 使用 errorhandler 修饰器，那么只有蓝本中的错误才能触发处理程序。
# 要注册程序全局的错误处理程序，必须使用 app_errorhandler
@main.app_errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@main.app_errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500

# 蓝本中定义路由
# 路由修饰器由蓝本提供
@main.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            session['known'] = False
            if current_app.config['FLASKY_ADMIN']:
                send_email(current_app.config['FLASKY_ADMIN'], 'New User',
                           'mail/new_user', user=user)
        else:
            session['known'] = True
        session['name'] = form.name.data
        return redirect(url_for('.index'))    # url_for() 函数的用法不同，第一个参数是路由的端点名，默认为视图函数的名字，
        # Flask 会为蓝本中的全部端点加上一个命名空间，
        # 这样就可以在不同的蓝本中使用相同的端点名定义视图函数，而不会产生冲突。
        # 命名空间就是蓝本的名字（Blueprint 构造函数的第一个参数），
        # 所以视图函数 index() 注册的端点名是 main.index，
        # 其 URL 使用 url_for('main.index') 获取。
        # 在蓝本中可以省略蓝本名，例如 url_for('.index')。
    return render_template('index.html',
                           form=form, name=session.get('name'),
                           known=session.get('known', False))
```

蓝本使用参考:[Blueprint](http://www.pythondoc.com/flask/blueprints.html)

__单元测试:__

参考官方文档: [unittest-doc](https://docs.python.org/2/library/unittest.html)

```python
import unittest
from flask import current_app
from app import create_app, db

class BasicsTestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()

    def test_app_exists(self):
        self.assertFalse(current_app is None)

    def test_app_is_testing(self):
        self.assertTrue(current_app.config['TESTING'])
```

__添加命令行测试:__

```python
from app import create_app
from flask_script import Manager

app = create_app(os.getenv('FLASK_CONFIG') or 'default')
manager = Manager(app)

@manager.command
def test():
    """Run the unit tests."""
    import unittest
    tests = unittest.TestLoader().discover('tests')
    unittest.TextTestRunner(verbosity=2).run(tests)

# 测试:
(venv) $ python manage.py test
test_app_exists (test_basics.BasicsTestCase) ... ok
test_app_is_testing (test_basics.BasicsTestCase) ... ok
.----------------------------------------------------------------------
Ran 2 tests in 0.001s
OK
```

__创建数据库:__

```python
# 创建数据表或者升级到最新修订版本
(venv) $ python manage.py db upgrade
```













