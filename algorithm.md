---
layout: page
title: Algorithm
permalink: /algorithm/
---
<h2>Algorithm Posts</h2>

<ul class="post-list">
  {% for post in site.posts %}
    {% if post.tags contains "Algorithm" %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <span class="post-date">{{ post.date | date: "%Y/%m/%d" }}</span>
      </li>
    {% endif %}
  {% endfor %}
</ul>
