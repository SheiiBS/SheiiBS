---
layout: post_en
title: Migrate Jenkins to a Docker Service
permalink: /en/blog/migrate-jenkins-to-a-docker-container/
translate_es: /blog/migrar-jenkins-hacia-un-contenedor-en-docker/
lang: en
sidebar: yes
promo: yes
category: [article]
tags: [docker, jenkins, ubuntu]
image: /images/banners/docker-og.png
excerpt: Migrate your current <strong><em>Jenkins Service</em></strong> to a <strong><em>Docker Container</em></strong> is very easy. The Jenkins system <strong><em>just will be out of service during 5 seconds and without be lost information</em></strong>.
---

<img src="{{ site.baseurl }}/images/banners/jenkins-docker.png" title="Migrar Jenkins hacia Docker" name="Migrar Jenkins hacia Docker" />

### Introduction

**_Jenkins_** is a continuous integration system used widely by **_software development groups_**. By the other hand, **_Docker_** show us a new way to structure ours services.

Migrate our **_Jenkins_** system to the philosophy provided by **_Docker_** can be a desire or a goal in our **_development group_**. However, you may have some questions or doubts:

- _Will be lost the configurations made so far ?_
- _Will be lost the statistics of jobs?_
- _¿How many time the Jenkins service will be down?_

The answer in both cases is **_no_**. The objective of this article is show how modify our structure of service for **_Jenkins_** without start from the beginning.

All modifications showed in the article where realized in the same server although you can use a second one.

**_Jenkins just will be out of service during 5 seconds._**

### Prerequisite

To take the steps mentioned in this article you must complete the following requirement:

- Having **_Jenkins_** installed.

### Environment

The environment configuration used for this article is the following:

```
### Continuous Integration System ###
         OS: Ubuntu 14.04
    Jenkins: 1.645
     Domain: jenkins.example.com
```
The `jenkins.example.com` domain is used to access to the Jenkins system.

## Step One – Install Docker.

The best information related with **_Docker_** installation can be found on his <a target="_blank" href="https://docs.docker.com/engine/installation/ubuntulinux/">Official Website</a>. During the **_Docker_** installation you must take into account the operating system installed in your server.

## Step Two – Start the Jenkins service.

**_Start Jenkins_**

To start **_Jenkins_** using **_Docker_** you must write the following command:

```
docker run -p 8085:8080 --name jenkins jenkins
```

- `-p 8085:8080` link the port `8080` of jenkins container with the host port `8085`.
- `--name jenkins` establish the name `jenkins` as the container name created.

**_Check if Jenkins works_**

Access to the following web link to check if Jenkins works correctly.

```
http://jenkins.example.com:8085
```

<img src="{{ site.baseurl }}/images/migrate-docker-jenkins/welcome-jenkins.png" title="Welcome Jenkins" name="Welcome Jenkins" />

**_Stop the service_**

It is necessary stop the service in order to move on with the configurations.

```
Ctrl-C
```

## Step Three – Set a data volume for Jenkins.

**_Create structure_**

Create folders to store both, data and configurations for the service. 

```
sudo mkdir -p ~/jenkins/data
```

Establish the permission `777` to the created folders so that the `jenkins` user can write in them.

```
sudo chmod -R 777 ~/jenkins/
```

- _jenkins folder_: store **_Docker_** configurations.
- _data folder_: store **_Jenkins_** data. 

**_Start the service using the data volume_**

Write the following command to start the service using the data volume:

```
docker run -d -p 8085:8080 --name jenkins-with-volume -v ~/jenkins/data:/var/jenkins_home jenkins
```

- We use `-d` to start **_Jenkins_** as a service.

You can see the files created by **_Jenkins_** inside the folder `~/jenkins/data`. These files are **_Jenkins_** configurations.

**_Stop the service_**

It is necessary stop the service so that to move on with the configurations.

```
docker stop jenkins-with-volume
```


## Step Four – Migrate the information to the container.

Once the folders structure has been created, the migration of the **_Jenkins_** information is very simple.

First, you must delete all the information located inside the folder `~/jenkins/data`.

```
sudo rm -r ~/jenkins/data/*
sudo rm -r ~/jenkins/data/.*
```


After, search the value of `JENKINS_HOME` inside the **_Jenkins_** website on: **_Jenkins > Manage Jenkins > System Information_**.

`JENKINS_HOME` is the physical path where Jenkins has his information.

<img src="{{ site.baseurl }}/images/migrate-docker-jenkins/jenkins-home.png" title="Jenkins Home" name="Jenkins Home" />

Once identified the path of **_Jenkins_** information, you can copy those files to the folder `~/jenkins/data`.

```
sudo cp -r /var/lib/jenkins/* ~/jenkins/data/
```

**_Start the service_**

It is necessary restart the service and checking that **_Jenkins_** have all the information in the link `http://jenkins.example.com:8085`.

```
docker start jenkins-with-volume
```

## Step Five – Install Docker Compose.

The best information related with **_Docker Compose_** installation can be found on his <a target="_blank" href="https://docs.docker.com/compose/install/">Official Website</a>.

## Step Six – Use Docker Compose in the process.

**_Create YAML file_**

Create the file `docker-compose.yml` in the folder `~/jenkins/`.

```
touch ~/jenkins/docker-compose.yml
```

Copy the following information inside the file:

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

- `app` is the **_Jenkins_** service.
- `data` is the container for **_Jenkins_** data.
- `restart:always` assure to start the service when the host restart.
- We use `user` inside the string `/home/user/jenkins/data:/var/jenkins_home:rw por` because must be a physical path.

**_Stop the service_**

The following command stop the **_Docker_** service:

```
docker stop jenkins-with-volume
```

**_Start the service using Docker Compose_**

This time the Jenkins service starts using **_Docker Compose_**. To do this, type:

```
cd ~/jenkins/
docker-compose up -d
```

Use this link: `http://jenkins.example.com:8085` to check the right behaviors.

## Step Seven – Establish Jenkins as a Docker Service.

**_Modify the port inside the file yml_**

Modify the host port in the file `docker-compose.yml`. The file look like this now:

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

**_Stop the Jenkins service_**

The **_Jenkins_** service is stopped using the following command:

```
sudo service jenkins stop
```

**_Start Jenkins as a Docker service_**

Restart the **_Jenkins_** service with **_Docker_**.

```
docker-compose up -d
```

 Use this link `http://jenkins.example.com` to check the right behaviours.

**Ready!!! Migration finished.**

## Final Thoughts

The philosophy of **_Docker_** to establish **_Containers as a Services (CaaS)_** is widely accepted by software community. By the other hand, **_Jenkins_** constitute a powerful tool for organize both process, **_Continuous Integration and Continuous Delivery_**. This article establish the following:

- Establish **_Jenkins_** as a service inside a container.
- The migration of **_Jenkins_** service allow keep the data and configurations.
- The **_Jenkins_** service just will be out of service 5 seconds during the migration.

### Significant Revisions
- <a target="_blank" href="http://docker.com/">Docker - Official Website</a>.
- <a target="_blank" href="https://jenkins-ci.org/">Jenkins - Official Website</a>.