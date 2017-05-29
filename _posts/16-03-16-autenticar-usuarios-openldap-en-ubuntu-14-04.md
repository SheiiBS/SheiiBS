---
layout: post
title: Cómo autenticar usuarios de OpenLdap en Ubuntu 14.04
permalink: /blog/como-autenticar-usuarios-openldap-en-ubuntu-14-04/
translate_en: /en/blog/how-to-authenticate-openldap-users-on-ubuntu-14-04/
lang: es
sidebar: yes
promo: yes
category: [configuracion]
tags: [openldap, ubuntu]
image: /images/banners/ubuntu-og.png
excerpt: Conecte sus clientes en <strong><em>Ubuntu 14.04</em></strong> a su directorio <strong><em>OpenLdap</em></strong> con solo <strong><em>5 pasos!!!</em></strong>
---

<img src="{{ site.baseurl }}/images/banners/openldap-ubuntu.png" title="OpenLdap - Ubuntu Trusty" name="OpenLdap - Ubuntu Trusty" />

## Necesidad

Autenticar los usuarios existentes en el directorio **_OpenLdap_** en las máquinas con sistema operativo **_Ubuntu 14.04_**.

## Entorno

**_Máquinas clientes_**

```
  SO: Ubuntu 14.04
```

**_Sistema OpenLdap_**

```
        OpenLdap: 2.4.31
Tipo de usuarios: posixAccount
```

## Solución

Instalar los siguientes paquetes. Durante la instalación deberá ingresar los datos referentes a la configuración del sistema OpenLdap.

```
sudo apt-get update
sudo apt-get install libpam-ldap nscd sysv-rc-conf
```

Modificar el fichero `/etc/nsswitch.conf` para garantizar que los usuarios sean buscados dentro del directorio OpenLdap. Debe **_adicionar_** `ldap` al final de cada línea.

```
passwd:        compat  ldap
group:         compat  ldap
shadow:        compat  ldap
```

En el fichero `/etc/pam.d/common-password` se debe eliminar el texto `use_authtok`. La línea modificada debe quedar de la siguiente forma:

```
password        [success=1 user_unknown=ignore default=die]     pam_ldap.so try_first_pass
```

Al final del fichero `/etc/pam.d/common-session` adicionar la siguiente línea:

```
session required    pam_mkhomedir.so skel=/etc/skel umask=0077
```

Activar el módulo `libnss-ldap` con el siguiente comando:

```
sysv-rc-conf libnss-ldap on
```

Reiniciar el sistema operativo

```
sudo reboot
```

**_Listo!!! Cliente Ubuntu 14.04 configurado!!!_**