---
title: "Experience"
permalink: /experience/
layout: archive
author_profile: true
classes: wide
---

## TRI System Integration Engineer

{% include experience/tri-engineer-body.md %}

## TRI Intern

{% include experience/tri-intern-body.md %}

<div class="entries-grid">
  {% for post in site.posts %}
    {% if post.group == "experience" %}
      {% unless post.title == "TRI Intern" or post.title == "TRI System Integration Engineer" %}
        {% include archive-single.html type="grid" %}
      {% endunless %}
    {% endif %}
  {% endfor %}
</div>
