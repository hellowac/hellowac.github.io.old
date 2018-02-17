---
nav: blog
layout: post
title: "日记"
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
{{ site.nav.blog.subnav.diary.name }}

{% for category in site.categories %}

{% if category[0] == '日记' %}

{% for post in category[1] %}
## [{{ post.title }}]({{ post.url }})
{% endfor %}

{% endif %}

{% endfor %}