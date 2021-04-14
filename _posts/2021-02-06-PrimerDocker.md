---
title: Primer contenedor Docker
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../images/
typora-root-url: ../
---

## Creación del contenedor

A partir de un fichero **Dockerfile**, lanzaremos un contenedor donde se ejecutará un servidor web **Apache** con el módulo de **php** instalado. Y le indicaremos que copie todos los ficheros de nuestro directorio actual, al directorio por defecto de **Apache**. En nuestro directorio real tendremos diversos scripts, los cuales nos permitirán probar algunas opciones de **Docker** .

![](/images/practicadocker/dockerfile.PNG)

 Una vez tenemos nuestro **Dockerfile**, lanzaremos el script **build.sh** para proceder a la creación de la imagen. Con la opción **-t**, le indicaremos el nombre que tendrá la imagen.

![](/images/practicadocker/build.PNG)

![](/images/practicadocker/2.png)





## Ejecutar el contenedor

Ahora, lo más normal(ya que estaríamos en producción) sería lanzar el contenedor con el script **debug.sh** para compbrobar que todo funciona correctamente. Que contiene lo siguiente:

![](/images/practicadocker/7.png)

Se nos quedaría el terminal esperando ya que hemos ejecutado el contenedor en modo **debug/foreground**. Y podríamos comprobar que la página web funciona.

![](/images/practicadocker/3.png)

Vemos que nuestra página web cuenta cada visita que se realiza, pero si pararamos el servidor este valor se reiniciaría. Por lo que para evitar esto deberíamos crear un volumen en el contenedor a partir de uno del huésped, donde se encontrará un fichero que guardé las visitas. Para esto tenemos el siguiente script, que es **persist.sh**:

![](/images/practicadocker/5.png)

De esta forma el número de visitas no se borraría cada vez que parasemos el contenedor. El único inconveniente sería que se lanza en primer plano y no podemos cerrar el terminal o seguir trabajando en el mismo. Por lo que, por último, tenemos el script **run.sh**, que hace lo mismo que el script anterior pero con la opción **--detach**, que lanza el contenedor en segundo plano. Y contiene lo siguiente:

![](/images/practicadocker/6.png)

De esta forma habremos creado nuestro primer contenedor.