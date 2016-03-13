---
layout: post_en
title: How to authenticate OpenLdap users on Ubuntu 14.04
permalink: /en/blog/how-to-authenticate-openldap-users-on-ubuntu-14-04/
translate_es: /blog/como-autenticar-usuarios-openldap-en-ubuntu-14-04/
lang: en
sidebar: yes
category: [configuration]
tags: [openldap, ubuntu]
image: /images/banners/docker-og.png
excerpt: Connect you clients on <strong><em>Ubuntu 14.04</em></strong> with you <strong><em>OpenLdap</em></strong> directory in just <strong><em>5 steps!!!</em></strong>
---

<img src="{{ site.baseurl }}/images/banners/django-docker.png" title="Docker - Django" name="Docker - Django" />

## Necessity

Autenticar los usuarios existentes en el directorio **_OpenLdap_** en las máquinas cno sistema operativo **_Ubuntu 14.04_**.

## Environment

**_Clients maquines_**

```
  OS: Ubuntu 14.04
```

**_OpenLdap system_**

```
 OpenLdap: x
User type: Posxit
```

## Solution

Installing the following packages. During the instalation you should set the data referent with the OpenLdap system.

```
sudo apt-get update
sudo apt-get install libpam-ldap nscd sysv-rc-conf
```

Modifying the file `/etc/nsswitch.conf` in order to search the users into OpenLdap directory. You must **_add_** `ldap` at the end of each line.

```
passwd:        compat  ldap
group:         compat  ldap
shadow:        compat  ldap
```

Removing the text `use_authtok` from the following line located in the file `/etc/pam.d/common-password`. The line should stay like this:

```
password        [success=1 user_unknown=ignore default=die]     pam_ldap.so try_first_pass
```

Adding the following line at the end of the file `/etc/pam.d/common-session`.

```
session required    pam_mkhomedir.so skel=/etc/skel umask=0077
```

Activating the module `libnss-ldap` by typing:

```
sysv-rc-conf libnss-ldap on
```

Reboot the opetaing system

```
sudo reboot
```

**_Done!!! Ubuntu 14.04 client has been configurated!!!_**