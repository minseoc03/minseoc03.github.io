---
layout: single
permalink: /ml/math/
title: "Math"
author_profile: false
sidebar:
  nav: "ml"
---

{%- assign ml_math_posts = site.posts
    | where_exp: "p", "p.categories contains 'ml' and p.categories contains 'math'"
    | sort: "date" | reverse -%}

{%- for post in ml_math_posts -%}
  {%- include archive-single.html type="post" -%}
{%- endfor -%}
