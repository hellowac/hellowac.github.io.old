---
nav: blog
layout: post
title: "阅读"
---

[{{ site.nav.home.name }}]({% link index.md %})/
[{{ site.nav.blog.name }}]({% link blog/index.md %})/
{{ site.nav.blog.subnav.reading.name }}

{% for category in site.categories %}

{% if category[0] == 'Reading book' %}

{% for post in category[1] %}
## [{{ post.title }}]({{ post.url }})
{% endfor %}

{% endif %}

{% endfor %}