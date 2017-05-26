---
layout: page_en
title: Tags
permalink: /en/blog/tags/
sidebar: yes
translate_es: /blog/etiquetas/
promo: yes
---

Publications sorted by tags.

<ul class="categories-tags-page">
	{% for tag in site.tags %}

		{% assign posts_en = tag[1] | where:"lang","en" %}
		{% if posts_en.size > 0 %}
			<li  id="{{ tag[0] }}">{{ tag[0] }}</li>
			{% for post in posts_en %}
				<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time> &raquo;
				<a href="{{ site.baseurl }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a><br />

			{% endfor %}
		{% endif %}
	{% endfor %}
</ul>