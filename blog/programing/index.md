---
nav: blog
layout: post
title: "编程"
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
{{ site.nav.blog.subnav.programing.name }}

{% for category in site.categories %}

{% if category[0] == 'Programing Teach' %}

{% for post in category[1] %}
## [{{ post.title }}]({{ post.url | absolute_url }})
{% endfor %}

{% endif %}

{% endfor %}