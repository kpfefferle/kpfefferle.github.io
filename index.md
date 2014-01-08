---
layout: page
title: Welcome
tagline:
---
{% include JB/setup %}

{% for post in site.posts limit:25 %}
  <h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a> <small>{{ post.date | date:"%Y-%m-%d" }}</small></h3>
  <p>{{ post.excerpt }}</p>
{% endfor %}
