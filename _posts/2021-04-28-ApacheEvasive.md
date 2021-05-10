---
title: ModSecurity en Nginx
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/evasive_apache/
typora-root-url: ../../
---

## Dockerfile

En esta práctica vamos a realizar la instalación de un servidor web `Apache` pero utilizando el módulo `mod_evasive`. Este módulo permite evitar ataques de denegación de servicio (`DoS`), realizando un escaneo constante de las conexiones entrantes. Si estas conexiones, superan el umbral establecida las baneara y ya no podrán acceder.

```dockerfile
# we will inherit from  the Debian image on DockerHub
FROM debian

# set timezone so files' timestamps are correct
ENV TZ=Europe/Madrid

# install apache and mod-evasive
RUN apt-get update -y -qq >/dev/null \
    && apt-get install -y apache2 libapache2-mod-evasive >/dev/null \
    && apt-get purge --auto-remove \
    && apt-get clean \
    && mkdir -p /var/log/mod_evasive \
    && chown -R root:www-data /var/log/mod_evasive

# HTML server directory
WORKDIR /var/www/html

#COPY of the entrypoint file, and the evasive configuration file
COPY entrypoint.sh /var/www/html/
COPY evasive.conf /etc/apache2/mods-available/evasive.conf

#RUN a restart of Apache
RUN service apache2 restart

#Entrypoint execution
CMD  ["./entrypoint.sh"]
```



Utilizando la imagen base `Debian`, realizaremos una actualización e instalación de los paquetes necesarios para realizar la práctica. Estableceremos el directorio por defecto y copiaremos en el el `entrypoint.sh`. Finalmente copiaremos el fichero configuración `evasive.conf`, que contiene los párametros que configuran el módulo, reiniciamos apache y ejecutamos el `enrypoint.sh`, que eejcutará `Apache` en primer plano.



## Fichero de configuración del módulo "evasive"

```bash
<IfModule mod_evasive20.c>
    DOSHashTableSize    2048
    DOSPageCount        5
    DOSSiteCount        100
    DOSPageInterval     1
    DOSSiteInterval     2
    DOSBlockingPeriod   10

    #DOSWhitelist
    #DOSEmailNotify      you@yourdomain.com
    #DOSSystemCommand    "su - someuser -c '/sbin/... %s ...'"
    DOSLogDir           "/var/log/mod_evasive"
</IfModule>
```

Este es el fichero en el que configuramos la cantidad de conexiones, el intervalo entre estas, y el tiempo de bloqieo. Este fichero se copiará al directorio por defecto, de `Apache`, que almacena los módulos.



## Informe de Apache Bench

El propio `Apache`, dispone de una herramienta para realizar pruebas de rendimiento de nuestro servidor. Esta herrameienta se llama `Apache Bench`, y la podemos utilizar mediante el comando `ab`, seguido del número de intentos y repeticiones.

![](/patrickcampillo/assets/img/evasive_apache/1.png)



Pudiendo obtener el informe del rendimiento de nuestro servidor web. Además si visualizamos el fichero  de logs `error.log`, podremos observar como bloquea el equipo.

![](/patrickcampillo/assets/img/evasive_apache/1-1.png)



## GitHub

Finalmente, dejo un repositorio de `GitHub`, con el `Dockerfile` y todos sus ficheros de configuración para poder "buildear" la imagen. Así como el fichero error.log, donde se puede comprobar como funciona este módulo.