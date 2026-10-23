# 2. Conexión a MariaDB desde PHP

## 2.1 La extensión mysqli

PHP dispone de varias formas de acceder a bases de datos. Cuando trabajamos con MariaDB o MySQL, la opción más habitual es la extensión **mysqli** (*MySQL Improved*).

Esta extensión proporciona funciones para:

- Conectarse a un servidor MariaDB.
- Ejecutar consultas SQL.
- Recuperar resultados.
- Insertar registros.
- Modificar información.
- Eliminar registros.

A lo largo de esta unidad utilizaremos esta extensión para todas las operaciones de acceso a datos.

## 2.2 Parámetros necesarios para la conexión

Para establecer una conexión con MariaDB necesitamos conocer varios datos:

| Parámetro | Descripción |
|-----------|-------------|
| Servidor | Equipo donde se ejecuta MariaDB |
| Usuario | Usuario autorizado para acceder |
| Contraseña | Clave del usuario |
| Base de datos | Base de datos con la que se trabajará |

En muchos entornos de desarrollo, tanto el servidor web como MariaDB se ejecutan en la misma máquina. En ese caso suele utilizarse:

```text
localhost
```

como dirección del servidor.

## 2.3 Primera conexión

La función utilizada para abrir una conexión es:

```php
mysqli_connect()
```

Su sintaxis básica es:

```php
mysqli_connect(
    servidor,
    usuario,
    contraseña,
    base_de_datos
);
```

Por ejemplo:

```php
<?php

$conexion = mysqli_connect(
    "localhost",
    "usuario",
    "password",
    "empresa"
);

if (!$conexion) {

    die(
        "Error de conexión: "
        . mysqli_connect_error()
    );

}

echo "<h1>Conexión realizada correctamente</h1>";

?>
```

Si la conexión se realiza correctamente, la variable `$conexion` contendrá toda la información necesaria para comunicarse con MariaDB.

### Errores de conexión

Una conexión puede fallar por diversos motivos:

- El servidor MariaDB está apagado.
- El usuario no existe.
- La contraseña es incorrecta.
- La base de datos no existe.
- Existen problemas de red.

Por este motivo siempre debemos comprobar el resultado de la conexión.

La función:

```php
mysqli_connect_error()
```

devuelve una descripción del error producido, mientras que:

```php
die()
```

finaliza inmediatamente la ejecución del script.

## 2.4 Cierre de la conexión

Cuando terminamos de trabajar con la base de datos es recomendable cerrar la conexión mediante:

```php
mysqli_close($conexion);
```

Por ejemplo:

```php
<?php

$conexion = mysqli_connect(
    "localhost",
    "usuario",
    "password",
    "empresa"
);

if (!$conexion) {

    die(
        "Error de conexión"
    );

}

echo "Conexión correcta";

mysqli_close($conexion);

?>
```

En aplicaciones sencillas el cierre suele realizarse automáticamente al finalizar el script, pero cerrarlo explícitamente es una buena práctica.

## 2.5 Reutilización del código de conexión

Una aplicación web suele estar formada por numerosos archivos:

```text
index.php
clientes.php
insertar.php
editar.php
borrar.php
```

Todos ellos necesitarán conectarse a la misma base de datos.

Copiar el mismo código en cada archivo dificultaría el mantenimiento. La solución consiste en centralizar la conexión en un único fichero.

Por ejemplo, podemos crear un archivo llamado `conexion.php`:

```php
<?php

function conexion()
{

    $conexion = mysqli_connect(
        "localhost",
        "usuario",
        "password",
        "empresa"
    );

    if (!$conexion) {

        die(
            "Error de conexión: "
            . mysqli_connect_error()
        );

    }

    return $conexion;

}

?>
```

La función devuelve una conexión lista para ser utilizada.

## 2.6 Utilización de la función de conexión

Cualquier página de la aplicación puede reutilizar la función anterior mediante `include`.

```php
<?php

include "conexion.php";

$conexion = conexion();

echo "Base de datos conectada";

mysqli_close($conexion);

?>
```

La principal ventaja es que todos los parámetros de conexión se mantienen en un único fichero.

Si en el futuro cambia la contraseña, el nombre de la base de datos o el servidor, únicamente habrá que modificar un archivo.

## 2.7 Estructura básica de una aplicación

A medida que una aplicación crece resulta habitual separar cada responsabilidad en archivos diferentes.

Por ejemplo:

```text
proyecto/

├── conexion.php
├── index.php
├── clientes.php
├── insertar.php
├── editar.php
└── borrar.php
```

Esta organización aporta varias ventajas:

- Facilita el mantenimiento.
- Favorece la reutilización de código.
- Simplifica la localización de errores.
- Facilita el trabajo en equipo.

Durante el resto de la unidad seguiremos esta filosofía, manteniendo la conexión centralizada y reutilizando los componentes comunes siempre que sea posible.