---
layout: default
title: Categories
permalink: /categories/
---

{% comment %}Collect all unique categories from posts{% endcomment %}
{% assign all_categories = "" %}

{% for post in site.posts %}
{% if post.categories %}
{% assign raw_categories = post.categories | join: " " %}
{% assign clean_categories = raw_categories | replace: "[", "" | replace: "]", "" | replace: '"', "" %}
{% assign category_array = clean_categories | split: " " %}
{% for category in category_array %}
{% assign all_categories = all_categories | append: category | append: "," %}
{% endfor %}
{% endif %}
{% endfor %}

{% assign unique_categories = all_categories | split: "," | uniq | sort %}

{% for category in unique_categories %}
{% if category != "" %}

## {{ category }}

<ul>
{% for post in site.posts %}
  {% if post.categories %}
    {% assign raw_categories = post.categories | join: " " %}
    {% assign clean_categories = raw_categories | replace: "[", "" | replace: "]", "" | replace: '"', "" %}
    {% assign category_array = clean_categories | split: " " %}
    {% if category_array contains category %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>({{ post.date | date: "%Y-%m-%d" }})</small>
    </li>
    {% endif %}
  {% endif %}
{% endfor %}
</ul>

{% endif %}
{% endfor %}
