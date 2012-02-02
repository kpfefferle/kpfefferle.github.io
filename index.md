---
layout: page
title: Welcome
---
{% include JB/setup %}

{% for post in site.posts limit:5 %}
  <h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a><br />
  <small>{{ post.date | date_to_string }}</small></h3>
  <p>{{ post.excerpt }}</p>
{% endfor %}
