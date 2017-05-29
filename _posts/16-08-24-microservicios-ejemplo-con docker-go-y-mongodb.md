---
layout: post
title: Microservicios - Ejemplo con Docker, Go y MongoDB
permalink: /blog/microservicios-ejemplo-con-docker-go-y-mongodb/
translate_en: /en/blog/microservices-example-with-docker-go-and-mongodb/
lang: es
sidebar: yes
promo: yes
category: [articulo]
tags: [docker, go, mongodb]
image: /images/banners/docker-og.png
excerpt: <strong><em>Microservicios, de la teoría a la práctica.</em></strong> Cinema es un <strong><em>proyecto ficticio</em></strong> escrito en <strong><em>Go</em></strong>, desplegado con <strong><em>Docker</em></strong> y utilizando <strong><em>MongoDB</em></strong> para almacenar la información.
---
<img src="{{ site.baseurl }}/images/banners/microservices-docker.png" title="Microservicios - Cinema" name="Microservicios - Cinema" />

### Introducción

**_Cinema es un proyecto ficticio_** que utiliza **_microservicios_** para el sistema de un cine. La lógica de Cinema está compuesta por 4 microservicios, todos escritos en **_Go_**, utilizando **_MongoDB_** para la gestión de datos y **_Docker_** para aislar y desplegar el ecosistema.

 * Servicio Users: Brinda información de los usuarios.
 * Servicio Movie: Brinda información de las películas como valoración, título, etc.
 * Servicio Show Times: Brinda información sobre la programación de las películas.
 * Servicio Booking: Brinda información sobre las reservaciones de los usuarios referente a la programación de las películas.

El caso de uso Cinema está basado en el proyecto escrito en Python por [Umer Mansoor](https://github.com/umermansoor/microservices).

La estructura del proyecto está basada en el conocimiento obtenido del libro **_Web Development with Go_** escrito por **_Shiju Varghese_**, ISBN 978-1-4842-1053-6

### Requerimientos

* Docker 1.12
* Docker Compose 1.8

Se tiene que agregar **dominios virtuales** en la máquina local para poder utilizar los puntos de entrada a las funcionalidades. Para el ejemplo se utiliza: **movies.local**, **bookings.local**, **users.local** y **showtimes.local**.

Los **_dominios virtuales_** han sido definidos en el fichero `docker-compose.yml` y configurados en el fichero `/etc/hosts`. Adicione la siguiente línea en el fichero `/etc/hosts`:

```
127.0.0.1   movies.local bookings.local users.local showtimes.local
```


### Código fuente

Usted puede descargar <a href="https://github.com/mmorejon/microservices-docker-go-mongodb" target="_blank"> el código fuente utilizando este enlace</a>.

### Diagrama de servicios

<img src="{{ site.baseurl }}/images/microservices-cinema/diagrama-despliegue.png" title="Microservicios - Cinema" name="Microservicios - Cinema" />

## Iniciar servicios

```
docker-compose up -d
```

## Detener servicios

```
docker-compose stop
```

## Incluir nuevos cambios

Si necesita modificar el código fuente usted puede desplegar los nuevos cambios escribiendo:

```
docker-compose build
```

## Restaurar información en la base de datos.

Puede iniciar los microservicios utilizando la base de datos vacía, pero si prefiere, puede restaurar un juego de datos siguiendo los siguientes pasos:

**_Acceda al contenedor de MongoDB escribiendo:_**

```
docker exec -it cinema-db /bin/bash
```

**_Restaure la información escribiendo:_**

```
/backup/restore.sh
```

**_Salga del contenedor:_**

```
exit
```


## Documentación

### Servicio User

Este servicio brinda información sobre los usuarios de Cinema.

**_Rutas:_**

* GET - _http://users.local/users_ : Obtiene todos los usuarios.
* POST - _http://users.local/users_ : Crea un usuario
* DELETE - _http://users.local/users/{id}_ : Elimina a un usuario por el id

### Servicio Movie

Este es utilizado para gestionar la información de las películas. Cada película tiene el título, valoración en escala del 1-10, director, entre otros.

**_Rutas:_**

* GET - _http://movies.local/movies_ : Obtiene todas las películas
* POST - _http://movies.local/movies_ : Crea una película
* GET - _http://movies.local/movies/{id}_ : Obtiene una película por el id
* DELETE - _http://movies.local/movies/{id}_ : Elimina una película por el id

### Servicio Showtimes

Este servicio brinda la programación de las películas en ciertas fechas.

**_Rutas:_**

* GET - _http://showtimes.local/showtimes_ : Obtiene todas las programaciones
* POST - _http://showtimes.local/showtimes_ : Crea una programación
* GET - _http://showtimes.local/showtimes/{id}_ : Obtiene una programación por el id
* DELETE - _http://showtimes.local/showtimes/{id}_ : Elimina una programación por el id

### Servicio Booking

Este servicio brinda información sobre las reservaciones realizadas por los usuarios referente a las programaciones.

**_Rutas:_**

* GET - _http://bookings.local/bookings_ : Obtiene todas las reservaciones
* POST - _http://bookings.local/bookings_ : Crear una reservación.

### Puertos expuestos

El puerto `27017` está expuesto para que sea consultado de manera local por el sistema **_Robomongo_**. El puerto `80` está expuesto para que sea consultado por **_dispositivos_**, **_navegadores webs_** u otros **_microservicios_**.

<img src="{{ site.baseurl }}/images/microservices-cinema/exposed-ports.png" title="Microservicios - Puertos expuestos" name="Microservicios - Puertos expuestos" />


## Pantallas

**_Iniciar servicios_**

<img src="{{ site.baseurl }}/images/microservices-cinema/starting-services.png" title="Microservicios - Iniciar servicios" name="Microservicios - Iniciar servicios" />

**_Restaurar base de datos_**

<img src="{{ site.baseurl }}/images/microservices-cinema/restoring-database-information.png" title="Microservicios - Restaurar base de datos" name="Microservicios - Restaurar base de datos" />

**_Servicio - Obtener todos los usuarios (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-users.png" title="Microservicios - Obtener todos los usuarios (Postman)" name="Microservicios - Obtener todos los usuarios (Postman)" />

**_Servicio - Obtener toas las películas (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-movies.png" title="Microservicios - Obtener toas las películas (Postman)" name="Microservicios - Obtener toas las películas (Postman)" />

**_Servicio - Obtener todos las programaciones (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-showtimes.png" title="Microservicios - Obtener todos las programaciones" name="Microservicios - Obtener todos las programaciones" />

**_Servicio - Obtener todos las reservaciones (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-bookings.png" title="Microservicios - Obtener todos las programaciones (Postman)" name="Microservicios - Obtener todos las programaciones (Postman)" />

**_Base de datos, imagen completa (Robomongo)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/database-big-picture.png" title="Microservicios - Base de datos, imagen completa (Robomongo)" name="Microservicios - Base de datos, imagen completa (Robomongo)" />

### Revisiones significativas

* [Microservices - Martin Fowler](http://martinfowler.com/articles/microservices.html)
* [Web Development with Go](http://www.apress.com/9781484210536)
* [Umer Mansoor - Cinema](https://github.com/umermansoor/microservices)
