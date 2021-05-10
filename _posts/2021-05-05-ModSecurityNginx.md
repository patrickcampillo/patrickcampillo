---
title: ModSecurity en Nginx
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
---

## Dockerfile

El siguiente código representa un `Dockerfile`, con el cual "buildeamos" una imagen de `nginx` con `ModSecurity`. Además contiene algunas configuraciones que hacen que `nginx` sea más seguro. En los puntos siguientes explicaré parte por parte este `Dockerfile`.

```dockerfile
#Indicar la primera imagen a utilizar, en este caso Debian, y establecer un Alias para después poder importar
#archivos desde este contenedor.
FROM debian AS builder

#Establecer variables de zona y para que no nos salgan errores de frontend
ENV TZ=Europe/Madrid
ENV DEBIAN_FRONTEND=noninteractive

#Establecemos el diretcorio por defecto
WORKDIR /var/www/html

#Copiamos el script de instalación de todos los módulos de modsecurity.
COPY modsec.conf /var/www/html

#Instalamos todos los paquetes necesarios y ejecutamos el script.
RUN apt-get update -y -qq >/dev/null \
    && apt-get install -y bison build-essential ca-certificates curl dh-autoreconf doxygen flex gawk git iputils-ping libcurl4-	gnutls-dev libexpat1-dev libgeoip-dev liblmdb-dev libpcre3-dev libpcre++-dev libssl-dev libtool libxslt1-dev libgd-dev libxml2 libxml2-dev libyajl-dev locales lua5.3-dev pkg-config wget zlib1g-dev zlibc nginx php7.3-fpm >/dev/null \
    && apt-get purge --auto-remove \
    && apt-get clean \
    && rm -r /var/lib/apt/lists/* \
    && bash modsec.conf


#Indicar la imagen a utilizar para el contenedor final, en este caso Debian
FROM debian

#Establecer variables de zona y para que no nos salgan errores de frontend
ENV TZ=Europe/Madrid
ENV DEBIAN_FRONTEND=noninteractive

#Directorio por defecto
WORKDIR /var/www/html

#Instalación de paquetes
RUN apt-get update -y -qq >/dev/null \
    && apt-get install -y nano nginx php7.3-fpm >/dev/null \
    && apt-get purge --auto-remove \
    && apt-get clean \
    && rm -r /var/lib/apt/lists/* \
    && mkdir /etc/nginx/ssl

#Copia de los archivos de la página, de toda la configuración de Modsecurity del contenedor "builder", y algunos ficheros para configurar nginx
COPY public_html  /var/www/html/public_html
COPY private /var/www/html/private
COPY index.php /var/www/html/index.php
COPY --from=builder /downloads/nginx-1.14.2/objs/ngx_http_modsecurity_module.so /usr/share/nginx/modules/
COPY --from=builder /downloads/owasp-modsecurity-crs /usr/local/owasp-modsecurity-crs
COPY --from=builder /usr/local/modsecurity /usr/local/modsecurity
COPY --from=builder /usr/local/lib /usr/local/lib
COPY --from=builder /usr/lib/x86_64-linux-gnu/ /usr/lib/x86_64-linux-gnu/
COPY --from=builder /downloads/ModSecurity/unicode.mapping /etc/nginx/modsec/
COPY --from=builder /lib/x86_64-linux-gnu /lib/x86_64-linux-gnu
COPY nginx.conf /etc/nginx/nginx.conf
COPY htpasswd /etc/nginx/ssl/.htpasswd
COPY modsecurity.conf-recommened /etc/nginx/modsec/modsecurity.conf
COPY main.conf /etc/nginx/modsec/main.conf
COPY default /etc/nginx/sites-available/default
COPY ssl-cert-snakeoil.pem /etc/nginx/ssl/ssl-cert-snakeoil.pem
COPY ssl-cert-snakeoil.key /etc/nginx/ssl/ssl-cert-snakeoil.key

#Cambiamos los permisos de la clave privada para añadir seguridad y actualizamos los enlaces de los archivos. Ya que hemos importado nuevos de otra máquina
RUN chmod 400 /etc/nginx/ssl/ssl-cert-snakeoil.key \
    && ldconfig

#En este caso, el entrypoint se lo he especificado directamente, porque mediante archivo no funcionaba
CMD service php7.3-fpm start && nginx -g "daemon off;"
```

<br>

<br>

## Imagen builder

En las primeras líneas, partimos de una imagen `Debian`,  en la cual realizaremos una instalación de todos los complementos necesarios para poner en funcionamiento la herramienta `ModSecurity` en `Nginx`. Para esto, a parte de utilizar paquetes oficiales, utilizaremos utilidades externas, la gran mayoría provenientes de `Github`. Y para reducir la cantidad de código del `Dockerfile`, este último paso lo realizaremos en un script de `bash`. El cual ejecutaremos una vez se finalice toda la instalación de los paquetes oficiales.



El siguiente código hace referencia, solo a la primera parte del Dockerfile. Donde podemos observar la gran cantidad de paquetes a instalar y como después llamamos a ejecutar el script mediante el comando `bash`.

```dockerfile
#Indicar la primera imagen a utilizar, en este caso Debian, y establecer un Alias para después poder importar
#archivos desde este contenedor.
FROM debian AS builder

#Establecer variables de zona y para que no nos salgan errores de frontend
ENV TZ=Europe/Madrid
ENV DEBIAN_FRONTEND=noninteractive

#Establecemos el diretcorio por defecto
WORKDIR /var/www/html

#Copiamos el script de instalación de todos los módulos de modsecurity.
COPY modsec.conf /var/www/html

#Instalamos todos los paquetes necesarios y ejecutamos el script.
RUN apt-get update -y -qq >/dev/null \
    && apt-get install -y bison build-essential ca-certificates curl dh-autoreconf doxygen flex gawk git iputils-ping libcurl4-	gnutls-dev libexpat1-dev libgeoip-dev liblmdb-dev libpcre3-dev libpcre++-dev libssl-dev libtool libxslt1-dev libgd-dev libxml2 libxml2-dev libyajl-dev locales lua5.3-dev pkg-config wget zlib1g-dev zlibc nginx php7.3-fpm >/dev/null \
    && apt-get purge --auto-remove \
    && apt-get clean \
    && rm -r /var/lib/apt/lists/* \
    && bash modsec.conf
```



Seguidamente pasaré a mostrar el código que se encuentra dentro del `bash` script. El cual, iré mostrando parte por parte para poder explicarlo de manera más cómoda e intuitiva.



Comenzamos clonando, compilando e instalando el repositorio de `ssdeep`, utilizando el comando `make`. Este repositorio, posee un software de `fuzzy hashing` el cual permite, posteriormente, a `ModSecurity` realizar comparaciones de hashes para evitar el spam y algunos tipos de malware.

```bash
#!/bin/bash
mkdir /downloads
cd /downloads
git clone https://github.com/ssdeep-project/ssdeep
cd ssdeep/
/bin/bash bootstrap
/bin/bash configure
make
make install
```



Seguidamente, nos aseguramos que nos encontramos en la carpeta creada "Downloads", y clonamos el repositorio de `ModSecurity`, realizamos distintos pasos para asegurarnos que es la última versión, utilizando diferentes comandos de `git`, y finalmente lo compilamos e instalamos con `make`.

```bash
cd /downloads
git clone https://github.com/SpiderLabs/ModSecurity 
cd ModSecurity 
git checkout -b v3/master origin/v3/master 
git submodule init 
git submodule update 
sh build.sh 
/bin/bash configure 
make
make install
```



Clonamos la última versión del conector de `ModSecurity` y `Nginx`, es decir, este repositorio proporciona un canal de comunicación entre el paquete `libmodsecurity` y el propio `Nginx`. Además, descargamos la misma versión de `Nginx` que tengamos instalada en el sistema, de la página oficial. Utilizando el comando `wget`, y descomprimimos el archivo descargado.

```bash
cd /downloads
git clone https://github.com/SpiderLabs/ModSecurity-nginx

cd /downloads
wget http://nginx.org/download/nginx-1.14.2.tar.gz
tar -zxvf nginx-1.14.2.tar.gz
```



Tras esto, viene una parte muy importante para que consigamos que `Nginx` pueda utilizar el módulo de `ModSecurity` posteriormente. Accediendo a la carpeta recién descomprimida utilizaremos el script llamado "configure" y deberemos indicarle todos los parámetros que podemos ver. Y finalmente compilar e instalar los módulos.

```bash
cd nginx-1.14.2
/bin/bash configure --add-dynamic-module=../ModSecurity-nginx --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-sWHVb6/nginx-1.14.2=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' \
 --with-ld-opt='-Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log \
 --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi \
 --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module \
 --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-mail=dynamic --with-mail_ssl_module
make modules
```



Por último, clonaremos el repositorio que almacena las reglas `OWASP`, y la configuración estándar de `ModSecurity`. Modificaremos sus nombres para que `ModSecurity` sea capaz de cargarlas posteriormente y ya tendremos casi finalizada la instalación.

```bash
cd /downloads
wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/v3.2.0.tar.gz
tar -zxvf v3.2.0.tar.gz
mv owasp-modsecurity-crs-3.2.0 owasp-modsecurity-crs
mv owasp-modsecurity-crs/crs-setup.conf.example owasp-modsecurity-crs/crs-setup.conf
mv owasp-modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example  owasp-modsecurity-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
```







## Imagen principal

Ahora, a partir de lo realizado antes, generaremos la imagen pero sin necesidad de utilizar tanta memoria, ya que podremos utilizar toda la estructura generada en el `builder`. Esta parte, también partirá de una imagen inicial de `Debian`, pero esta vez solo instalaremos lo que realmente necesitamos, `Nginx` y `PHP`. Que para que este último funcione, deberemos utilizar su variante `FastCGI Process Manager`, la cual esta pensada para entornos web de tráfico elevado. Aunque no es nuestro caso, `Nginx` requiere este paquete en vez del estándar. Y además, crearemos una carpeta dentro del directorio por defecto de `Nginx` llamado "ssl", en el cual añadiremos las claves autofirmadas para el acceso `https`.

```dockerfile
#Indicar la imagen a utilizar para el contenedor final, en este caso Debian
FROM debian

#Establecer variables de zona y para que no nos salgan errores de frontend
ENV TZ=Europe/Madrid
ENV DEBIAN_FRONTEND=noninteractive

#Directorio por defecto
WORKDIR /var/www/html

#Instalación de paquetes
RUN apt-get update -y -qq >/dev/null \
    && apt-get install -y nano nginx php7.3-fpm >/dev/null \
    && apt-get purge --auto-remove \
    && apt-get clean \
    && rm -r /var/lib/apt/lists/* \
    && mkdir /etc/nginx/ssl
```



A esta parte le añadiremos toda la copia de ficheros, algunos que tendremos en nuestra máquina real, y otros que los importaremos desde el `builder`. Primero copiaremos los directorios de nuestro sitio web, separando el que contendrá el post público, del post privado, el cual protegeremos con contraseña como más tarde veremos. Tendremos que copiar los módulos y ficheros compilados o generados en el `builder`, utilizando la opción `--from` que nos ofrece el parámetro `COPY` de `Docker`. Después copiaremos el archivo principal de configuración de `Nginx`, el fichero que almacena el usuario y contraseña de nuestro sitio privado, la configuración de `ModSecurity`, nuestro host virtual por defecto y nuestros certificados autofirmados.

```dockerfile
#Copia de los archivos de la página, de toda la configuración de Modsecurity del contenedor "builder", y algunos ficheros para configurar nginx
COPY public_html  /var/www/html/public_html
COPY private /var/www/html/private
COPY index.php /var/www/html/index.php
COPY --from=builder /downloads/nginx-1.14.2/objs/ngx_http_modsecurity_module.so /usr/share/nginx/modules/
COPY --from=builder /downloads/owasp-modsecurity-crs /usr/local/owasp-modsecurity-crs
COPY --from=builder /usr/local/modsecurity /usr/local/modsecurity
COPY --from=builder /usr/local/lib /usr/local/lib
COPY --from=builder /usr/lib/x86_64-linux-gnu/ /usr/lib/x86_64-linux-gnu/
COPY --from=builder /downloads/ModSecurity/unicode.mapping /etc/nginx/modsec/
COPY --from=builder /lib/x86_64-linux-gnu /lib/x86_64-linux-gnu
COPY nginx.conf /etc/nginx/nginx.conf
COPY htpasswd /etc/nginx/ssl/.htpasswd
COPY modsecurity.conf-recommened /etc/nginx/modsec/modsecurity.conf
COPY main.conf /etc/nginx/modsec/main.conf
COPY default /etc/nginx/sites-available/default
COPY ssl-cert-snakeoil.pem /etc/nginx/ssl/ssl-cert-snakeoil.pem
COPY ssl-cert-snakeoil.key /etc/nginx/ssl/ssl-cert-snakeoil.key
```



Y finalmente, cambiamos los permisos de la clave privada para garantizar su seguridad, y actualizamos los enlaces simbólicos de los archivos que hemos importado del `builder` y han sustituido los locales. Además, deberemos indicar el `entrypoint` de nuestra imagen, es decir, que comando se ejecutará cuando lancemos un contenedor de esta imagen. Aquí estableceremos que se inicie el servicio de `PHP-fpm` y que `Nginx` deje de ejecutarse como demonio y se ejecute en primer plano.

```dockerfile
#Cambiamos los permisos de la clave privada para añadir seguridad y actualizamos los enlaces de los archivos. Ya que hemos importado nuevos de otra máquina
RUN chmod 400 /etc/nginx/ssl/ssl-cert-snakeoil.key \
    && ldconfig

#En este caso, el entrypoint se lo he especificado directamente, porque mediante archivo no funcionaba
CMD service php7.3-fpm start && nginx -g "daemon off;"
```





## Ficheros de configuración

Para finalizar, comentaré los cambios que hemos realizado sobre los ficheros de configuración de `Nginx`, para habilitar las diferentes funciones de seguridad. El primero es solo permitir el tráfico por `https` utilizando los certificados autofirmados. Esto lo estableceremos en el fichero `default` en la ruta `/etc/nginx/sites-available`. Además, también podemos observar las cabeceras `HSTS` y `CSP`, la configuración del directorio privado llamado `private` que utiliza el fichero `.httpaswd` para validar el acceso, y la habilitación de `ModSecurity` y la ruta donde se encuentran sus reglas. 

```nginx
server {
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
	add_header Content-Security-Policy "default-src 'self';" always;
	#listen 80 default_server;
	#listen [::]:80 default_server;

	# SSL configuration
	listen 443 ssl default_server;
	listen [::]:443 ssl default_server;
	ssl_certificate /etc/nginx/ssl/ssl-cert-snakeoil.pem;
	ssl_certificate_key /etc/nginx/ssl/ssl-cert-snakeoil.key;
	modsecurity on;
	modsecurity_rules_file /etc/nginx/modsec/main.conf;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.php index.html index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	location /private/ {
		auth_basic "Restricted";
  		auth_basic_user_file /etc/nginx/ssl/.htpasswd;
	}

	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/run/php/php7.3-fpm.sock;
	}
```



Por otro lado, también hemos de cargar el módulo de `ModSecurity` en el fichero `nginx.conf`, en la ruta `/etc/nginx/`

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
load_module modules/ngx_http_modsecurity_module.so;
include /etc/nginx/modules-enabled/*.conf;
```



Además comentar que en el fichero de configuración de `ModSecurity` establecemos `SecRuleEngine On` y `SecRequestBodyAccess On`, para que pueda aplicar las reglas al encontrar algún problema de seguridad.







## Conclusión

Con esto acabaríamos el reto de `Nginx` con `Modsecurity` en una imagen `Docker`. Ha sido un reto algo complicado por la gran cantidad de pasos a seguir para conseguir que funcione correctamente. Ya que al tener tanta configuración la cantidad de fallos que surgen también es muy grande. Pero, ha sido entretenido y útil, ya que no conocía como se configuraba `Nginx` y menos que este podía llegar a utilizar `ModSecurity`, ya que este esta pensando para `Apache`. Dejo el repositorio de `GitHub` con todos los archivos y configuraciones para realizar el "build" de la imagen en este [link](https://github.com/patrickcampillo/retonginx2.0). Si surge cualquier duda, no dudéis en preguntarme.

