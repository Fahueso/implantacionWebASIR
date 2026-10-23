# 3. Recuperación de Datos mediante SELECT

## 3.1 Introducción

Una vez establecida la conexión entre PHP y MariaDB, ya podemos recuperar información almacenada en la base de datos.

La operación de lectura es una de las más utilizadas en cualquier aplicación web.

Algunos ejemplos habituales son:

- Mostrar el catálogo de productos de una tienda online.
- Listar los usuarios registrados de una plataforma.
- Consultar incidencias de un sistema de soporte.
- Visualizar reservas de una aplicación de gestión.

Todas estas tareas tienen algo en común: necesitan obtener información desde una base de datos y mostrarla al usuario.

Para ello utilizaremos consultas SQL de tipo `SELECT`.

---

## 3.2 Ejecución de consultas con mysqli_query()

Una vez disponemos de una conexión activa, podemos ejecutar consultas SQL mediante la función:

```php
mysqli_query()
```

Su sintaxis es:

```php
mysqli_query(
    $conexion,
    $consulta
);
```

Donde:

- `$conexion` contiene la conexión abierta con MariaDB.
- `$consulta` contiene una sentencia SQL.

Por ejemplo:

```php
<?php

include "conexion.php";

$conexion = conexion();

$sql = "SELECT * FROM clientes";

$resultado = mysqli_query(
    $conexion,
    $sql
);

?>
```

La consulta solicita todos los registros de la tabla `clientes`.

---

## 3.3 El resultado de una consulta

La función `mysqli_query()` no devuelve directamente los registros.

Devuelve un objeto resultado que contiene el conjunto de filas obtenido por MariaDB.

```php
$resultado = mysqli_query(
    $conexion,
    $sql
);
```

Para acceder a cada registro debemos recorrer ese resultado utilizando funciones específicas.

La más habitual es:

```php
mysqli_fetch_assoc()
```

Esta función recupera una fila y la devuelve como un array asociativo.

Por ejemplo, si la tabla contiene:

| id | nombre | telefono |
|----|---------|----------|
| 1 | Ana | 600111111 |

al ejecutar:

```php
$fila = mysqli_fetch_assoc(
    $resultado
);
```

obtendremos:

```php
Array
(
    [id] => 1
    [nombre] => Ana
    [telefono] => 600111111
)
```

Cada columna puede accederse mediante su nombre:

```php
echo $fila["nombre"];
```

Resultado:

```text
Ana
```

---

## 3.4 Recuperación de registros

### Un único registro

Si sabemos que la consulta devolverá una única fila, podemos recuperarla directamente:

```php
<?php

include "conexion.php";

$conexion = conexion();

$sql = "
SELECT *
FROM clientes
WHERE id = 1
";

$resultado = mysqli_query(
    $conexion,
    $sql
);

$fila = mysqli_fetch_assoc(
    $resultado
);

echo $fila["nombre"];

mysqli_close($conexion);

?>
```

Salida:

```text
Ana
```

### Múltiples registros

Cuando una consulta devuelve varias filas utilizamos un bucle `while`.

```php
while ($fila = mysqli_fetch_assoc($resultado)) {

    echo $fila["nombre"];

}
```

Cada llamada a `mysqli_fetch_assoc()` obtiene una nueva fila.

Cuando ya no existen más registros, la función devuelve:

```php
false
```

y el bucle finaliza automáticamente.

---

## 3.5 Mostrar resultados en HTML

Normalmente los datos recuperados se integran dentro de páginas HTML.

### Lista HTML

```php
<ul>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<li>";
    echo $fila["nombre"];
    echo "</li>";

}

?>

</ul>
```

Resultado aproximado:

```html
<ul>
    <li>Ana</li>
    <li>Pedro</li>
    <li>Marta</li>
</ul>
```

### Tabla HTML

Una de las formas más habituales de presentar información procedente de una base de datos es mediante tablas.

```php
<table>

<tr>
    <th>ID</th>
    <th>Nombre</th>
    <th>Teléfono</th>
</tr>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<tr>";

    echo "<td>";
    echo $fila["id"];
    echo "</td>";

    echo "<td>";
    echo $fila["nombre"];
    echo "</td>";

    echo "<td>";
    echo $fila["telefono"];
    echo "</td>";

    echo "</tr>";

}

?>

</table>
```

---

## 3.6 Liberación de recursos

Cuando terminamos de trabajar con un resultado es recomendable liberar los recursos asociados.

```php
mysqli_free_result(
    $resultado
);
```

Posteriormente cerraremos la conexión:

```php
mysqli_close(
    $conexion
);
```

El orden habitual será:

```php
mysqli_free_result(
    $resultado
);

mysqli_close(
    $conexion
);
```

---

## 3.7 Ejemplo completo

El siguiente programa muestra todos los clientes almacenados en la base de datos.

```php
<?php

include "conexion.php";

$conexion = conexion();

$sql = "SELECT * FROM clientes";

$resultado = mysqli_query(
    $conexion,
    $sql
);

?>

<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>Clientes</title>
</head>

<body>

<h1>Listado de clientes</h1>

<table border="1">

<tr>
    <th>ID</th>
    <th>Nombre</th>
    <th>Teléfono</th>
</tr>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<tr>";

    echo "<td>";
    echo $fila["id"];
    echo "</td>";

    echo "<td>";
    echo $fila["nombre"];
    echo "</td>";

    echo "<td>";
    echo $fila["telefono"];
    echo "</td>";

    echo "</tr>";

}

?>

</table>

</body>
</html>

<?php

mysqli_free_result(
    $resultado
);

mysqli_close(
    $conexion
);

?>
```

Este ejemplo representa el flujo completo de lectura de información:

1. Se abre la conexión.
2. Se ejecuta una consulta SQL.
3. Se recuperan los registros.
4. Se genera HTML.
5. Se liberan los recursos.
6. Se cierra la conexión.

---

## 3.8 Filtrado de resultados

Hasta ahora hemos recuperado todos los registros de una tabla:

```php
$sql = "SELECT * FROM clientes";
```

En aplicaciones reales normalmente necesitaremos recuperar únicamente la información que nos interesa.

Para ello utilizaremos cláusulas como:

- `WHERE`
- `ORDER BY`
- `LIMIT`

### Filtrado mediante WHERE

La cláusula `WHERE` permite seleccionar únicamente los registros que cumplen una condición.

```sql
SELECT *
FROM clientes
WHERE id = 1;
```

En PHP:

```php
$sql = "
SELECT *
FROM clientes
WHERE id = 1
";
```

También podemos filtrar por columnas de texto:

```sql
SELECT *
FROM clientes
WHERE nombre = 'Ana';
```

```php
$sql = "
SELECT *
FROM clientes
WHERE nombre = 'Ana'
";
```

Los valores de texto deben escribirse entre comillas simples dentro de la sentencia SQL.

---

## 3.9 Ordenación de resultados

La cláusula `ORDER BY` permite ordenar los registros obtenidos.

Por ejemplo:

```sql
SELECT *
FROM clientes
ORDER BY nombre;
```

El orden ascendente es el comportamiento predeterminado.

Si queremos ordenar de forma descendente:

```sql
SELECT *
FROM clientes
ORDER BY nombre DESC;
```

Resultado aproximado:

```text
Pedro
Marta
Ana
```

---

## 3.10 Limitación de resultados

En ocasiones necesitamos recuperar únicamente una parte de los registros.

Para ello utilizamos la cláusula `LIMIT`.

```sql
SELECT *
FROM clientes
LIMIT 5;
```

La consulta devolverá únicamente los cinco primeros registros.

En PHP:

```php
$sql = "
SELECT *
FROM clientes
LIMIT 5
";
```

Esta técnica resulta muy útil para crear sistemas de paginación.

---

## 3.11 Búsquedas desde formularios

Una necesidad habitual consiste en permitir al usuario buscar información.

Por ejemplo:

```html
<form method="GET">

    Nombre:

    <input type="text"
           name="nombre">

    <input type="submit"
           value="Buscar">

</form>
```

Cuando el usuario envía el formulario, el dato estará disponible mediante:

```php
$nombre = $_GET["nombre"];
```

Podemos utilizar dicho valor para construir una consulta:

```php
$sql = "
SELECT *
FROM clientes
WHERE nombre = '$nombre'
";

$resultado = mysqli_query(
    $conexion,
    $sql
);
```

Posteriormente mostraremos los registros exactamente igual que en cualquier consulta `SELECT`.

---

## 3.12 Un problema de seguridad

Las consultas anteriores funcionan correctamente, pero presentan un importante problema.

```php
$nombre = $_GET["nombre"];

$sql = "
SELECT *
FROM clientes
WHERE nombre = '$nombre'
";
```

El valor introducido por el usuario pasa directamente a formar parte de la consulta SQL.

Esto significa que un usuario podría introducir datos especialmente diseñados para alterar el comportamiento de la consulta.

Esta vulnerabilidad recibe el nombre de **inyección SQL** (*SQL Injection*).

Para evitar este problema utilizaremos **sentencias preparadas**, que permiten separar la consulta SQL de los datos proporcionados por el usuario.

En el siguiente capítulo aprenderemos a utilizarlas para construir aplicaciones más seguras y robustas.