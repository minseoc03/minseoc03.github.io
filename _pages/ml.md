---
layout: single
permalink: /ml/
title: "ML"
author_profile: false
sidebar:
  nav: "ml"
---

{%- assign ml_posts = site.posts
    | where_exp: "p", "p.categories contains 'ml'"
    | sort: "date" | reverse -%}

{%- for post in ml_posts -%}
  {%- include archive-single.html type="post" -%}
{%- endfor -%}
