---
layout: default
title: Welcome
---
<ul id="posts" class="index post-overview">
  {% for post in site.posts %}
    <li>
      <i class="icon-arrow-right"></i>
      <a href="{{ post.url }}">{{ post.title | xml_escape }}</a>
      <span class="datetime">
      	<time datetime="{{ post.date | date: "%Y-%m-%d" }}">
      		{{ post.date | date: "%B %d, %Y" }}
      	</time>
      </span>
    </li>
  {% endfor %}
</ul>