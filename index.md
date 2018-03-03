---
nav: home
layout: default
title: ""
---

{% comment %} 
最多输出几次
{% endcomment %}

{% for post in site.posts limit:20 %}
{% if post.show %}

<div class="post-preview">
  <a href="{{ post.url }}">
    <h2 class="post-title"> {{ post.title }}</h2>
    <h3 class="post-subtitle"></h3>
  </a>
  <p class="post-meta">Posted by
    <a href="#">hellowac</a>
    on {{ post.date | date_to_long_string }}</p>
</div>
<hr>

{% endif %}
{% endfor %}





