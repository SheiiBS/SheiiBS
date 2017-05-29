---
layout: page
title: Categorías
permalink: /blog/categorias/
sidebar: yes
translate_en: /en/blog/categories/
promo: yes
---

Publicaciones ordenadas por categoría.

<ul class="categories-tags-page">

    <li id="articulo">Arctículo</li>

    {% for post in site.categories.articulo %}

      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

    <li id="articulo">Configuración</li>

    {% for post in site.categories.configuracion %}

      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

    <li id="articulo">Diagrama</li>

    {% for post in site.categories.diagrama %}

      <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;

      <a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />
    
    {% endfor %}

</ul>