---
layout: default
title: Blog
---

<style>
.project-list { list-style: none; padding: 0; margin: 0; }
.project-card { display: flex; gap: 1.25rem; padding: 1.25rem 0; border-bottom: 1px solid #e0ddd6; align-items: flex-start; }
.project-card img { width: 140px; height: 95px; object-fit: cover; border-radius: 5px; flex-shrink: 0; }
.project-card .placeholder { width: 140px; height: 95px; border-radius: 5px; background: #eeece7; flex-shrink: 0; }
.project-info h2 { font-size: 1rem; margin: 0 0 0.3rem; }
.project-info h2 a { color: #1a1917; text-decoration: none; }
.project-info h2 a:hover { color: #2c5f8a; }
.project-meta { font-size: 0.78rem; color: #6b6860; margin-bottom: 0.4rem; }
.project-excerpt { font-size: 0.87rem; color: #3d3b35; line-height: 1.6; }
</style>

# Blog

<ul class="project-list">
{% assign sorted = site.projects | sort: 'date' | reverse %}
{% for project in sorted %}
<li class="project-card">
  {% if project.image %}
  <img src="{{ project.image }}" alt="{{ project.title }}">
  {% else %}
  <div class="placeholder"></div>
  {% endif %}
  <div class="project-info">
    <h2><a href="{{ project.url | prepend: site.baseurl }}">{{ project.title }}</a></h2>
    <div class="project-meta">{{ project.date | date: "%B %d, %Y" }}{% if project.tech %} · {{ project.tech | join: " · " }}{% endif %}</div>
    {% if project.excerpt %}<p class="project-excerpt">{{ project.excerpt }}</p>{% endif %}
  </div>
</li>
{% endfor %}
</ul>