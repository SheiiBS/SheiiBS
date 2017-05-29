---
layout: page
title: Etiquetas
permalink: /blog/etiquetas/
sidebar: yes
translate_en: /en/blog/tags/
promo: yes
---

Publicaciones ordenadas por etiquetas.

<ul class="categories-tags-page">
	{% for tag in site.tags %}

		{% assign posts_es = tag[1] | where:"lang","es" %}
		{% if posts_es.size > 0 %}
			<li  id="{{ tag[0] }}">{{ tag[0] }}</li>
			{% for post in posts_es %}
				<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;
				<a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />

			{% endfor %}
		{% endif %}
	{% endfor %}
</ul>