---
layout: default
title: Blog
---

# Documented Projects

{% for project in site.projects %}

## [{{ project.title }}]({{ project.url | project.url }})

<small>{{ project.date | date: "%B %d, %Y" }}</small>

{% endfor %}