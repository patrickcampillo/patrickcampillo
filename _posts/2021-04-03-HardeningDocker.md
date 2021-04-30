---
title: Hardening Docker
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../images/hardendocker/
typora-root-url: ../
---

## Seguridad Host

Para comenzar a utilizar Docker, es recomendable, ya sea por comodidad y seguridad. Poder ejecutar sus comandos sin necesidad de tener permisos de administrador. Por lo que, deberemos utilizar los 2 siguientes comandos: 

![](/1.png)



![](/patrickcampillo/images/hardendocker/2.png)

Con el primero, nos añadiremos al grupo **Docker**, y con el segundo actualizaremos la configuración de los grupos para no tener que reiniciar la sesión para conseguir los privilegios.







## Seguridad en el demonio

Ya que, el demonio de Docker, suele correrse en modo superusuario. Deberemos impedir que nadie pueda conseguir manipular la configuración del mismo. Este archivo se llama `daemon.json` y debe contener el parámetro `"debug": false`,  para evitar su ejecución en este modo. Además, hay otras posibilidades, como el parámetro `ulimits`, mediante el  cual, puedes definir el límite de archivos que puede cargar un contenedor. Y otro parámetro muy interesante, es `icc`, que impide la conexión entre los contendores sino se específica en su propia configuración. Y lo podríamos observar en el fichero de una forma parecida a la siguiente:

![](/patrickcampillo/images/hardendocker/3.png)







## Seguridad en contenedores

Los contenedores, son los ejes principales del bastionado, ya que es un objetivo potencial. Debido a la gran de configuraciones que se pueden ver afectadas si uno de estos falla, o se comporta de forma indeseada. Por lo que, al igual que antes, con el parámetro `ulimits`, podemos fijar un límite personalizado, dependiendo del contenedor que se quiera lanzar.

![](/patrickcampillo/images/hardendocker/4.png)



O de forma parecida, se le puede establecer un valor máximo de uso de CPU, por lo que no podría superarlo en ningún caso.

![](/patrickcampillo/images/hardendocker/5.png) 







## Privilegios

Cada vez que lanzamos un contenedor, el usuario por defecto que **Docker** utiliza es **root**. Pero esto podemos evitarlo utilizando el flag `-u uuid`. Por ejemplo, si deseáramos correr un contenedor con el usuario con **uuid** 4400, lanzaríamos el contenedor de la siguiente forma:

![](/patrickcampillo/images/hardendocker/6.png)

Observando, que como no tenemos permisos en el directorio por defecto de lanzamiento del contenedor, no nos permite, si quiera, realizar un **ls**.



Otro flag interesante, relacionado con el tema de permisos es `--privileged`,  con el cual, podemos obtener permisos para realizar una gran cantidad de acciones, las cuales no deberíamos poder realizar. Por lo que si ejecutamos un contenedor con esta opción y realizamos un montaje de una unidad, no nos mostrará ningún problema. 

![](/patrickcampillo/images/hardendocker/7.png)



Sin embargo, si no le indicamos esta opción, nos saldría el mensaje de que carecemos de los permisos necesarios.

![](/patrickcampillo/images/hardendocker/7-1.png)





Otra posibilidad, muy interesante, que nos da **Docker**, es poder montar nuestro propio socket Docker dentro de otro contenedor. Para esto deberemos utilizar el siguiente comando:

![](/patrickcampillo/images/hardendocker/8.png)



Y podremos observar que tenemos acceso a las imágenes que se encuentran fuera del contenedor y hasta lanzarlas si lo deseamos.







## Escaneo pasivo de vulnerabilidades

Una práctica muy útil a la hora de trabajar con imágenes de **Docker Hub**, es poder verificar que vulnerabilidades pueden poseer, ya sea por mala configuración, versiones no actualizadas, etc. Esto nos lo permite comprobar el propio **Docker Hub**, pero mediante la versión **Pro**. Pero, existen una gran cantidad de posibilidades gratuitas que realizan la misma función. Como es el caso de **Trivy**. Mediante el cual, podemos realizar un escaneo de vulnerabilidades de la gran mayoría de imágenes que se encuentran en **Docker Hub**. En este caso, ya que no he encontrado ninguna interesante de **WordPress**, he realizado el análisis de un **Alpine** con **Git**.

![](/patrickcampillo/images/hardendocker/9.png)



Pudiendo observar algunas vulnerabilidades de riesgo elevado.
