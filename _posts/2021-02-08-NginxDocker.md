---
title: Nginx con Docker
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../images/
typora-root-url: ../
---
## Instalación

Para empezar con la práctica, lanzaremos un contenedor simple de Docker con las siguientes opciones:

- **--rm**: para borrar el contenedor una vez se pare.
- **-d**: para lanzar el contenedor en segundo plano.
- **-p**: para indicar el puerto, de la máquina real, por el cual se accede al contenedor. 
- **--name**: para indicar el nombre que le ponemos al contenedor.

Y accediendo vía web, podemos observar que funciona **nginx**.

![](/images/nginx/1.png)





## Agregar HTML personalizado

Ahora crearemos una página personalizada, la cual será mostrada de forma predeterminada por **nginx.** Para esto deberemos crearla dentro de un directorio llamado **site-content**.

![](/images/nginx/2.png)

Y volveremos a lanzar el comando de lanzamiento del contenedor de nginx, pero esta vez con la opción **-v**, para indicarle que la carpeta **site-content**, con nuestro archivo **index.html**, será lo que contenga el directorio por defecto de **nginx**.

![](/images/nginx/3.png)





## Crear una imagen personalizada

Finalmente, agilizaremos todo el proceso anterior creando un fichero **Dockerfile**, que contenga las opciones que deseemos para nuestro servidor web **nginx**. Y contendrá lo siguiente:

![](/images/nginx/4.png)

Ahora necesitaremos construir nuestra imagen a partir de este fichero, para esto solo necesitaremos el comando docker build, con la opción **-t** para indicarle el nombre de la imagen.

![](/images/nginx/5.png)

Por último, para comprobar que todo ha salido bien, ejecutaremos el comando, utilizado anteriormente, para lanzar el contenedor y no necesitaremos indicar la opción **-v**, ya que se lo hemos indicado en el **Dockerfile**.

![](/images/nginx/6.png)