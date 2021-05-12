---
title: BWAPP SQL Injection
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/bwappsqli/
typora-root-url: ../../
---

## SQL Injection (GET/Select)

En esta primera inyección, utilizaremos un ataque `UNION`, para conseguir información del entorno donde se esta ejecutando la aplicación. 



Para comenzar, realizaremos una búsqueda para saber cuantas columnas contiene la tabla principal. Para esto, utilizaremos `ORDER BY`, seguido de las posible cantidad de columnas. Cabe destacar, que en esta base de datos, nos permite introducir cláusulas `SQL` sin necesidad de utilizar `'`, y que los comentarios se realizan mediante el símbolo `#`. Por lo que, al llegar al número 8, podemos observar que nos devuelve un error, que dice "Columna 8 desconocida". Y si comprobamos con el número 7, no nos muestra ningún error. Confirmando que la tabla contiene 7 columnas.



![](/patrickcampillo/assets/img/bwappsqli/get-select-1.png)



![](/patrickcampillo/assets/img/bwappsqli/get-select-2.png)



Una vez conocemos este número, podemos probar a realizar un ataque `UNION`, para poder conocer el usuario al que pertenece la sesión de la conexión SQL. Introduciendo una número de película que no existe, como por ejemplo el 30 y realizando un `UNION`, seleccionando las 7 columnas e introduciendo la función `user()`, en la segunda columna, ya que la primera no se muestra, nos devuelve "root@localhost". Con lo que habremos completado un ataque de `SQL Injection` satisfactoriamente.



![](/patrickcampillo/assets/img/bwappsqli/get-select.png)