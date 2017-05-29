---
layout: post_en
title: Microservices - Example with Docker, Go and MongoDB
permalink: /en/blog/microservices-example-with-docker-go-and-mongodb/
translate_es: /blog/microservicios-ejemplo-con-docker-go-y-mongodb/
lang: en
sidebar: yes
promo: yes
category: [article]
tags: [docker, go, mongodb]
image: /images/banners/docker-og.png
excerpt: <strong><em>Microservices, from theory to practice.</em></strong> Cinema is an <strong><em>example project</em></strong> written in <strong><em>Go</em></strong>, deployed with <strong><em>Docker</em></strong> and stored in <strong><em>MongoDB</em></strong> databases.
---

<img src="{{ site.baseurl }}/images/banners/microservices-docker.png" title="Microservices - Cinema" name="Microservices - Cinema" />

### Introduction

**_Cinema is an example project_** which demonstrates the use of **_microservices_** for a fictional movie theater.
The Cinema backend is powered by 4 microservices, all of witch happen to be written in **_Go_**, using **_MongoDB_** for manage the database and **_Docker_** to isolate and deploy the ecosystem.

 * **_Users Service_**: Provides users information.
 * **_Movie Service_**: Provides information like movie ratings, title, etc.
 * **_Show Times Service_**: Provides show times information.
 * **_Booking Service_**: Provides booking information.

**_The Cinema use case_** is based on the project written in Python by [Umer Mansoor](https://github.com/umermansoor/microservices).

The project structure is based in the knowledge learned in the book: **_Web
Development with Go_** by **_Shiju Varghese_**, ISBN 978-1-4842-1053-6

### Prerequisite

* Docker 1.12
* Docker Compose 1.8

We must **add virtual domains** in order to use each **api entry point**. By default we are using: **movies.local**, **bookings.local**, **users.local** and **showtimes.local**

**Virtual domains** has been defined in `docker-compose.yml` file and configured in `/etc/hosts` file. Add the following line in your `/etc/hosts` file:

```
127.0.0.1   movies.local bookings.local users.local showtimes.local
```


### Source Code

You can download <a href="https://github.com/mmorejon/microservices-docker-go-mongodb" target="_blank">the source code using this link</a>.

### Services diagram

<img src="{{ site.baseurl }}/images/microservices-cinema/deployment-diagram.png" title="Microservices - Cinema" name="Microservices - Cinema" />

## Starting services

```
docker-compose up -d
```

## Stopping services

```
docker-compose stop
```

## Including new changes

If you need change some source code you can deploy it typing:

```
docker-compose build
```

## Restore database information

You can start using an empty database for all microservices, but if you want you can restore a configured data following this steps:

**_Access to mongodb container typing:_**

```
docker exec -it cinema-db /bin/bash
```

**_Restore data typing:_**

```
/backup/restore.sh
```

**_Leave the container:_**

```
exit
```


## Documentation

### User Service

This service returns information about the users of Cinema.

**_Routes:_**

* GET - _http://users.local/users_ : Get all users
* POST - _http://users.local/users_ : Create user
* DELETE - _http://users.local/users/{id}_ : Remove user by id

### Movie Service

This service is used to get information about a movie. It provides the movie title, rating on a 1-10 scale, director and other information.

**_Routes:_**

* GET - _http://movies.local/movies_ : Get all movies
* POST - _http://movies.local/movies_ : Create movie
* GET - _http://movies.local/movies/{id}_ : Get movie by id
* DELETE - _http://movies.local/movies/{id}_ : Remove movie by id

### Showtimes Service

This service is used get a list of movies playing on a certain date.

**_Routes:_**

* GET - _http://showtimes.local/showtimes_ : Get all showtimes
* POST - _http://showtimes.local/showtimes_ : Create showtime
* GET - _http://showtimes.local/showtimes/{id}_ : Get showtime by id
* DELETE - _http://showtimes.local/showtimes/{id}_ : Remove showtime by id

### Booking Service

Used to lookup booking information for users.

**_Routes:_**

* GET - _http://bookings.local/bookings_ : Get all bookings
* POST - _http://bookings.local/bookings_ : Create booking

### Exposed ports

The port `27017` is exposed to be consulted by **_Robomongo_** system.
The port `80` is exposed to be consulted by **_devices_**, **_web browsers_** or others **_microservices_**.

<img src="{{ site.baseurl }}/images/microservices-cinema/exposed-ports.png" title="Microservices - Exposed ports" name="Microservices - Exposed ports" />


## Screenshots

**_Starting services_**

<img src="{{ site.baseurl }}/images/microservices-cinema/starting-services.png" title="Microservices - Starting services" name="Microservices - Starting services" />

**_Restoring database information_**

<img src="{{ site.baseurl }}/images/microservices-cinema/restoring-database-information.png" title="Microservices - Restoring database information" name="Microservices - Restoring database information" />

**_Service - Get all users (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-users.png" title="Microservices - Get all users" name="Microservices - Get all users" />

**_Service - Get all movies (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-movies.png" title="Microservices - Get all movies" name="Microservices - Get all movies" />

**_Service - Get all showtimes (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-showtimes.png" title="Microservices - Get all showtimes" name="Microservices - Get all showtimes" />

**_Service - Get all bookings (Postman)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/get-all-bookings.png" title="Microservices - Get all bookings" name="Microservices - Get all bookings" />

**_Database big picture (Robomongo)_**

<img src="{{ site.baseurl }}/images/microservices-cinema/database-big-picture.png" title="Microservices - Database big picture" name="Microservices - Database big picture" />

### Significant Revisions

* [Microservices - Martin Fowler](http://martinfowler.com/articles/microservices.html)
* [Web Development with Go](http://www.apress.com/9781484210536)
* [Umer Mansoor - Cinema](https://github.com/umermansoor/microservices)
