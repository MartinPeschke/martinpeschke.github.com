---
layout: default
title: Welcome
---
<ul id="posts" class="index post-overview">
  {% for post in site.posts %}
    <li>
      <article>
      <h2>{{ post.title | xml_escape }}</a></h2>
          <span class="permalink"><a href="{{ post.permalink }}">permalink</a></span>
          </span>
          <span class="datetime">
            <time datetime="{{ post.date | date: "%Y-%m-%d" }}">
              {{ post.date | date: "%B %d, %Y" }}
            </time>
          </span>
        {{ post.content }}
      </article>
    </li>
  {% endfor %}
</ul>