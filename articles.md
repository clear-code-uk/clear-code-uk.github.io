---
title: Articles
menu: articles
---

## Categories

{% for categoryrecord in site.categories %}
  {% assign category = categoryrecord[0] %}
  {% assign posts = categoryrecord[1] %}
### {{ category }}
  {% for post in posts %}
- [{{ post.title }}]({{ post.url }})
  {% endfor %}
{% endfor %}

<hr/>

## Tags

{% for tagrecord in site.tags %}
  {% assign tag = tagrecord[0] %}
  {% assign posts = tagrecord[1] %}
### {{ tag }}
  {% for post in posts %}
- [{{ post.title }}]({{ post.url }})
  {% endfor %}
{% endfor %}

<hr/>

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
