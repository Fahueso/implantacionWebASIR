# 4. Sentencias Preparadas

## 4.1 Introducción

En el capítulo anterior hemos aprendido a recuperar información desde MariaDB mediante consultas SQL. En el tema anterior vimos cómo recibir datos desde GET.

Por ejemplo:

```php
$nombre = $_GET["nombre"];

$sql = "
SELECT *
FROM clientes
WHERE nombre = '$nombre'
";
```

Aunque este código funciona correctamente, presenta un problema importante: el valor introducido por el usuario se incorpora directamente a la consulta SQL.

Si un usuario introduce datos inesperados o maliciosos, la consulta podría comportarse de forma distinta a la prevista.

Para evitar este problema utilizaremos **sentencias preparadas** (*prepared statements*), el mecanismo recomendado para trabajar con bases de datos desde PHP.

---

## 4.2 ¿Qué es una sentencia preparada?

Una sentencia preparada es una consulta SQL definida previamente que contiene marcadores de posición para los valores que se proporcionarán más adelante.

Por ejemplo, en lugar de escribir:

```sql
SELECT *
FROM clientes
WHERE id = 5;
```

utilizamos:

```sql
SELECT *
FROM clientes
WHERE id = ?;
```

El símbolo `?` actúa como marcador de posición (*placeholder*).

Más adelante sustituiremos ese marcador por un valor real.

Esta separación entre consulta y datos proporciona varias ventajas:

- Mayor seguridad.
- Menor riesgo de errores.
- Código más legible.
- Posibles optimizaciones internas del servidor de bases de datos.

---

## 4.3 Flujo de trabajo

El uso de sentencias preparadas sigue siempre el mismo proceso:

1. Preparar la consulta.
2. Asociar los parámetros.
3. Ejecutar la consulta.
4. Recuperar los resultados.

Visualmente:

```text
Preparar
    ↓
Asociar parámetros
    ↓
Ejecutar
    ↓
Obtener resultados
```

Aunque inicialmente parecen más complejas que las consultas simples, pronto veremos que siguen siempre el mismo patrón.

---

## 4.4 Preparación de la consulta

La preparación se realiza mediante:

```php
mysqli_prepare()
```

Ejemplo:

```php
$stmt = mysqli_prepare(
    $conexion,
    "SELECT *
     FROM clientes
     WHERE id = ?"
);
```

En este ejemplo:

- `$conexion` contiene la conexión activa.
- `$stmt` almacenará la sentencia preparada.
- `?` indica que posteriormente se proporcionará un valor.

En este punto la consulta todavía no se ha ejecutado.

---

## 4.5 Asociación de parámetros

Una vez preparada la consulta, debemos indicar qué valor ocupará cada marcador.

Para ello utilizamos:

```php
mysqli_stmt_bind_param()
```

Por ejemplo:

```php
$id = 1;

mysqli_stmt_bind_param(
    $stmt,
    "i",
    $id
);
```

El segundo parámetro indica el tipo de dato asociado.

Los códigos más habituales son:

| Código | Tipo |
|---------|---------|
| i | Integer |
| d | Double |
| s | String |
| b | Binary |

Por ejemplo:

```php
"i"
```

indica un número entero, mientras que:

```php
"s"
```

indica una cadena de texto.

---

## 4.6 Ejecución y recuperación de resultados

Una vez asociados los parámetros ejecutamos la consulta:

```php
mysqli_stmt_execute(
    $stmt
);
```

Posteriormente obtenemos el resultado:

```php
$resultado =
    mysqli_stmt_get_result(
        $stmt
    );
```

El objeto devuelto funciona exactamente igual que el obtenido mediante `mysqli_query()`.

Por tanto, podemos recorrerlo utilizando:

```php
while ($fila = mysqli_fetch_assoc($resultado)) {

    echo $fila["nombre"];

}
```

---

## 4.7 Ejemplo completo

El siguiente ejemplo recupera un cliente utilizando su identificador.

```php
<?php

include "conexion.php";

$conexion = conexion();

$id = 1;

$stmt = mysqli_prepare(
    $conexion,
    "SELECT *
     FROM clientes
     WHERE id = ?"
);

mysqli_stmt_bind_param(
    $stmt,
    "i",
    $id
);

mysqli_stmt_execute(
    $stmt
);

$resultado =
    mysqli_stmt_get_result(
        $stmt
    );

$fila =
    mysqli_fetch_assoc(
        $resultado
    );

echo $fila["nombre"];

mysqli_free_result(
    $resultado
);

mysqli_stmt_close(
    $stmt
);

mysqli_close(
    $conexion
);

?>
```

---

## 4.8 Parámetros de texto y múltiples parámetros

Las sentencias preparadas funcionan igualmente con cadenas de texto.

Por ejemplo:

```php
$nombre = "Ana";

$stmt = mysqli_prepare(
    $conexion,
    "SELECT *
     FROM clientes
     WHERE nombre = ?"
);

mysqli_stmt_bind_param(
    $stmt,
    "s",
    $nombre
);
```

También es posible utilizar varios parámetros.

Consulta:

```sql
SELECT *
FROM clientes
WHERE nombre = ?
AND telefono = ?;
```

Asociación de parámetros:

```php
mysqli_stmt_bind_param(
    $stmt,
    "ss",
    $nombre,
    $telefono
);
```

La primera `s` corresponde a `$nombre` y la segunda a `$telefono`.

El orden de los tipos debe coincidir exactamente con el orden de los marcadores.

---

## 4.9 Uso con formularios

Las sentencias preparadas muestran toda su utilidad cuando trabajamos con datos introducidos por los usuarios.

Supongamos el siguiente formulario:

```html
<form method="GET">

    Nombre:

    <input type="text"
           name="nombre">

    <input type="submit"
           value="Buscar">

</form>
```

Recuperamos el valor:

```php
$nombre = $_GET["nombre"];
```

Y ejecutamos la consulta:

```php
$stmt = mysqli_prepare(
    $conexion,
    "SELECT *
     FROM clientes
     WHERE nombre = ?"
);

mysqli_stmt_bind_param(
    $stmt,
    "s",
    $nombre
);

mysqli_stmt_execute(
    $stmt
);
```

De esta forma el valor proporcionado por el usuario nunca se incorpora directamente a la sentencia SQL.

---

## 4.10 Recuperación de múltiples resultados

Las sentencias preparadas también pueden devolver varios registros.

Por ejemplo, supongamos que queremos mostrar todos los clientes cuyo nombre sea "Ana".

```php
$nombre = "Ana";

$stmt = mysqli_prepare(
    $conexion,
    "SELECT *
     FROM clientes
     WHERE nombre = ?"
);

mysqli_stmt_bind_param(
    $stmt,
    "s",
    $nombre
);

mysqli_stmt_execute(
    $stmt
);

$resultado =
    mysqli_stmt_get_result(
        $stmt
    );
```

El resultado puede recorrerse exactamente igual que cualquier consulta realizada mediante `mysqli_query()`.

```php
while ($fila = mysqli_fetch_assoc($resultado)) {

    echo $fila["nombre"];

    echo "<br>";

}
```

También es habitual mostrar los datos dentro de una tabla HTML.

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

Observa que una vez obtenido el objeto `$resultado`, el tratamiento es exactamente igual que el aprendido en el tema anterior.

---

## 4.11 Liberación de recursos

Además de liberar el resultado y cerrar la conexión, también es recomendable cerrar la sentencia preparada.

```php
mysqli_stmt_close(
    $stmt
);
```

El orden habitual será:

```php
mysqli_free_result(
    $resultado
);

mysqli_stmt_close(
    $stmt
);

mysqli_close(
    $conexion
);
```

A partir de este momento utilizaremos sentencias preparadas siempre que una consulta incluya datos variables proporcionados por los usuarios.