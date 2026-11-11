# Examen de recuperación - PHP

## Contexto

Una cadena de gimnasios desea desarrollar una pequeña aplicación web para controlar el acceso de sus usuarios y la inscripción a actividades deportivas.

La aplicación estará formada por varios archivos PHP que deberán completarse utilizando formularios, arrays asociativos, sesiones y estructuras de control.

---

# Ejercicio 1. Control de acceso al gimnasio (6 puntos)

La aplicación utilizará los siguientes usuarios:

```php
$usuarios = [
    "mario" => "Monitor",
    "laura" => "Administración",
    "pedro" => "Socio"
];
```

La contraseña será común para todos los usuarios y se almacenará mediante una constante:

```php
define("CLAVE_ACCESO", "gimnasio2026");
```

---

## Archivo login.php

Este archivo ya está realizado y contiene el formulario de acceso.

```html
<form action="acceso.php" method="post">

    <label>Usuario:</label>
    <input type="text" name="usuario" required>

    <br>

    <label>Contraseña:</label>
    <input type="password" name="password" required>

    <br><br>

    <input type="submit" value="Entrar">

</form>
```

---

## Apartado 1. acceso.php (2 puntos)

Completa el archivo para que:

1. Recupere el usuario y la contraseña enviados mediante POST.
2. Compruebe que el usuario existe dentro del array `$usuarios`.
3. Compruebe que la contraseña coincide con la constante `CLAVE_ACCESO`.
4. Si las credenciales son válidas:
   - Guarde el usuario en la sesión.
   - Guarde el rol en la sesión.
   - Redirija a `panel.php`.
5. Si las credenciales son incorrectas:
   - Muestre un mensaje de error.
   - Muestre un enlace para volver al formulario de acceso.

### Código de partida

```php
<?php

session_start();

define("CLAVE_ACCESO", "gimnasio2026");

$usuarios = [
    "mario" => "Monitor",
    "laura" => "Administración",
    "pedro" => "Socio"
];

// TODO

?>
```

### Pistas

Recuperar datos enviados por POST:

```php
$usuario = $_POST["usuario"];
$password = $_POST["password"];
```

Comprobar si existe una clave en un array asociativo:

```php
isset($usuarios[$usuario])
```

Redirigir a una página:

```php
header("Location: panel.php");
exit;
```

---

## Apartado 2. panel.php (2 puntos)

Completa el archivo para que:

1. Compruebe si existe una sesión iniciada.
2. Si no existe usuario en sesión, redirija a `login.php`.
3. Muestre un mensaje de bienvenida.
4. Muestre información distinta según el rol almacenado en sesión.
5. Incluya un enlace para cerrar sesión.

### Información a mostrar

#### Monitor

- Ver clases programadas
- Registrar asistencia

#### Administración

- Gestionar cuotas
- Gestionar socios

#### Socio

- Consultar horario
- Reservar actividades

### Código de partida

```php
<?php

session_start();

// TODO comprobar si existe la sesión

?>

<h2>Panel privado</h2>

<?php

// TODO mostrar información según el rol

?>

<a href="logout.php">Cerrar sesión</a>
```

### Pista

Acceder a una variable almacenada en sesión:

```php
$_SESSION["usuario"]
$_SESSION["rol"]
```

---

## Apartado 3. logout.php (2 puntos)

Completa el archivo para que:

1. Elimine las variables de sesión.
2. Destruya la sesión.
3. Redirija al usuario a `login.php`.

### Código de partida

```php
<?php

session_start();

// TODO eliminar datos de sesión

// TODO destruir sesión

// TODO redirigir a login.php

?>
```

### Pistas

```php
session_unset();
session_destroy();
```

---

# Ejercicio 2. Inscripción a actividades (4 puntos)

Se dispone de un formulario que permitirá seleccionar varias actividades deportivas.

---

## Archivo actividades.php

Este archivo ya está realizado.

```html
<form action="resultado.php" method="post">

    Nombre:
    <input type="text" name="nombre">

    <br><br>

    <input type="checkbox" name="actividades[]" value="Spinning"> Spinning
    <br>

    <input type="checkbox" name="actividades[]" value="Yoga"> Yoga
    <br>

    <input type="checkbox" name="actividades[]" value="Zumba"> Zumba
    <br>

    <input type="checkbox" name="actividades[]" value="Musculación"> Musculación
    <br>

    <input type="checkbox" name="actividades[]" value="Natación"> Natación
    <br><br>

    <input type="submit" value="Enviar">

</form>
```

---

## Apartado 4. resultado.php (4 puntos)

Completa el archivo para que:

1. Recupere el nombre enviado por POST.
2. Recupere las actividades seleccionadas.
3. Muestre el nombre del socio.
4. Genere una tabla HTML con dos columnas:
   - Actividad
   - Categoría
5. Recorra las actividades utilizando un `foreach`.
6. Determine la categoría correspondiente a cada actividad.

### Categorías

| Actividad | Categoría |
|------------|------------|
| Spinning | Cardio |
| Zumba | Cardio |
| Yoga | Bienestar |
| Musculación | Fuerza |
| Natación | Acuática |
| Cualquier otra | Desconocida |

### Código de partida

```php
<?php

// TODO recuperar nombre

// TODO recuperar actividades

echo "<h3>Socio: $nombre</h3>";

echo "<table border='1'>";
echo "<tr>";
echo "<th>Actividad</th>";
echo "<th>Categoría</th>";
echo "</tr>";

// TODO recorrer actividades

// TODO determinar categoría

// TODO mostrar fila

echo "</table>";

?>
```

### Pista

Recorrer un array:

```php
foreach ($actividades as $actividad) {

}
```

Crear una fila de una tabla:

```php
echo "<tr>";
echo "<td>$actividad</td>";
echo "<td>$categoria</td>";
echo "</tr>";
```

---

# Conceptos evaluados

- Formularios HTML
- Método POST
- Arrays asociativos
- Constantes (`define`)
- Condicionales (`if`, `elseif`, `else`)
- Bucles (`foreach`)
- Sesiones PHP
- `session_start()`
- `$_SESSION`
- `isset()`
- Redirecciones con `header()`
- Generación dinámica de HTML
- Arrays enviados mediante checkboxes
```**Tiempo recomendado:** 60 minutos.
