---
layout: list
title: All articles
menu: [articles, all]
---

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
