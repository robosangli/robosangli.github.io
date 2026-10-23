---
layout: archive
author_profile: true
entries_layout: grid
classes: wide
---

<h3 class="archive__subtitle">Featured projects</h3>
<div class="entries-grid">
  {% for order in (1..4) %}
    {% assign post = site.posts | where: "featured_order", order | first %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
</div>

<h3 class="archive__subtitle">Experience</h3>
<div class="entries-grid">
  {% assign post = site.posts | where: "title", "TRI System Integration Engineer" | first %}
  {% include archive-single.html type="grid" %}
  {% assign post = site.posts | where: "title", "TRI Intern" | first %}
  {% include archive-single.html type="grid" %}
</div>

<h3 class="archive__subtitle">Additional projects</h3>
<div class="entries-grid">
  {% for post in site.posts %}
    {% if post.group == "project" %}
      {% unless post.featured_order %}
        {% include archive-single.html type="grid" %}
      {% endunless %}
    {% endif %}
  {% endfor %}
</div>

<h3 class="archive__subtitle">About</h3>

{% include about-body.md %}
