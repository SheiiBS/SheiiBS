---
layout: page_en
title: Blog
permalink: /en/blog/
translate_es: /blog/
sidebar: yes
promo: yes
---

Here is where I share my thoughts on DevOps and Configuration Management.

<div class="posts">

  {% assign posts_en = site.posts | where:"lang","en" %}
  {% for post in posts_en %}
    <article class="post">
    
      <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>

      <div class="entry">
        {{ post.excerpt }}
      </div>

      <div class="date">
        Published {{ post.date | date: "%B %e, %Y" }}
      </div>

      <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read more ...</a>
    </article>
  {% endfor %}
</div>