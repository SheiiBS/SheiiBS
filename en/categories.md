---
layout: page_en
title: Categories
permalink: /en/blog/categories/
sidebar: yes
translate_es: /blog/categorias/
promo: yes
---

Publications sorted by category.

<ul class="categories-tags-page">

    <li id="articulo">Article</li>

    {% for post in site.categories.article %}
  
      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

    <li id="articulo">Configuration</li>

    {% for post in site.categories.configuration %}
  
      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

    <li id="articulo">Diagram</li>

    {% for post in site.categories.diagram %}
  
      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

</ul>