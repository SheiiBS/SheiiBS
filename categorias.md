---
layout: page
title: Categorías
permalink: /blog/categorias/
sidebar: yes
translate_en: /en/blog/categories/
---

Publicaciones ordenadas por categoría.

<ul class="tags-box">

    <li id="articulo">Arctículo</li>

    {% for post in site.categories.articulo %}

      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

</ul>