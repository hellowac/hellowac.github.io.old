---
layout:     post
title:      "Hello jekyll"
tags:
    - jekyll
categories: "编程"
---
## Jekyll

### 什么是jekyll
**jekyll**是一个静态网页托管服务。可以托管markdown，Textile，html类型的文件.并且**jekyll**也是github的静态页面托管服务.参考官网:[Jekyll.com](http://jekyllrb.com/)

### 快速开始
安装依赖:

```
$ gem install jekyll  //安装jekyll。gem执行文件可能需要其他依赖，比如ruby
$ gem install kramdown // markdown语言解析包
$ gem install pygments.rb // 代码高亮包
$ gem install liquid // 这个包我不知道，但是好像有用
// 还有些别的包，看需要进行安装
```
目录结构:

```
$ jekyll new myblog //生成博客目录
.
├── _config.yml
├── _data
|   └── members.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.md
|   └── on-simplicity-in-technology.md
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
|   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
|   ├── _base.scss
|   └── _layout.scss
├── _site
├── .jekyll-metadata
└── index.html # can also be an 'index.md' with valid YAML Frontmatter
```
启动服务:

```
$ jekyl serve
```
默认本地服务4000端口,然后打开[localhost:4000](http://localhost:4000)进行预览

### 基本用法
通过gem安装包管理器安装好jekyll以后，就能够在命令行中执行jekyll.
到项目根目录,执行编译命令:

```
$ cd myblog
$ jekyll build
```
生成的静态文件会存放在***_site***目录下.

编译到指定地方？

```
$ jekyll build --destination <destination>
```

编译指定目录？

```
$ jekyll build --source <source> --destination <destination>
```

监听文件变化?

```
$ jekyll build --watch
```
>***注意:*** jekyll 2.4版本以后会自动检测文件的改变. 如不监听:执行**jekyll serve --no-watch**,**编译到的目标文件夹会被清空**, 除了--no-watch等配置项，还有其他很多配置
一般是放在根目录下面的_config.xml文件下面，前面的放在命令行也是一种方式

调用***jekyll命令***的时候会自动用_config.xml里面的配置。比如:

**_config.xml**里的 **source:_source destination:_deploy** 相当于:

```
$ jekyll build --source _source --destination _deploy
```

### 目录结构
* **_config.yml:** 存储配置数据。把配置写在这个文件里面，可以让你不用在命令行中写。
* **_drafts:** 草稿，格式是:没有日期.md
* **_includes:** 包含一些模板，可以重复利用。你可以用通过{% raw %}{% include file.ext %}{% endraw %}包含_includes/file.ext文件{这种方式是[liquid](https://github.com/Shopify/liquid)语法}
* **_layouts:** 里面的文件通过{{ content }}包含_posts里面的文章。
* **_posts:** 存放你要发表的文章。格式YEAR-MONTH-DAY-title.MARKUP。文件名确定了发表的日期和标记语言。博客的日期格式通过_config.yml的[permalink](http://jekyllrb.com/docs/permalinks/)字段设置或者通过YAML FRONT Matter设置
* **_data:**保存数据的。jekyll会自动加载这里的所有.jml或者.yaml结尾的文件。比如你有一个<code>members.yml</code>。那么你可以通过**site.data**.<code>members</code>访问该文件里的数据。
* **_site:**jekyll生成的网站会放在该文件夹下。最好把它放到**.gitignore**文件里面，这样Git就不会管理它了。
* **index.html**:该文件里面有一个Yaml Front Matter。jekyll会转换它。包括所有的根目录下面的，或者不是以上提到的 目录。
里的<code>.html</code> <code>.markdown</code> <code.md<code> 和<code>.textile</code>文件。除了上面提到的其他文件或者文件夹，会被自动拷贝到_site文件夹里面。包括
css和img文件夹，favicon.icon文件。Yaml front matter 大概就长下面这样：

```
---
layout: index
title: FEX
page_id: index
varible: value  //可以自定义变量和值
....
---
```

### 配置
有两种方式配置: 一个是命令行，一个是通过_config.yml文件

* **safe:** 是否禁用自定义插件 不理解暂时不管他
* **source:** 定义jekyll读取文件的位置 比如本地就直接用.
* **destination:** 定义网站生成的位置，比如_site
* **encoding:** 通过名字定义文件的编码 只ruby1.9以后才有效, 2.0.0以后默认的编码是utf-8，而之前的默认编码是ascii-8bit
* **Front Matter defaults**
用这个东东可以具体地配置你的页面或者发表的文章。

你可能会重复配置一些东西，比如作者，为了避免这种情况的解决方案是 在_config.xml中配置defaults字段

默认配置:参考官网：<http://jekyllrb.com/docs/configuration/>

Front Matter
通过这个可以设置一些变量（甚至可以自定义变量），比如title

```
---
layout: post
title: Blogging Like a Hacker
---
```
设置好变量以后，
你就可以在当前页面或者你的页面依赖的_layouts或者_includes
里的文件通过Liquid 标记，比如 {% raw %}{page.title}{% endraw %} 访问了。

>不允许存在BOM字符 ，这个头不写也是可以的：比如CSS and RSS feeds!可能不需要。

**已定义的全局变量:**

**layout:**指定用_layouts下面的文件。

**自定义的变量:**

**在Front Matter定义的变量**（不是已定义的全局变量）都会在会话期间绑定数据给Liquid模板引擎。
比如你在定义了<code>title</code>，那你就可以再<code>_layouts</code>里面的模板使用它<code>{% raw %}{{ page.title }}{% endraw %}</code>

### 写文章
jekyll有一个最好的特性就是：你写文章并发表他们只是意味着你只要管理一些文本文件即可。
而不需要配置和维护数据库以及良好的CMS系统。

发文章的文件夹:**_posts**里面都是些<code>md</code>或者<code>testile</code>文件。只要有<code>yaml front matter</code>，它们就会被转换为html格式的静态页面。

创建文件：创建一个文件<code>YEAR-MONTH-DAY-title.md</code> <code>YEAR</code>是4位数，<code>month</code>和<code>day</code>是两位数,例如:<code><i>2011-12-31-new-years-eve-is-awesome.md</i></code><code><i>2012-09-12-how-to-write-a-blog.textile</i></code>

>* 通过post_url访问其他的文章，不用担心链接（ the site permalink）的样式改变而访问不到。
>* 所有的文章都必须要有yaml front matter头。
>* 将\<meta charset="utf-8"\>包含在head标签里面。

#### 包含图片和资源
在根目录下创建文件夹比如assets和downloads然后markdown语法访问通过这种形式：

```
![截屏]({% raw %}{{ site.url }}{% endraw %}/assets/screenshot.jpg)
```
说明：

* 截屏链接的文字，（）里面的东西不会显示，是链接到的地址。
* site.url可以访问你配置的（_config.yml）的网站url。
* 链接到pdf阅读器让用户下载：
* 
```
PDF [下载]({% raw %}{{ site.url }}{% endraw %}/assets/mydoc.pdf) directly.
```

**显示一系列的文章:**

```
<ul>
  {% raw %}{% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}{% endraw %}
</ul>
```
**显示文章的第一个段:**

```
<ul>
  {% raw %}{% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}{% endraw %}
</ul>
```
**高亮代码片段:**

```
//通过Pygments or Rouge，jekyll具有内建的语法高亮能力
{% raw %}{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}{% endraw %}
```

**高亮代码同时显示行号:**

```
{% raw %}
{% highlight ruby linenos %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}
{% endraw %}
```

**_drafts文件夹工作方式:**

* *_drafts*里面的文章是你暂时不想发表的
* 通过 <code>jekyll serve --drafts</code>预览
* <code>jekyll build --drafts</code> 编译


### 创建页面
即使是主页[index.html]也可以用_layouts和_includes里面的东西 以及 其他的额外页面

**一般方式:**

```
|-- _config.yml
|-- _includes/
|-- _layouts/
|-- _posts/
|-- _site/
|-- about.html    # => http://example.com/about.html
|-- index.html    # => http://example.com/
└── contact.html  # => http://example.com/contact.html
```
**干净的url方式（不带有文件后缀）:**

```
├── _config.yml
├── _includes/
├── _layouts/
├── _posts/
├── _site/
├── about/
|   └── index.html  # => http://example.com/about/
├── contact/
|   └── index.html  # => http://example.com/contact/
└── index.html      # => http://example.com/
```

### 变量
jekyll会遍历所有的文件，只要带有yaml front matter的文件 都可以通过Liquid 模板系统访问一些变量。

**全局变量：**

* **site:** 包含了网站信息和_config.yml里面的信息
* **page:** 在yaml front matter的自定义的变量通过page访问
* **content:** 作用于<code>\_layouts</code>里面，不作用在_post和其他页面中。包含了post和其他页面里面的文章内容。
* **paginator:** paginate在<code>\_config.yml</code>里面配置以后，这个变量就可以用了。

**site变量:**

* **site.time:** 当前运行jekyll的时间
* **site.pages:** 所有的页面
* **site.posts:** 以时间逆序排序的所有的文章
* **site.data:** 包含从目录_data里面加载的数据列表

**page变量：**

* **page.content:** 页面内容
* **page.title:** 文章标题
* **page.url:** 页面地址：比如/2008/12/14/my-post.html
* **page.date:** 页面的日期。可以在front matter重写：2008-12-14 10:30:00 +0900或者YYYY-MM-DD HH:MM:SS
* **page.id:** 页面id。比如/2008/12/14/my-post 在RSS feeds里面有用。

> front matter里面可以自己定义变量：比如custom_css: true , 然后你可以通过page.custom_css访问

**Paginator变量:**

* **paginator.per_page:** 每一页的文章数
* **paginator.posts:** 那一页可用的文章
* **paginator.page:** 当前页的值

>Paginator只在index.html(或者/blog/index.html)中有效 

### 自定义数据

* 通过<code>[liquid](https://github.com/Shopify/liquid)</code>模板系统可以自定义数据.
* **jekyll**支持从位于<code>_data</code>的<code>yaml</code><code>json</code><code>csv</code>文件中加载数据，（<code>csv</code>必须包含一个 <code>header row</code>）
* 通过**site.data**访问里面的数据

**例如:** 定义一个<code>_data/members.yml</code>文件

```
- name: Tom Preston-Werner
  github: mojombo

- name: Parker Moore
  github: parkr

- name: Liu Fengyun
  github: liufengyun
```
然后可以通过<code>site.data.members</code>访问该文件（文件名决定了字段名）

```
<ul>
{% raw %} {% for member in site.data.members %}
  <li>
    <a href="https://github.com/{{ member.github }}">
      {{ member.name }}
    </a>
  </li>
{% endfor %}{% endraw %} 
</ul>
```

**定义组织(包含子文件)**

<code>_data/orgs/jekyll.yml</code>中：

```
username: jekyll
name: Jekyll
members:
  - name: Tom Preston-Werner
    github: mojombo

  - name: Parker Moore
    github: parkr
```
<code>_data/orgs/doeorg.yml</code>中：

```
username: doeorg
name: Doe Org
members:
  - name: John Doe
    github: jdoe
```
**使用:**

```
<ul>
{% raw %}{% for org_hash in site.data.orgs %}
{% assign org = org_hash[1] %}
  <li>
    <a href="https://github.com/{{ org.username }}">
      {{ org.name }}
    </a>
    ({{ org.members | size }} members)
  </li>
{% endfor %}{% endraw %}
</ul>
```
