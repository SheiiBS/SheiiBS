---
layout: post_en
title: Start Django project with Docker
permalink: /en/blog/start-django-project-with-docker/
translate_es: /blog/iniciar-proyecto-en-django-con-docker/
lang: en
sidebar: yes
promo: yes
category: [article]
tags: [django, docker]
image: /images/banners/docker-og.png
excerpt: Steps to start <strong><em>Django</em></strong> projects using <strong><em>Docker</em></strong> as a part of the <strong><em>workflow</em></strong>.
---
<img src="{{ site.baseurl }}/images/banners/django-docker.png" title="Docker - Django" name="Docker - Django" />

### Introduction

Steps to start Django projects using Docker as a part of the workflow. We use `projectname` to refer the project name.

### Prerequisite

* Having **Docker** installed. If you are using on Mac OS X or Windows you should install **Docker Toolbox**.

### Source Code

You can download <a href="https://github.com/mmorejon/docker-django" target="_blank">the source code using this link</a>.

### UML Diagrams

You can consult deployment diagrams related to this post in this <a href="{{ site.baseurl }}/en/blog/deployment-diagram-docker-django/">link</a>.

## Step One - Setup the Structure.

**_Download the project structure_**

The project containing the general structure is discharged.

```
git clone https://github.com/mmorejon/docker-django.git projectname
```

**_Remove Git folder_**

The folder `.git` is removed in order to create a new repository.

```
cd projectname
rm -rf .git/
```

**_Initialize new repository inside the project_**

Initialize a new version control over the folder to track all changes.

```
git init
```

## Step Two - Create Docker Image

**_Create Docker image_**

You must create the Docker image for the project. The image contains the installation requirements established into `requirements.txt` file.

The file `requirements.txt` include basic requirements to start and deploy our Django application. If you need to add new elements now is a good moment. Use the following command to create the image:

```
docker build -t projectname:1.0 .
```

Every time that you modify the elements inside `requirements.txt` file you should repeat this step.

**_Docker Compose configuration_**

In the file `docker-compose.yml` is set the image name that will be used. The image name was setting in the previous step. In the file change the following line:

```
image: projectname:1.0
```

## Step Three - Create Django Project

**_Create project_**

The project is created using the same commands described into Django website.

```
docker-compose run web django-admin startproject projectname .
```

**_Test the system_**

Start the system typing this:

```
docker-compose up
```

The system can be tested writing in the browser the following: `http://<machine-ip:8000>`. The port can be changed in the `docker-composer.yml` file.

**_Stop the system_**

The system is stopped to keep on the configurations typing this:

```
Ctrl-C
```

## Step Four - Create Application

Use the following command to create an application inside Django project.

```
docker-compose run web python manage.py startapp app
```

## Step Five - Create User

The users can be created using the same command line explained in Django documentation.

```
docker-compose run web python manage.py createsuperuser
```

## Step Six - Production Environment

If you desire to take advantage of the application like production environment, you just need to configure the following files:

Add the this line at the end of the `projectname/settings.py` file:

```
STATIC_ROOT = './static/'
```

Add this line `command: ./run-production.sh` in `docker-compose.yml` file. The file should look like this:

```
web:
  image: projectname:1.0
  command: ./run-production.sh
  volumes:
    - .:/code
  ports:
    - "8000:80"
```

The file `run-production.sh` include the necessaries configurations to run your Django application like a production environment.

Finally, change `projectname` by `your-app-name` in `conf/app.ini` file.

### Significant Revisions

* <a target="_blank" href="https://docs.docker.com/compose/django/">Docker Compose with Django projects</a>
* <a target="_blank" href="https://docs.djangoproject.com/es/1.9/intro/tutorial01/">First steps in Django projects</a>
