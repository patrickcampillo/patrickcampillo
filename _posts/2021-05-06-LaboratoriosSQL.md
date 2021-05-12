---
title: Laboratorios SQL Injection
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/laboratorios_sql/
typora-root-url: ../../
---

## Laboratorio 1

Este laboratorio contiene una vulnerabilidad de Inyección `SQL` en la cláusula `WHERE`, la cual permite, poder ver detalles de todos los productos de cualquier categoría en vez de la seleccionada.



La solución a este laboratorio es introducir al final de la búsqueda lo siguiente: `'+OR+1=1--`. Ya que, como 1 siempre va ser igual a 1 va a devolver todos los elementos de la lista. Y además al añadir `--`, le indicamos que la demás información que vaya después del `WHERE` es un comentario.

![](/patrickcampillo/assets/img/laboratorios_sql/1-1.png)

![](/patrickcampillo/assets/img/laboratorios_sql/1.png)



## Laboratorio 2

En el segundo laboratorio, nos encontramos con una vulnerabilidad de `SQL Injection` en el formulario de inicio de sesión. Lo cual podemos aprovechar para iniciar sesión con el usuario administrador sin conocer la contraseña.



Solo tendremos que introducir el nombre del usuario y insertar `--`, para comentar el resto de la consulta. Al poseer esta vulnerabilidad, obtendremos el acceso y por tanto habremos resuelto este laboratorio.

![](/patrickcampillo/assets/img/laboratorios_sql/2-1.png)

![](/patrickcampillo/assets/img/laboratorios_sql/2.png)







## Laboratorio 3

En este laboratorio, se nos presenta una vulnerabilidad de `SQL Injection` que permite el uso de ataques `UNION`, mediante los cuales podremos mostrar información de otras tablas diferentes.



Para poder realizar este ataque, primero deberemos conocer cuántas columnas devuelve la consulta normal de la página. Para esto, se utiliza la orden `ORDER BY`, seguida de un número y de `--`, para comentar el resto de la consulta. Se tiene que ir incrementando el número por el que ordenamos, hasta que la página muestre un error. Cuando esto suceda, sabremos que el número de columnas es el número anterior al del error. Y una vez lo conocemos, poder inyectar la cláusula UNION seguida de tantas columnas como las que tiene. En este caso solo teníamos que mostrar valores vacíos y para eso solo se tenía que introducir 3 `NULL`(las columnas que tenía la tabla principal), e igual que siempre seguido de `--`, para comentar el resto de la consulta. Y con esto habríamos resuelto el laboratorio.



![](/patrickcampillo/assets/img/laboratorios_sql/3-1.png)

![](/patrickcampillo/assets/img/laboratorios_sql/3.png)



## Laboratorio 4

Por último, en el cuarto laboratorio, nos encontramos de nuevo con una vulnerabilidad que nos permite realizar un ataque `UNION` para conocer el tipo y la versión de la base de datos en **Oracle**.



Para realizar este ataque, primero deberemos averiguar cómo están formadas las bases de datos de **Oracle**. Ya que, deberemos saber como poder encontrar los que nos pide el ejercicio, y lo más normal es no conocerlo. Una vez sabemos de qué forma almacena los datos pedidos **Oracle**, solo tendremos que comprobar las columnas de la tabla principal (2) y realizar un `UNION` con el `BANNER` y un campo vacío(`NULL`), sobre la tabla `v$version` y comentar el resto de la consulta. Y con esto habremos resuelto el último laboratorio.



![](/patrickcampillo/assets/img/laboratorios_sql/4-1.png)

![](/patrickcampillo/assets/img/laboratorios_sql/4.png)