---
nav: blog
layout: post
title: "阅读分类"
---

{% for category in site.categories %}

{% if category[0] == 'Reading book' %}

{% for post in category[1] %}
## [{{ post.title }}]({{ post.url | absolute_url }})
{% endfor %}

{% endif %}

{% endfor %}