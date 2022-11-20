---
layout: list
title: Other articles
menu: [articles, other]
---

{% for categoryrecord in site.categories %}
  {% assign category = categoryrecord[0] %}
  {% assign posts = categoryrecord[1] %}
  {% unless category == "Code" or category == "Design" %}
- ## {{ category }} <span class="counter">{{ posts.size}}</span>
    {% for post in posts %}
  - [{{ post.title }}]({{ post.url }})
    {% endfor %}
  {% endunless %}
{% endfor %}
