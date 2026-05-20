---
layout: default
title: Blog
description: This is where Thuyen Code shares his thoughts
---

# Posts:

{% if site.posts.size > 0 %}
{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
{% else %}
No posts yet.
{% endif %}
