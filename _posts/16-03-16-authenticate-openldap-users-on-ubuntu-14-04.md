---
layout: post_en
title: How to authenticate OpenLdap users on Ubuntu 14.04
permalink: /en/blog/how-to-authenticate-openldap-users-on-ubuntu-14-04/
translate_es: /blog/como-autenticar-usuarios-openldap-en-ubuntu-14-04/
lang: en
sidebar: yes
promo: yes
category: [configuration]
tags: [openldap, ubuntu]
image: /images/banners/ubuntu-og.png
excerpt: Connect you clients on <strong><em>Ubuntu 14.04</em></strong> with you <strong><em>OpenLdap</em></strong> directory in just <strong><em>5 steps!!!</em></strong>
---

<img src="{{ site.baseurl }}/images/banners/openldap-ubuntu.png" title="OpenLdap - Ubuntu Trusty" name="OpenLdap - Ubuntu Trusty" />

## Necessity

Authenticate **_OpenLdap_** directory users into machine clients with operating system **_Ubuntu 14.04_**.

## Environment

**_Clients machines_**

```
  OS: Ubuntu 14.04
```

**_OpenLdap system_**

```
  OpenLdap: 2.4.31
Users type: posixAccount
```

## Solution

Installing the following packages. During the installation you should set the data referent with the OpenLdap system.

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

Reboot the operating system

```
sudo reboot
```

**_Done!!! Ubuntu 14.04 client has been configured!!!_**