---
layout: list
title: Code
menu: [articles, Code]
---

{% for categoryrecord in site.categories %}
  {% assign category = categoryrecord[0] %}
  {% assign posts = categoryrecord[1] %}
  {% if category == "Code" %}
    {% for post in posts %}
- [{{ post.title }}]({{ post.url }})
    {% endfor %}
  {% endif %}
{% endfor %}
