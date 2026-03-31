---
layout: default
title: Blog
permalink: /blog/
---

<h1>Blog</h1>

<h2 class="section-title">Perception</h2>
<div class="grid">
{% for post in site.posts %}
  {% if post.category == "perception" %}
  <div class="card">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p>{{ post.date | date: "%B %d, %Y" }}</p>
    {% if post.excerpt %}<p>{{ post.excerpt }}</p>{% endif %}
  </div>
  {% endif %}
{% endfor %}
</div>

<h2 class="section-title">Mapping</h2>
<div class="grid">
{% for post in site.posts %}
  {% if post.category == "mapping" %}
  <div class="card">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p>{{ post.date | date: "%B %d, %Y" }}</p>
    {% if post.excerpt %}<p>{{ post.excerpt }}</p>{% endif %}
  </div>
  {% endif %}
{% endfor %}
</div>

<h2 class="section-title">Control</h2>
<div class="grid">
{% for post in site.posts %}
  {% if post.category == "control" %}
  <div class="card">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p>{{ post.date | date: "%B %d, %Y" }}</p>
    {% if post.excerpt %}<p>{{ post.excerpt }}</p>{% endif %}
  </div>
  {% endif %}
{% endfor %}
</div>

<h2 class="section-title">Motion Planning</h2>
<div class="grid">
{% for post in site.posts %}
  {% if post.category == "motion-planning" %}
  <div class="card">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p>{{ post.date | date: "%B %d, %Y" }}</p>
    {% if post.excerpt %}<p>{{ post.excerpt }}</p>{% endif %}
  </div>
  {% endif %}
{% endfor %}
</div>