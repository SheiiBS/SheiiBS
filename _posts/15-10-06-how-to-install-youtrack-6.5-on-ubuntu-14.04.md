---
layout: post_en
title: How to install YouTrack 6.5 on Ubuntu 14.04?
permalink: /en/blog/how-to-install-youtrack-6.5-on-ubuntu-14.04/
translate_es: /blog/como-instalar-youtrack-6.5-en-ubuntu-14.04/
lang: en
sidebar: yes
category: article
tags: [youtrack, ubuntu]
excerpt: <strong><em>Installing and configuring You Track 6.5</em></strong> will allow you to <strong><em>manage your projects</em></strong> in a simple way. By simply using the web browser you will be able to <strong><em>control your enterprise</em></strong>.
image: /images/banners/youtrack-og.jpeg
---

<img src="{{ site.baseurl }}/images/banners/youtrack.gif" title="YouTrack" name="YouTrack" />

### Introduction

**_YouTrack_** is a project management software and issue tracking system developed by **_JetBrains_**. YouTrack is a proprietary, commercial browser-based bug tracker written in Java. The system allow up to 10 users free of cost.

**_YouTrack_** contains modules that allow its integration with source code management, lightweight directories (**_LDAP_**) and project management systems.

Once you finished reading this article you will have the following benefits:

* **_YouTrack_** 6.5 installed on **_Ubuntu 14.04_**.
* Use YouTrack YouTrack as a service to start, stop and restart the system.
* YouTrack integrated with Gmail.
* YouTrack integrated with **_OpenLDAP_**.

## Prerequisites

For this tutorial you need:

* A server running Ubuntu 14.04.
* A user who is not `root` and that has privileges `sudo`.

## Step 1 – Installing Java Runtime Environment (JRE)

YouTrack has been developed in Java and therefore it is necessary to install Java Runtime Environment.

Begin by updating your local packages cache and then installing `openjdk-7-jre`.

```
sudo apt-get update
sudo apt-get install openjdk-7-jre
```

To confirm the installation was completed successfully using the following command:

```
java -version
```

As a result it should be displayed on the **_Terminal_** Java version installed with other system data. For instance:

```
java version "1.7.0_75"
OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-1~trusty1)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)
```

## Step 2 – Create YouTrack user account

The user `youtrack` is created to manage **_YouTrack_** data and services. The user doesn´t need to have an associated password, so the `--disabled-password` parameter is used during its creation. To do this, type:

```
sudo adduser youtrack --disabled-password
```
During the creation of `youtrack` user you must fill some extra fields.

## Step 3 – Create YouTrack directory

The directory `/usr/local/youtrack` is created to store scripts, trace files and executable **_YouTrack_** (.jar). You can create the directory by typing:

```
sudo mkdir -p /usr/local/youtrack
```

The directory created must be configured to belong to the user and group `youtrack` created in the second step. To accomplish this, type:

```
sudo chown youtrack.youtrack /usr/local/youtrack
```

## Step 4 – Create init.d script

You need to create the script responsible for starting, stopping and restarting **_YouTrack_**. The script must be created inside the folder `/etc/init.d/` to be managed as a service. The script name is `youtrack`.

To better understand the script `youtrack` in the paragraph below explaining their variables.

* `HOME` variable to store the reference where YouTrack's files will be deployed. 
* `NAME` variable used to print messages on the Terminal and to set the path of YouTrack execution script.
* `SCRIPT` variable to the path of the script execution is stored.
* `d_start()` function to start YouTrack service.
* `d_stop()` function to stop YouTrack service.

You can create and open the file typing:

```
sudo nano /etc/init.d/youtrack
```

Once the file was created and opening we must include the following information:

```
#! /bin/sh

export HOME=/home/youtrack

NAME=youtrack
SCRIPT=/usr/local/$NAME/$NAME.sh

d_start() {
	su youtrack -l -c "$SCRIPT start"
}

d_stop() {
	su youtrack -l -c "$SCRIPT stop"
}

case "$1" in
	start)
		echo "Starting $NAME..."
		d_start
		;;
	stop)
		echo "Stopping $NAME..."
		d_stop
		;;
	restart|force-reload)
		echo "Restarting $NAME..."
		d_stop
		d_start
		;;
	*)
		echo "Usage: sudo service youtrack {start|stop|restart}" >&2
		exit 1
		;;
esac

exit 0
```

After the file has been closed you must assigned execute permissions. To do this, the code is as follows:

```
sudo chmod +x /etc/init.d/youtrack
```

`youtrack` script is inserted at the beginning of the process using the operating system defaults. Thus **_YouTrack_** will start automatically. For this, enter the following code:

```
sudo /usr/sbin/update-rc.d youtrack defaults
```

## Step 5 – Create execution script for YouTrack

The script is created with the runtime configuration for **_YouTrack_**. The script created inside the folder `/usr/local/youTrack/`. The file name is `youtrack.sh`. 

To better understand the information in script below explaining their variables.

* `NAME` variable to create trace files.
* `PORT` variable to set the port to be used in the browser. This port can not be the **_80_**.
* `USR` variable to set the position of YouTrack files.
* `JAR` variable to set the route of YouTrack .jar file.
* `LOG` variable to set the file name YouTrack trace.
* `PID` variable to set the file name where the identifier of the process that started YouTrack is stored.
* `d_start` function to start YouTrack service.
* `d_stop` function to stop YouTrack service.

To create it using the following command:

```
sudo nano /usr/local/youtrack/youtrack.sh
```

The file created contain the following information:

```
#! /bin/sh

export HOME=/home/youtrack

NAME=youtrack
PORT=8080
USR=/usr/local/$NAME
JAR=$USR/`ls -Lt $USR/*.jar | grep -o "$NAME-[^/]*.jar" | head -1`
LOG=$USR/$NAME-$PORT.log
PID=$USR/$NAME-$PORT.pid

d_start() {
	if [ -f $PID ]; then
		PID_VALUE=`cat $PID`
		if [ ! -z "$PID_VALUE" ]; then
		PID_VALUE=`ps ax | grep $PID_VALUE | grep -v grep | awk '{print $1}'`
			if [ ! -z "$PID_VALUE" ]; then
				exit 1;
			fi
		fi
	fi

	PREV_DIR=`pwd`
	cd $USR
	exec java -Xmx1g -XX:MaxPermSize=250m -Djava.awt.headless=true -jar $JAR $PORT >> $LOG 2>&1 &
	echo $! > $PID
	cd $PREV_DIR
}

d_stop() {
	if [ -f $PID ]; then
		PID_VALUE=`cat $PID`
		if [ ! -z "$PID_VALUE" ]; then
			PID_VALUE=`ps ax | grep $PID_VALUE | grep -v grep | awk '{print $1}'`
			if [ ! -z "$PID_VALUE" ]; then
				kill $PID_VALUE
				WAIT_TIME=0
				while [ `ps ax | grep $PID_VALUE | grep -v grep | wc -l` -ne 0 -a "$WAIT_TIME" -lt 2 ]
				do
					sleep 1
					WAIT_TIME=$(expr $WAIT_TIME + 1)
				done
				if [ `ps ax | grep $PID_VALUE | grep -v grep | wc -l` -ne 0 ]; then
					WAIT_TIME=0
					while [ `ps ax | grep $PID_VALUE | grep -v grep | wc -l` -ne 0 -a "$WAIT_TIME" -lt 15 ]
					do
						sleep 1
						WAIT_TIME=$(expr $WAIT_TIME + 1)
					done
					echo
				fi
				if [ `ps ax | grep $PID_VALUE | grep -v grep | wc -l` -ne 0 ]; then
					kill -9 $PID_VALUE
				fi
			fi
		fi
		rm -f $PID
	fi
}

case "$1" in
	start)
		d_start
	;;
	stop)
		d_stop
	;;
	*)
		echo "Usage: $0 {start|stop|restart}" >&2
		exit 1
	;;
esac

exit 0
```

After the file has been closed you must assigned execute permissions. To do this, the code is as follows:

```
sudo chmod +x /usr/local/youtrack/youtrack.sh
```

## Step 6 – Download YouTrack

**_YouTrack_** is downloaded from the official site. This guide has been used version **_6.5_**. The downloaded file has extension `.jar`.

```
sudo su youtrack -l -c "cd /usr/local/youtrack && wget http://download-cf.jetbrains.com/charisma/youtrack-6.5.16713.jar"
```

## Step 7 – Start YouTrack

Once you´ve finished the settings, YouTrack system is ready to be started. For this, write:

```
sudo service youtrack start
```

To confirm operation the application will be running for the port you have set. For example: `http://<server>:<port>`.

## (Optional) Step – 8 Configure Gmail

**_YouTrack_** allows mail settings **_SMTP + SSL_** protocol. To configure your email account with **_Gmail_** the fields should be filled  as it follows:

```
          SMTP host: smtp.gmail.com
          SMTP port: 465
      Mail protocol: SMTP+SSL
         SMTP login: your_account@gmail.com
      SMTP password: your_password
     Select SSL key: Please select option
Server 'from' email: your_account@gmail.com
```

## (Optional) Step – 9 Authentication using OpenLDAP

**_YouTrack_** allows user authentication from OpenLDAP directories. For configuration you must add a new authentication module from the panel of administration, type **_LDAP_**.

The data to be configured are:

* **_Name:_** directory name.
* **_Server URL:_** directory url.
* **_DN Transform:_** location where users are located within the directory.
* **_Filter:_** field used as a filter of the user.

OpenLDAP attributes that must be filled are:

* **_Name:_** user name field in the Open LDAP.
* **_Login:_** field used to register the user in YouTrack.
* **_Email:_** OpenLDAP user field in the mail.

A sample configuration is as follows:

```
			Name: Company OpenLDAP
Server URL: ldap://ldap.company.com:389/dc=company,dc=com
    DN Transform: cn=%u,ou=people,dc=company,dc=com
          Filter: cn=%u
  Select SSL key: No Key

Atributos LDAP
			Name: displayName
		   Login: cn
		   Email: mail
```

## (Optional) Step 10 – Update YouTrack

With settings made so far will be very easy to upgrade the system YouTrack. Simply need to download the new version from the official site and restart the service. To achieve this you must enter the following code:

First download `.jar` file.

```
sudo su youtrack -l -c "cd /usr/local/youtrack && wget http://download.jetbrains.com/charisma/youtrack-<version>.jar"
```

Restart system.

```
sudo service youtrack restart
```

## Final Thoughts

**_Installing and configuring You Track 6.5_** will allow you to manage your projects in a simple way. By simply using the web browser you will be able to control your enterprise.

### Significant Revisions

* <a href="https://www.jetbrains.com/youtrack/" target="_blank">Official Site YouTrack</a>