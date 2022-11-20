---
layout: list
title: All tags
menu: [articles, tags]
---

{% for tagrecord in site.tags %}
  {% assign tag = tagrecord[0] %}
  {% assign posts = tagrecord[1] %}
- ## {{ tag }} <span class="counter">{{ posts.size}}</span>
  {% for post in posts %}
  - [{{ post.title }}]({{ post.url }})
  {% endfor %}
{% endfor %}
