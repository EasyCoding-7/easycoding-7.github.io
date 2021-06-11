---
layout: page
title: confusing
permalink: /blog/categories/confusing/
---

<h5> Posts by Category : {{ page.title }} </h5>

<div class="card">
{% for post in site.categories.헷깔헷깔시리즈 %}
 <li class="category-posts"><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</div>