# 2. Conexión a MariaDB desde PHP

## 2.1 La extensión mysqli

PHP dispone de varias formas de acceder a bases de datos. Con MariaDB o MySQL, la más habitual es la extensión **mysqli** (*MySQL Improved*). Proporciona funciones para:

- Conectarse a un servidor MariaDB.
- Ejecutar consultas SQL.
- Recuperar resultados.
- Insertar, modificar y eliminar registros.

Usaremos `mysqli` en su estilo **procedural** (funciones `mysqli_*`) durante toda la unidad.

## 2.2 Parámetros necesarios para la conexión

| Parámetro | Descripción |
|-----------|-------------|
| Servidor | Equipo donde se ejecuta MariaDB |
| Usuario | Usuario autorizado para acceder |
| Contraseña | Clave del usuario |
| Base de datos | Base de datos con la que se trabajará |

Cuando el servidor web y MariaDB están en la misma máquina, el servidor es `localhost`.

## 2.3 Primera conexión

La función que abre una conexión es `mysqli_connect()`:

```php
mysqli_connect(servidor, usuario, contraseña, base_de_datos);
```

Ejemplo (solo para aprender; en el capítulo 8 veremos cómo evitar escribir la contraseña en el código):

```php
<?php

$conexion = mysqli_connect(
    "localhost",
    "app_web",
    "password_seguro",
    "empresa"
);

echo "<h1>Conexión realizada correctamente</h1>";

?>
```

Si la conexión se realiza correctamente, `$conexion` contiene lo necesario para comunicarse con MariaDB.

## 2.4 Errores de conexión: excepciones

Una conexión puede fallar por varios motivos:

- El servidor MariaDB está apagado.
- El usuario no existe o la contraseña es incorrecta.
- La base de datos no existe.
- Problemas de red.

### Qué ocurre en PHP 8.1 o superior

Desde PHP 8.1, `mysqli` **lanza excepciones** (`mysqli_sql_exception`) cuando algo falla. Esto significa que el clásico código:

```php
$conexion = mysqli_connect("localhost", "app_web", "mala_clave", "empresa");

if (!$conexion) {                       // ← nunca se ejecuta
    die("Error: " . mysqli_connect_error());
}
```

**no funciona como se esperaba**: si la conexión falla, PHP no devuelve `false`, sino que interrumpe el script con un *Fatal error: Uncaught mysqli_sql_exception*. Peor aún, con la configuración por defecto la traza del error puede mostrar en pantalla el usuario, el servidor e incluso la contraseña.

Vamos a trabajar **con excepciones**, que es la forma moderna y la que debes usar:

```php
<?php

// Activamos explícitamente el modo excepciones (es el comportamiento por defecto en PHP 8.1+,
// pero conviene dejarlo escrito para que el código funcione igual en cualquier versión)
mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

try {

    $conexion = mysqli_connect("localhost", "app_web", "password_seguro", "empresa");

    echo "Conexión correcta";

} catch (mysqli_sql_exception $e) {

    // Mensaje genérico para el usuario
    echo "No se pudo conectar con la base de datos.";

    // Detalle técnico solo para el administrador (queda en el log de errores del servidor)
    error_log("Error de conexión: " . $e->getMessage());

}

?>
```

Fíjate en dos ideas importantes:

- Al usuario final **nunca** se le muestra el mensaje técnico del error. Revela información útil para un atacante.
- El detalle se guarda con `error_log()`, que escribe en el registro de errores del servidor (en Debian/Ubuntu: `/var/log/apache2/error.log`).

> **Durante el desarrollo** puedes mostrar `$e->getMessage()` en pantalla para depurar más rápido, pero debes quitarlo antes de publicar la aplicación.

### Recordatorio: `try / catch`

```php
try {
    // código que puede fallar
} catch (mysqli_sql_exception $e) {
    // qué hacemos si falla
}
```

Si dentro del `try` se produce una excepción, PHP salta directamente al `catch`. Si no se produce, el `catch` se ignora.

## 2.5 Cierre de la conexión

Cuando terminamos de trabajar con la base de datos cerramos la conexión con `mysqli_close()`:

```php
mysqli_close($conexion);
```

En scripts sencillos PHP la cierra automáticamente al terminar, pero cerrarla explícitamente es una buena práctica.

## 2.6 Juego de caracteres (charset)

Después de conectar hay que indicar con qué codificación se comunicarán PHP y MariaDB. Si no lo hacemos, es muy probable que las tildes y la ñ se vean como `Ã±`, `Ã³`, etc.

```php
mysqli_set_charset($conexion, "utf8mb4");
```

`utf8mb4` es la codificación UTF-8 completa de MariaDB (incluye emojis y todos los caracteres). Debe coincidir con la codificación de las tablas (en el Anexo A se crean con `utf8mb4`) y con la de las páginas HTML (`<meta charset="UTF-8">`).

## 2.7 Reutilización del código de conexión

Una aplicación web suele estar formada por numerosos archivos. Todos necesitan conectarse a la misma base de datos. Copiar el código en cada archivo dificultaría el mantenimiento, así que lo **centralizamos** en un único fichero, `conexion.php`:

```php
<?php

mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

function conexion()
{

    try {

        $conexion = mysqli_connect(
            "localhost",
            "app_web",
            "password_seguro",
            "empresa"
        );

        mysqli_set_charset($conexion, "utf8mb4");

    } catch (mysqli_sql_exception $e) {

        error_log("Error de conexión: " . $e->getMessage());

        http_response_code(500);
        die("Error interno: no se pudo conectar con la base de datos.");

    }

    return $conexion;

}

?>
```

La función devuelve una conexión lista para usar (y ya configurada con el juego de caracteres correcto). Si algo falla, registra el motivo y detiene el script con un mensaje genérico.

## 2.8 Utilización de la función de conexión

Cualquier página puede reutilizarla con `require_once`:

```php
<?php

require_once "conexion.php";

$conexion = conexion();

echo "Base de datos conectada";

mysqli_close($conexion);

?>
```

> **¿`include` o `require_once`?**
> - `include` : si el fichero no existe, muestra un *warning* y **el script continúa**, para acabar fallando más adelante con un error confuso (`Call to undefined function conexion()`).
> - `require` : si el fichero no existe, el script se detiene en ese punto.
> - `require_once` : como `require`, pero evita cargar el mismo fichero dos veces (lo que daría un error por declarar la función `conexion()` repetida).
>
> Para ficheros imprescindibles como la conexión, usa siempre **`require_once`**.

Todos los parámetros de conexión están en un único sitio: si cambia la contraseña, la base de datos o el servidor, solo hay que modificar un archivo.

## 2.9 Estructura básica de una aplicación

Conforme la aplicación crece conviene separar responsabilidades en archivos distintos:

```text
proyecto/
├── conexion.php
├── index.php
├── clientes.php
├── insertar.php
├── actualizar.php
└── eliminar.php
```

Ventajas:

- Facilita el mantenimiento.
- Favorece la reutilización de código.
- Simplifica la localización de errores.
- Facilita el trabajo en equipo.

Esta es la estructura que usaremos en los capítulos 3 a 7. En el capítulo 8 la mejoraremos separando la configuración del código.

---

## Resumen

- `mysqli_connect()` abre la conexión; `mysqli_close()` la cierra.
- En PHP 8.1+ los errores de `mysqli` se notifican mediante **excepciones**: hay que usar `try / catch`, no `if (!$conexion)`.
- Nunca mostramos al usuario el mensaje técnico del error; lo guardamos con `error_log()`.
- Tras conectar se establece el juego de caracteres con `mysqli_set_charset($conexion, "utf8mb4")`.
- La conexión se centraliza en `conexion.php` y se carga con `require_once`.

## Actividades

1. Crea `conexion.php` con tus datos y comprueba que funciona. Después escribe una contraseña incorrecta a propósito y observa qué ocurre. ¿Dónde puedes leer el detalle del error?
2. Quita la línea `mysqli_set_charset(...)`, inserta un cliente llamado «Begoña» (lo haremos en el capítulo 5) y observa cómo se ve. Vuelve a ponerla.
3. Explica con tus palabras la diferencia entre `include`, `require` y `require_once`, y qué ocurriría si `conexion.php` tuviera un error de sintaxis con cada uno de ellos.
