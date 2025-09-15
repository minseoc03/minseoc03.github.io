---
layout: single
permalink: /ml/dl_theory/
title: "DL Theory"
author_profile: false
sidebar:
  nav: "ml"
---

{%- assign ml_dlt_posts = site.posts
    | where_exp: "p", "p.categories contains 'ml' and p.categories contains 'dl_theory'"
    | sort: "date" | reverse -%}

{%- for post in ml_dlt_posts -%}
  {%- include archive-single.html type="post" -%}
{%- endfor -%}
