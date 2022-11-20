---
layout: list
title: All categories
menu: [articles, categories]
---

{% for categoryrecord in site.categories %}
  {% assign category = categoryrecord[0] %}
  {% assign posts = categoryrecord[1] %}
- ## {{ category }} <span class="counter">{{ posts.size}}</span>
  {% for post in posts %}
  - [{{ post.title }}]({{ post.url }})
  {% endfor %}
{% endfor %}
