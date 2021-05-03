---
title: Autenticación HTTP
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../assets/img/P1-autenticacion/
typora-root-url: ../../
---

## Autenticación HTTP

Para comprobar el funcionamiento de la autenticación **HTTP**, utilizaremos una página en `php` de autenticación básica. Esta página contiene lo siguiente:



```php
<?php

$valid_passwords = array ("mario" => "qwerty");
$valid_users = array_keys($valid_passwords);

$user = $_SERVER['PHP_AUTH_USER'];
$pass = $_SERVER['PHP_AUTH_PW'];

$validated = (in_array($user, $valid_users)) && ($pass == $valid_passwords[$user]);

if (!$validated) {
  header('WWW-Authenticate: Basic realm="My Realm"');
  header('HTTP/1.0 401 Unauthorized');
  die ("Not authorized");
}

// If it arrives here, it is a valid user.
echo "<p>Welcome $user.</p>";
echo "<p>Congratulation, you are into the system.</p>";
```



Una vez almacenada en el directorio por defecto de nuestro navegador web, podemos acceder a ella y comprobar que nos pide usuario y contraseña.

![](/patrickcampillo/assets/img/P1-autenticacion/1.png)



Lo introducimos:

![](/patrickcampillo/assets/img/P1-autenticacion/2.png)



Y podremos observar como nos identifica y nos muestra un mensaje de bienvenida.

![](/patrickcampillo/assets/img/P1-autenticacion/3.png)



El punto negativo de esto, es que al realizarse la autenticación, si nos encontramos en `http`, el cliente y el servidor se envían sus credenciales en texto plano. Por lo que es casi obligatorio, utilizar el protocolo `https`, para garantizar la total seguridad de nuestra web.