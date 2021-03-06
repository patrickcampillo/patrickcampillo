---
title: WordPress con Docker-Compose
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/wordpress/
typora-root-url: ../../
---

## Primeros pasos

Creamos un directorio donde alojaremos los archivos necesarios para lanzar WordPress.

![](/patrickcampillo/assets/img/wordpress/1.png)



Creamos un fichero llamado `docker-compose.yml` que contiene lo siguiente:

![](/patrickcampillo/assets/img/wordpress/2.png)



Con esto lanzamos un contenedor con una base de datos **mysql**, donde le indicaremos su directorio por defecto y los parámetros que deseemos. Además de añadir **WordPress**, indicando el puerto y los parámetros necesarios para su ejecución.







## Construir el proyecto

Ahora, ejecutaremos el comando `docker-compose up -d` en el directorio que contiene el fichero `docker-compose-yml`. Mediante este comando, **Docker** procederá a descargar las imágenes si no las tiene, e iniciará los contenedores.

![](/patrickcampillo/assets/img/wordpress/3.png)

![](/patrickcampillo/assets/img/wordpress/3-1.png)



Al finalizar, podremos comprobar que todo funciona correctamente accediendo al puerto **8000** de nuestro servidor local.

![](/patrickcampillo/assets/img/wordpress/4.png)