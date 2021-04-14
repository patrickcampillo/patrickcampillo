---
title: Hardening de Apache
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../images/
typora-root-url: ../
---

## Configuración inicial

Para comenzar a añadir la seguridad a nuestro servidor **Apache**, desactivaremos el módulo, que viene activado por defecto, que crea un índice del contenido de una carpeta si esta no posee un fichero `index.html` o `index.php`. Este módulo se llama `mod_autoindex`, y lo encontraremos con el nombre `autoindex.load` en la carpeta `mods-enabled`. Tras esto deberemos reiniciar **Apache**.

![](/images/hardeningapache/1.PNG)



Si lo comprobamos, veremos que si realizamos la búsqueda de la carpeta `public_html`, no la encontrará.

![](/images/hardeningapache/1-1.PNG)

Sin embargo, si buscamos la página `login.php`, que se encuentra dentro de la carpeta si la mostrará.

![](/images/hardeningapache/1-2.PNG)





## Cabecera del Servidor

**Apache**, por defecto, muestra en la cabecera `Server` del navegador, su versión y el sistema operativo al que pertenece.

![](/images/hardeningapache/2.PNG)



Para solucionar esto deberemos añadir las líneas "`ServerTokens ProductOnly`" y "`ServerSignature Off`" al fichero de configuración de **Apache**, `apache2.conf`.

![](/images/hardeningapache/3.PNG)



Y como podremos observar al reiniciar el servicio, ya solo mostrará que se trata de un **Apache**.

![](/images/hardeningapache/3-1.PNG)







## HSTS y CSP

**HSTS** es una cabecera, que le indica al navegador, que durante un tiempo establecido sólo puede actuar mediante HTTPS. Y la cabecera **CSP**, le indica al navegador, desde que origen puede cargar contenido. Ya sean imágenes, scripts, etc. Para aplicarlas,  solo deberemos añadir estas dos siguientes líneas a nuestro archivo de host virtual por defecto:

![](/images/hardeningapache/7.PNG)



Y además, para que el navegador pueda verificar al servidor, deberemos crear certificados autofirmados y habilitar el módulo **SSL** de **Apache**. Y, como tenemos que realizarlo en una imagen, he decidido utilizar los certificados que suelen llevar por defecto, para estas situaciones.

![](/images/hardeningapache/12.PNG)



Y utilizando, el script `run.sh`, le indicaremos que deberá escuchar en dos puertos, uno para el 80 y otro para el 443. Pero como con el **HSTS**, le hemos indicado que solo permitiremos al navegador mostrar el contenido mediante HTTPS, si se realiza alguna búsqueda al puerto 80, este nos informará de que el servidor esta diseñado para solo manejar tráfico por el puerto 443.



Script `run.sh`:

![](/images/hardeningapache/8.PNG)



Comprobación de XSS en la página post, enviando el código `<script>alert(1)</script>`.

![](/images/hardeningapache/9.PNG)









## Web Application Firewall (WAF) con OWASP

Para añadir más seguridad a nuestro servidor, utilizaremos dos reglas de seguridad que tiene la configuración **OWASP**. Para esto, deberemos instalar el paquete `libapache2-mod-security2`. El cual, al configurarlo e instalarlo, nos aplicará dos reglas para fortificar más nuestro servidor.

![](/images/hardeningapache/4.PNG)



Una vez instalado, deberemos modificar el valor de "`SecRuleEngine`" a "`On`", del archivo de configuración de `modsecurity`. Ubicado en `/etc/modsecurity`.

![](/images/hardeningapache/5.PNG)



Tras esto reiniciamos el servidor y habremos aplicado las reglas. Ya que este, ya viene configurado con 3  archivos de configuración, que almacenan la mayoría de reglas importantes a la hora de implantar el **OWASP**. De esta forma, si deseáramos añadir más reglas, solo tendríamos que añadirlas a esta carpeta. Esta carpeta se encuentra en la ruta `/etc/modsecurity/crs`, y contiene 3 archivos: `crs-setup.conf`, `RQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf` y `RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf`.

![](/images/hardeningapache/10.PNG)



He intentado seguir la guía que nos has marcado, pero al crearse ya esta carpeta en la instalación del **modsecurity**, al intentar reiniciar **Apache**, me provocaba errores porque encontraba **IDs** repetidos, o si borraba esta carpeta y creaba la llamada **rules**, me saltaba un error de que no encontraba estos archivos. Por lo que al final, he optado por dejarlo como viene por defecto. Ya que el resultado es aparentemente el mismo, como muestro en las pruebas siguientes.

`?testparam=test`

![](/images/hardeningapache/11.PNG)



`?exec=/bin/bash`

![](/images/hardeningapache/11-1.PNG)



## Dockerfile

Por último, crearemos una imagen `Dockerfile`, que realice todos los pasos anteriores al realizar el "build" de la imagen. Por eso, algunas de las capturas que he ido mostrando pertenecían a un `Dockerfile`. El Dockerfile final, quedaría tal que así:



![](/images/hardeningapache/6.PNG)

![](/images/hardeningapache/6-1.PNG)



Además podéis descargarlo y probarlo desde el siguiente [link](https://github.com/patrickcampillo/hardeningapache).