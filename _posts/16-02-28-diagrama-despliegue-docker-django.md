---
layout: post
title: Diagrama de Despliegue - Django con Docker
permalink: /blog/diagrama-despliegue-docker-django/
translate_en: /en/blog/deployment-diagram-docker-django/
lang: es
sidebar: yes
promo: yes
category: [diagrama]
tags: [django, uml, docker]
image: /images/banners/docker-og.png
excerpt: <strong><em>Diagrama de Despliegue</em></strong> mostrando similitudes y diferencias entre los <strong><em>entornos de desarrollo y producción</em></strong> para <strong><em>projectos con Django</em></strong> utilizando <strong><em>Docker</em></strong>.
---

<img src="{{ site.baseurl }}/images/banners/django-docker.png" title="Docker - Django" name="Docker - Django" />

## Diagrama de Despliegue - Entorno de desarrollo
<img src="{{ site.baseurl }}/images/diagrams/docker-django-development.png" title="Diagrama de Despliegue - Docker, Django" name="Diagrama de Despliegue - Docker, Django" />

## Diagrama de Despliegue - Entorno de producción
<img src="{{ site.baseurl }}/images/diagrams/docker-django-wsgi-production.png" title="Diagrama de Despliegue - Docker, Django, WSGI" name="Diagrama de Despliegue - Docker, Django, WSGI" />

### Comunicación

* Se muestra las imágenes de **_Docker_** necesarias para establecer el servicio de la aplicación en su primera versión (**_App:1.0_**). Durante la evolución del sistema la versión debe ir incrementando.
* El usuario accede desde su **_navegador web_** al alojamiento **_(host)_** a través del puerto **_80_**. La comunicación entre el alojamiento y el contenedor en **_Docker_** se puede establecer utilizando cualquier puerto.

### Volumen de datos
Ambos diagramas utilizan un volumen de datos, pero con objetivos distintos.

* **_Entorno de desarrollo:_** utiliza el **_volumen_** para almacenar la totalidad del **_código fuente_**. De esta forma los programadores pueden hacer los ajustes necesarios y ver reflejados los cambios fácilmente.
* **_Entorno de producción:_** utiliza el **_volumen_** solo para el almacenamiento de los ficheros de configuración porque el **_código fuente_** se copia hacia la imagen de **_Docker_** durante su construcción. Los ficheros de configuración son los referentes al servidor web **_Nginx_**.

### Base de datos
La base de datos utilizada en el ejemplo fue **_SQLite_**. Esta base de datos se configura automáticamente durante la confección del proyecto. En el **_entorno de producción_** la base de datos forma parte del **_volumen de datos_** para la realización de salvas.

### Código fuente

Si desea iniciar un **_proyecto con Django_** y utilizar esta arquitectura puede descargar el <a target="_blank" href="https://github.com/mmorejon/docker-django">código fuente desde aquí</a>.

### Revisiones significativas
* <a target="_blank" href="https://docs.docker.com/compose/django/">Docker Compose con proyectos Django</a>
* <a target="_blank" href="https://docs.djangoproject.com/es/1.9/intro/tutorial01/">Primeros pasos en projectos con Django</a>