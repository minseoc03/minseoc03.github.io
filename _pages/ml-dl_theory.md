---
layout: single
permalink: /ml/dl_theory/
title: "DL Theory"
author_profile: false
sidebar:
  nav: "ml"
---

{% for post in site.posts %}
  {% if post.categories contains "ml" and post.categories contains "dl_theory" %}
    {% include archive-single.html type="post" %}
  {% endif %}
{% endfor %}
