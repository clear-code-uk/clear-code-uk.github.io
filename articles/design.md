---
layout: list
title: Design
menu: [articles, Design]
---

{% for categoryrecord in site.categories %}
  {% assign category = categoryrecord[0] %}
  {% assign posts = categoryrecord[1] %}
  {% if category == "Design" %}
    {% for post in posts %}
- [{{ post.title }}]({{ post.url }})
    {% endfor %}
  {% endif %}
{% endfor %}
