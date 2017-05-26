---
layout: post
title: Migrar Jenkins hacia un Contenedor en Docker
permalink: /blog/migrar-jenkins-hacia-un-contenedor-en-docker/
translate_en: /en/blog/migrate-jenkins-to-a-docker-container/
lang: es
sidebar: yes
promo: yes
category: [articulo]
tags: [docker, jenkins, ubuntu]
image: /images/banners/docker-og.png
excerpt: Migrar tu actual <strong><em>Servicio de Jenkins</em></strong> hacia un <strong><em>contenedor en Docker</em></strong> es <strong><em>bien sencillo</em></strong>. El sistema Jenkins <strong><em>solo dejará de brindar servicios durante 5 segundos y sin perder información</em></strong>.
---

<img src="{{ site.baseurl }}/images/banners/jenkins-docker.png" title="Migrar Jenkins hacia Docker" name="Migrar Jenkins hacia Docker" />

### Introducción

**_Jenkins_** es un sistema de integración continua utilizado ampliamente por los **_grupos de desarrollo de software_**. Por otra parte, **_Docker_** nos brinda una nueva forma de estructurar nuestros servicios.

Migrar nuestro sistema **_Jenkins_** hacia la filosofía brindada por **_Docker_** puede ser un deseo o una meta de nuestro **_grupo de desarrollo_**. Sin embargo, existen preguntas a responder: 

- _¿Se perderán las configuraciones establecidas hasta el momento?_
- _¿Se perderán las estadísticas de las rutinas establecidas?_
- _¿Que tiempo estará Jenkins sin brindar servicios?_

La respuesta a ambas preguntas es **_no_**. El objetivo del presente artículo es mostrar cómo modificar la estructura del sistema **_Jenkins_** sin tener que empezar desde cero. 

Las modificaciones mostradas en el artículo fueron realizadas todas en el mismo servidor aunque se puede utilizar un segundo alojamiento para el cambio.

**_El sistema Jenkins solo dejará de brindar servicios 5 segundos._**

### Pre requisito

Para realizar los pasos del artículo deberá cumplir con el siguiente requerimiento:

- Tener instalado **_Jenkins_**.

### Entorno

La configuración del entorno donde fue desarrollado el artículo es la siguiente:

```
### Sistema de integración continua ###
         SO: Ubuntu 14.04
    Jenkins: 1.645
    Dominio: jenkins.example.com
```

El dominio `jenkins.example.com` se utiliza para acceder al sistema **_Jenkins_**.

## Paso Uno – Instalar Docker.

La información referente a la instalación de **_Docker_** se encuentra claramente descrita en su <a target="_blank" href="https://docs.docker.com/engine/installation/ubuntulinux/">Sitio Web Oficial</a>. Durante la instalación de **_Docker_** hay que tener en cuenta el sistema operativo instalado en nuestro servidor.

## Paso Dos – Iniciar el servicio de Jenkins.

**_Iniciar Jenkins_**

Para iniciar **_Jenkins_** utilizando **_Docker_** se escribe lo siguiente:

```
docker run -p 8085:8080 --name jenkins jenkins
```

- `-p 8085:8080` enlaza el puerto `8080` del contenedor jenkins al puerto `8085` del host.
- `--name jenkins` establece el nombre `jenkins` al contenedor creado.

**_Comprobar que funciona Jenkins_**

Para comprobar que Jenkins funciona correctamente acceda al siguiente enlace web:

```
http://jenkins.example.com:8085
```

<img src="{{ site.baseurl }}/images/migrate-docker-jenkins/welcome-jenkins.png" title="Bienvenido Jenkins" name="Bienvenido Jenkins" />

**_Detener el servicio_**

Es necesario detener el servicio para continuar con las configuraciones.

```
Ctrl-C
```

## Paso Tres – Establecer un volumen para los datos.

**_Crear estructura_**

Crear las carpetas donde serán almacenados los datos y las configuraciones del servicio. 

```
sudo mkdir -p ~/jenkins/data
```

Dar los permisos `777` a las carpetas creadas para que el usuario `jenkins` escriba dentro de ellas.

```
sudo chmod -R 777 ~/jenkins/
```

- _carpeta jenkins_: almacena las configuraciones para **_Docker_**.
- _carpeta data_: almacena los datos de **_Jenkins_**.

**_Iniciar el servicio utilizando el volumen de datos_**

Para iniciar el servicio utilizando el volumen de datos se escribe lo siguiente:

```
docker run -d -p 8085:8080 --name jenkins-with-volume -v ~/jenkins/data:/var/jenkins_home jenkins
```

- `-d` se utiliza para iniciar **_Jenkins_** como servicio.

Se puede observar que el contenido creado dentro de la carpeta `~/jenkins/data` son los ficheros de configuración de **_Jenkins_**.

**_Detener el servicio_**

Es necesario detener el servicio para continuar con las configuraciones.

```
docker stop jenkins-with-volume
```


## Paso Cuatro – Migrar la información hacia el contendor.

Después de creada la estructura de carpetas es muy sencillo migrar la infomación del sistema **_Jenkins_** que se encuentra actualmente en uso.

Primero se borra toda la información creada por **_Jenkins_** en `~/jenkins/data`.

```
sudo rm -r ~/jenkins/data/*
sudo rm -r ~/jenkins/data/.*
```

Después, se busca el valor de la variable de entorno `JENKINS_HOME` dentro del sitio de **_Jenkins_** en **_Jenkins > Manage Jenkins > System Information_**.

`JENKINS_HOME` es el camino físico donde Jenkins tiene su información.

<img src="{{ site.baseurl }}/images/migrate-docker-jenkins/jenkins-home.png" title="Jenkins Home" name="Jenkins Home" />

Una vez identificada la ubicación de los ficheros de **_Jenkins_** se realiza la copia desde la ruta `JENKINS_HOME` hacia la carpeta `~/jenkins/data`.

```
sudo cp -r /var/lib/jenkins/* ~/jenkins/data/
```

**_Iniciar el servicio_**

Es necesario reiniciar el serivio y revisar que **_Jenkins_** tenga los mismos datos en la dirección web `http://jenkins.example.com:8085`.

```
docker start jenkins-with-volume
```

## Paso Cinco – Instalar Docker Compose.

La información referente a la instalación de **_Docker Compose_** se encuentra claramente descrita en su <a target="_blank" href="https://docs.docker.com/compose/install/">Sitio Web Oficial</a>.

## Paso Seis – Utilizar Docker Compose en el proceso.

**_Crear fichero YAML_**

Se crea el fichero `docker-compose.yml` en la carpeta `~/jenkins/`.

```
touch ~/jenkins/docker-compose.yml
```

Dentro del fichero se agrega la siguiente información:

```
app:
  image: jenkins
  ports:
    - 8085:8080
  volumes_from:
    - data
  restart: always

data:
  image: busybox
  volumes:
    - /home/user/jenkins/data:/var/jenkins_home:rw
```

- `app` es el servicio de **_Jenkins_**.
- `data` es el contenedor para los datos de **_Jenkins_**.
- `restart:always` garantiza iniciar el servicio al reiniciar el host. 
- Se utiliza `user` dentro de `/home/user/jenkins/data:/var/jenkins_home:rw por` y no `~` porque tiene que ser un camino físico.

**_Detener el servicio_**

El servicio de **_Docker_** se detiene utilizando el siguiente comando:

```
docker stop jenkins-with-volume
```

**_Iniciar el servicio con Docker Compose_**

El servicio de Jenkins se inicia utilizando **_Docker Compose_**. Para lograrlo se escribe lo siguiente:

```
cd ~/jenkins/
docker-compose up -d
```

Revisar el correcto funcionamiento del sitio en `http://jenkins.example.com:8085`.

## Paso Siete – Establecer solamente Jenkins como Servicio en Docker.

**_Modificar el puerto en el fichero yml_**

Se modifica el puerto del host en el fichero `docker-compose.yml` quedando de la siguiente forma:

```
app:
  image: jenkins
  ports:
    - 80:8080
  volumes_from:
    - data
  restart: always

data:
  image: busybox
  volumes:
    - /home/user/jenkins/data:/var/jenkins_home:rw
```

**_Detener el servicio de Jenkins_**

Se detiene el servicio **_Jenkins_** utilizando el siguiente comando:

```
sudo service jenkins stop
```

**_Iniciar Jenkins como servicio de Docker_**

Reiniciar el servicio **_Jenkins_** en **_Docker_**.

```
docker-compose up -d
```

Revisar el correcto funcionamiento del sitio en `http://jenkins.example.com`.

**Listo!!! Migración terminada.**

## Reflexiones finales
La filosofía brindada por **_Docker_** para establecer **_Contenedores como Servicios (CaaS)_** es ampliamente aceptada por la comunidad del software. Por otra parte, **_Jenkins_** constituye una poderosa herramienta para garantizar los procesos de **_Integración Continua y Entrega Continua_**. En el presente artículo queda establecido lo siguiente:

- Se establece **_Jenkins_** como servicio dentro de un Contenedor.
- La migración del servicio de **_Jenkins_** permite conservar los datos y las configuraciones.
- El servicio de **_Jenkins_** solo deja de funcionar 5 segundos durante la migración.

### Revisiones significativas
- <a target="_blank" href="http://docker.com/">Docker - Sitio Web Oficial</a>.
- <a target="_blank" href="https://jenkins-ci.org/">Jenkins - Sitio Web Oficial</a>.

