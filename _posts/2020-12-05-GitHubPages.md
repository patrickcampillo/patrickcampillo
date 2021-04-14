---
title: GitHub Pages
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
---
<div class="style">
<h3>En este post explicaré como alojar nuestra web personal mediante GitHub Pages.</h3>
<!--more-->
    <b>1.</b> Crear nuestro repositorio en GitHub utilizando la siguiente sintaxis: <b>"tu-nombre.github.io"</b>.<br/>
<img src="/images/gpages1.png" alt="Primer paso"><br/>
    <b>2.</b> Ahora deberemos clonar este repositorio en nuestro equipo local. En el lugar que deseemos, en mi caso cree la carpeta github para facilitar su identificación. Mediante el comando <b>"git clone [url de nuestro repositorio]"</b>.<br/><br/>
<img src="/images/gpages2.png" alt="Segundo paso"><br/><br/>
    <b>3.</b> Una vez clonado podremos crear, editar, mover y eliminar los archivos que deseemos. Como ejemplo, crearé un archivo <b>"index.html"</b> para mostrar el funcionamiento de GitHub Pages.<br/><br/>
<img src="/images/gpages3.png" alt="Tercer paso"><br/><br/>
    <b>4.</b> Finalmente, tendremos que subir el archivo modificado a nuestro repositorio de GitHub. Para esto utilizaremos los siguientes comandos: 
<ul>
    <li><b>git add --all</b>, indicamos que queremos añadir a nuestro repositorio todo lo que hay en el reppositorio clonado.</li>
    <li><b>git commit -m "Initial commit"</b>, mediante este comando guardaremos las modificaciones indicando la razón mediante un pequeño comentario.</li>
    <li><b>git push -u origin master</b>, utilizando este comando subiremos los archivos, tras ejecutarlo nos pedriá nuestras credenciales de GitHub por seguridad.</li>
</ul><br/><br/>
<img src="/images/gpages4.png" alt="Cuarto paso"><br/><br/>
Y comprobaremos que todo ha salido de forma correcta, introduciendo la dirección de nuestro repositorio en nuestro buscador.<br/><br/>
<img src="/images/gpages5.png" alt="Comprobación">
</div>
