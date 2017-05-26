---
layout: post_en
title: Deployment Diagram - Django with Docker
permalink: /en/blog/deployment-diagram-docker-django/
translate_es: /blog/diagrama-despliegue-docker-django/
lang: en
sidebar: yes
promo: yes
category: [diagram]
tags: [django, uml, docker]
image: /images/banners/docker-og.png
excerpt: <strong><em>Deployment Diagram</em></strong> showing similarities and differences between <strong><em>development and production environments</em></strong> in <strong><em>Django projects</em></strong> using <strong><em>Docker</em></strong>.
---

<img src="{{ site.baseurl }}/images/banners/django-docker.png" title="Docker - Django" name="Docker - Django" />

## Deployment Diagram - Development environment
<img src="{{ site.baseurl }}/images/diagrams/docker-django-development.png" title="Deployment Diagram - Docker, Django" name="Deployment Diagram - Docker, Django" />

## Deployment Diagram - Production environment
<img src="{{ site.baseurl }}/images/diagrams/docker-django-wsgi-production.png" title="Deployment Diagram - Docker, Django, WSGI" name="Deployment Diagram - Docker, Django, WSGI" />

### Communication

* These diagrams shows the Docker images needed to set the application service in his first version (**_App:1.0_**). The version number will be increasing during the system evolution.
* The user access from his **_web browser_** to the **_host_** is through the port **_80_**. The communication between the host and the **_Docker container_** can be established using any port.

### Data Volumes
Both diagrams use one data volume, but for different targets.

* **_Development environment:_** uses the **_volume_** to store all the **_source code_**. Doing this, the developers can make the necessary adjustments and see the changes easily.
* **_Production environment:_** uses the **_volume_** only to store configuration files because all the **_source code_** is copied inside the **_Docker_** image during his build. The configuration files are references to the web server **_Nginx_**.

### Database
The database used for this example was **_SQLite_**. This database is configuring automatically during the project creation. The database is part of the **_data volume_** in the **_production environment_** in order to realize backups frequently.

### Source Code
If you want to start a Django project with this architecture, you can download <a target="_blank" href="https://github.com/mmorejon/docker-django">the source code using this link</a>.

### Significant Revisions
* <a target="_blank" href="https://docs.docker.com/compose/django/">Docker Compose with Django projects</a>
* <a target="_blank" href="https://docs.djangoproject.com/es/1.9/intro/tutorial01/">First steps with Django projects</a>