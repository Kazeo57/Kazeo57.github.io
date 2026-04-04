<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>{{ page.title }} | Johannes Hounton</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link href="https://fonts.googleapis.com/css2?family=Source+Serif+4:wght@400;600&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    :root {
      --bg: #f7f6f2; --surface: #ffffff; --border: #e0ddd6;
      --text: #1a1917; --muted: #6b6860; --accent: #2c5f8a;
    }
    body { background: var(--bg); color: var(--text); font-family: 'DM Sans', sans-serif; font-weight: 300; line-height: 1.7; font-size: 15px; }
    a { color: var(--accent); text-decoration: none; }
    a:hover { text-decoration: underline; }

    .top-bar {
      background: var(--surface);
      border-bottom: 1px solid var(--border);
      padding: 0.75rem 2rem;
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 0.85rem;
    }
    .top-bar .site-name { font-weight: 500; color: var(--text); }

    .hero {
      width: 100%;
      height: 280px;
      object-fit: cover;
      display: block;
    }
    .hero-placeholder {
      width: 100%;
      height: 280px;
      background: #eeece7;
    }

    .container {
      max-width: 760px;
      margin: 0 auto;
      padding: 2.5rem 1.5rem 4rem;
    }

    .project-meta {
      font-size: 0.8rem;
      color: var(--muted);
      margin-bottom: 0.5rem;
      letter-spacing: 0.03em;
    }

    h1 {
      font-family: 'Source Serif 4', serif;
      font-size: 1.9rem;
      font-weight: 600;
      line-height: 1.25;
      margin-bottom: 0.75rem;
    }

    .tech-tags {
      display: flex; flex-wrap: wrap; gap: 0.4rem;
      margin-bottom: 2rem;
      padding-bottom: 1.5rem;
      border-bottom: 1px solid var(--border);
    }
    .tag {
      background: #eeece7; color: #3d3b35;
      font-size: 0.78rem; padding: 0.2rem 0.6rem;
      border-radius: 3px;
    }

    .content h2 {
      font-family: 'Source Serif 4', serif;
      font-size: 1.2rem; font-weight: 600;
      margin: 2rem 0 0.75rem;
    }
    .content h3 { font-size: 1rem; font-weight: 500; margin: 1.5rem 0 0.5rem; }
    .content p { margin-bottom: 1rem; font-size: 0.93rem; color: #2e2c28; }
    .content ul, .content ol { padding-left: 1.5rem; margin-bottom: 1rem; font-size: 0.93rem; color: #2e2c28; }
    .content li { margin-bottom: 0.3rem; }
    .content blockquote {
      border-left: 3px solid var(--border);
      padding-left: 1rem; margin: 1.5rem 0;
      color: var(--muted); font-style: italic;
    }
    .content code {
      background: #eeece7; padding: 0.15rem 0.4rem;
      border-radius: 3px; font-size: 0.85rem;
    }
    .content pre {
      background: #eeece7; padding: 1rem;
      border-radius: 5px; overflow-x: auto;
      margin-bottom: 1rem;
    }
    .content pre code { background: none; padding: 0; }
  </style>
</head>
<body>

<div class="top-bar">
  <span class="site-name">Johannes Hounton</span>
  <a href="/blog">← Back to Blog</a>
</div>

{% if page.image %}
<img class="hero" src="{{ page.image }}" alt="{{ page.title }}">
{% else %}
<div class="hero-placeholder"></div>
{% endif %}

<div class="container">
  <div class="project-meta">{{ page.date | date: "%B %d, %Y" }}</div>
  <h1>{{ page.title }}</h1>

  {% if page.tech %}
  <div class="tech-tags">
    {% for t in page.tech %}<span class="tag">{{ t }}</span>{% endfor %}
  </div>
  {% endif %}

  <div class="content">
    {{ content }}
  </div>
</div>

</body>
</html>