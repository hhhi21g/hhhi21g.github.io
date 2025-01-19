---
layout: page
title: Deep Learning
permalink: /deep-learning/
---
<h2>Deep Learning Posts</h2>

<ul class="post-list">
  {% for post in site.posts %}
    {% if post.tags contains "DeepLearning" %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <span class="post-date">{{ post.date | date: "%Y/%m/%d" }}</span>
      </li>
    {% endif %}
  {% endfor %}
</ul>
