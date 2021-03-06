---
layout: post_en
title: Upload max file size to import on phpMyAdmin
permalink: /en/blog/upload-max-file-size-import-phpmyadmin/
translate_es: /blog/aumentar-tamano-ficheros-importar-phpmyadmin/
lang: en
sidebar: yes
category: [configuration]
tags: [phpmyadmin]
---

The **_max file size_** that will be imported on **_phpMyAdmin_** can be changed easily.

### Necessity

Import files on phpMyAdmin with largest size.

### Environment

```
 OS: Ubuntu 14.04
PHP: 5.5.9
```

### Solution

Open for edit `/etc/php5/apache2/php.ini` file.

```
sudo nano /etc/php5/apache2/php.ini
```

Change values depending of the size you need.

```
upload_max_filesize = 128M
post_max_size = 128M
```

Restart Apache service.

```
sudo service apache2 restart
```
