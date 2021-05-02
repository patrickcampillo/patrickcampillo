---
title: Google Cloud
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/google-cloud/
typora-root-url: ../../
---

## Crear un proyecto

Para crear un proyecto, nos dirigiremos a nuestra organización y en el menú que nos aparecerá seleccionaremos **"Nuevo Proyecto"**.

![](/patrickcampillo/assets/img/google-cloud/1.png)



Completamos el siguiente menú con el nombre, la organización y la ubicación y seleccionamos **"Crear"**.

![](/patrickcampillo/assets/img/google-cloud/1-1.png)

 Tras esto deberemos habilitar el **"App Engine"**, para poder trabajar con nuestra aplicación. Este lo encontraremos en el panel izquierdo de la consola de **Google Cloud**. Una vez localizada seleccionaremos **"Panel de control"**.

![](/patrickcampillo/assets/img/google-cloud/1-2.png)

![](/patrickcampillo/assets/img/google-cloud/1-3.png)

 Nos aparecerá en pantalla un fondo azul dándonos la bienvenida, solo deberemos seleccionar el botón **"Crear aplicación"**.

![](/patrickcampillo/assets/img/google-cloud/1-4.png)

 Seleccionaremos el lenguaje y el *"Enviroment"*, que lo dejaremos en estándar.

![](/patrickcampillo/assets/img/google-cloud/1-5.png)







## Escribe tu servicio web con Node.js

Empezaremos creando una carpeta llamada `my-nodejs-service`, para nuestro servicio de **Node.js**. Dentro de esta crearemos un archivo vacío llamado `package.json`, mediante el comando `touch`. Y ejecutaremos el comando `npm install express`. 



Tras esto, comprobaremos en el archivo `package.json`, si ha aparecido el campo *"dependencies"*.

![](/patrickcampillo/assets/img/google-cloud/2.png)



Ahora agregaremos una secuencia de comandos `start ` al mismo archivo.

![](/patrickcampillo/assets/img/google-cloud/2-1.png)

Y finalmente, crearemos un archivo llamado ``server.js` que contendrá el siguiente código:

![](/patrickcampillo/assets/img/google-cloud/2-2.png)



Esta aplicación funciona de forma muy básica, solo responde a todas las solicitudes `GET`, con el texto *"Hello from App Engine"*.



Para ejecutar el servidor de forma local solo tendremos que introducir el comando `npm start`. Y comprobar el puerto **8080** de nuestro servidor local.

![](/patrickcampillo/assets/img/google-cloud/2-3.png)

![](/patrickcampillo/assets/img/google-cloud/2-3-1.png)



Seguidamente, agregaremos un fichero llamado `app.yaml`, para especificar la configuración de nuestro entorno. Y, por el momento, añadiremos lo siguiente:

![](/patrickcampillo/assets/img/google-cloud/2-4.png)







## Implementa tu servicio web

A continuación, vamos a implementar nuestro servicio mediante el comando `gcloud app deploy`, en el directorio que se encuentra el fichero `app.yaml`.

![](/patrickcampillo/assets/img/google-cloud/3.png)



Para verificar el funcionamiento de nuestro servicio, solo deberemos ejecutar el comando `gcloud app browse`.

![](/patrickcampillo/assets/img/google-cloud/3-1.png)

Y se nos abrirá una pestaña en el navegador que muestra nuestra aplicación.

![](/patrickcampillo/assets/img/google-cloud/3-1-1.png)







## Actualiza tu servicio web

Para permitir que un usuario pueda enviar datos al servidor, crearemos una carpeta llama `views` en la que crearemos un archivo llamado `form.html`, con el siguiente código:

![](/patrickcampillo/assets/img/google-cloud/4.png)



Y para que este se muestre en nuestra app, deberemos modificar el archivo `server.js`,  dejándolo como el que muestro a continuación:

![](/patrickcampillo/assets/img/google-cloud/4-2.png)



Y tras las modificaciones, volveremos a lanzar el comando `npm start`, y accederemos a nuestro formulario.

![](/patrickcampillo/assets/img/google-cloud/4-3.png)



Al enviar el formulario, nos deberían aparecer los datos en el terminal.

![](/patrickcampillo/assets/img/google-cloud/4-3-1.png)

Y en nuestra aplicación debería mostrar un mensaje de agradecimiento.

![](/patrickcampillo/assets/img/google-cloud/4-3-2.png)







## Personalización de la aplicación

Ahora vamos a modificar nuestro fichero `server.js`, para que cuando ejecutemos nuestra aplicación nos muestre el mensaje *"Hello from Ciberseguridad"*, si no se accede al formulario.

Para esto solo hay que modificar el valor de `res.send` en el primer `app.get`. Y quedaría tal que así:

![](/patrickcampillo/assets/img/google-cloud/solu1-1.png)

 Y al acceder a la aplicación desde nuestro servidor local recibiremos lo siguiente:

![](/patrickcampillo/assets/img/google-cloud/solu1.png)