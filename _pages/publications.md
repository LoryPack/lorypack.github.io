---
layout: page
permalink: /publications/
title: publications
description: My academic publications
years: [2025, 2024, 2022, 2021, 2020]
nav: true
importance: 1
---

<div class="publications">

{% for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
