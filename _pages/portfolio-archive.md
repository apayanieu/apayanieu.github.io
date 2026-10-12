---
layout: archive
title: "Projects"
permalink: /portfolio/
author_profile: true
---

A selection of quantitative, computational, and statistical projects I've completed during my studies in Economics and Applied Mathematics. Each project includes a full write-up, methodology, and links to the code on GitHub.

---

{% for post in site.portfolio %}
  {% include archive-single.html type="grid" %}
{% endfor %}
