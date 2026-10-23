# 7. Consultas Multitabla y JOIN

## 7.1 Introducción

Hasta ahora hemos trabajado con consultas que recuperan información de una única tabla.

Por ejemplo:

```sql
SELECT *
FROM clientes;
```

o bien:

```sql
SELECT *
FROM alumnos;
```

Sin embargo, las bases de datos relacionales obtienen gran parte de su potencia de la capacidad para relacionar información almacenada en distintas tablas.

Algunos ejemplos habituales son:

- Un alumno pertenece a un grupo.
- Un pedido pertenece a un cliente.
- Una incidencia pertenece a un usuario.
- Un libro pertenece a una categoría.

En todos estos casos la información se encuentra distribuida entre varias tablas relacionadas entre sí.

Para recuperar información procedente de varias tablas utilizaremos consultas **JOIN**.

---

## 7.2 ¿Por qué dividir la información en varias tablas?

Supongamos una aplicación para gestionar alumnos.

Una posible solución sería almacenar toda la información en una única tabla.

| id | nombre | grupo |
|----|---------|--------|
| 1 | Ana | ASIR |
| 2 | Pedro | ASIR |
| 3 | Marta | DAW |

Aunque este diseño funciona, presenta varios inconvenientes:

- Se repite información.
- Es más difícil mantener la consistencia.
- Aumenta el riesgo de errores al modificar datos.

Por ejemplo, si el nombre de un grupo cambia, habría que modificar todos los registros relacionados.

Por este motivo es habitual separar la información en varias tablas y relacionarlas entre sí.

---

## 7.3 Claves primarias y claves foráneas

Consideremos las siguientes tablas.

### Tabla grupos

```sql
CREATE TABLE grupos (

    id INT AUTO_INCREMENT PRIMARY KEY,

    nombre VARCHAR(100)

);
```

Contenido:

| id | nombre |
|----|---------|
| 1 | ASIR |
| 2 | DAW |
| 3 | SMR |

### Tabla alumnos

```sql
CREATE TABLE alumnos (

    id INT AUTO_INCREMENT PRIMARY KEY,

    nombre VARCHAR(100),

    grupo_id INT,

    FOREIGN KEY (grupo_id)
        REFERENCES grupos(id)

);
```

Contenido:

| id | nombre | grupo_id |
|----|---------|----------|
| 1 | Ana | 1 |
| 2 | Pedro | 1 |
| 3 | Marta | 2 |

En esta tabla:

- `id` es la clave primaria del alumno.
- `grupo_id` es una clave foránea.
- `grupo_id` apunta al identificador de la tabla `grupos`.

Gracias a esta relación podemos conectar la información de ambas tablas.

---

## 7.4 El problema que resuelve JOIN

Supongamos que queremos obtener el siguiente resultado:

| Alumno | Grupo |
|---------|---------|
| Ana | ASIR |
| Pedro | ASIR |
| Marta | DAW |

La tabla `alumnos` contiene los nombres de los alumnos, pero no los nombres de los grupos.

La tabla `grupos` contiene los nombres de los grupos, pero no los nombres de los alumnos.

Necesitamos combinar ambas tablas para obtener la información completa.

---

## 7.5 La cláusula JOIN

La cláusula `JOIN` permite combinar registros procedentes de varias tablas relacionadas.

Su sintaxis general es:

```sql
SELECT columnas
FROM tabla1
JOIN tabla2
ON condicion;
```

La cláusula `ON` indica cómo deben relacionarse ambas tablas.

---

## 7.6 Primer JOIN

Partiendo de las tablas anteriores:

```sql
SELECT

    alumnos.nombre,
    grupos.nombre

FROM alumnos

JOIN grupos
ON alumnos.grupo_id = grupos.id;
```

La condición:

```sql
alumnos.grupo_id = grupos.id
```

indica que ambas tablas deben vincularse mediante esos campos.

El resultado será:

| nombre | nombre |
|---------|---------|
| Ana | ASIR |
| Pedro | ASIR |
| Marta | DAW |

---

## 7.7 Alias de columnas

La consulta anterior presenta un inconveniente: ambas tablas tienen una columna llamada `nombre`.

Para evitar confusiones podemos utilizar alias mediante la palabra clave `AS`.

```sql
SELECT

    alumnos.nombre,

    grupos.nombre AS grupo

FROM alumnos

JOIN grupos
ON alumnos.grupo_id = grupos.id;
```

Resultado:

| nombre | grupo |
|---------|---------|
| Ana | ASIR |
| Pedro | ASIR |
| Marta | DAW |

Los alias permiten asignar nombres más claros a las columnas obtenidas por la consulta.

---

## 7.8 Ejecución de JOIN desde PHP

Las consultas con `JOIN` se ejecutan exactamente igual que cualquier otra consulta SQL.

```php
$sql = "

SELECT

    alumnos.nombre,

    grupos.nombre AS grupo

FROM alumnos

JOIN grupos
ON alumnos.grupo_id = grupos.id

";

$resultado = mysqli_query(
    $conexion,
    $sql
);
```

El resultado puede recorrerse mediante `mysqli_fetch_assoc()`.

```php
while ($fila = mysqli_fetch_assoc($resultado)) {

    echo $fila["nombre"];
    echo $fila["grupo"];

}
```

Salida aproximada:

```text
Ana ASIR
Pedro ASIR
Marta DAW
```

---

## 7.9 Mostrar resultados en HTML

Habitualmente los resultados se presentan dentro de una tabla HTML.

```php
<table>

<tr>

    <th>Alumno</th>

    <th>Grupo</th>

</tr>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<tr>";

    echo "<td>";
    echo $fila["nombre"];
    echo "</td>";

    echo "<td>";
    echo $fila["grupo"];
    echo "</td>";

    echo "</tr>";

}

?>

</table>
```

El navegador mostrará una tabla donde cada fila contiene información procedente de ambas tablas.

---

## 7.10 JOIN y sentencias preparadas

Las consultas JOIN también pueden utilizar sentencias preparadas.

Por ejemplo, si queremos recuperar únicamente los alumnos de un grupo concreto:

```sql
SELECT

    alumnos.nombre,

    grupos.nombre AS grupo

FROM alumnos

JOIN grupos
ON alumnos.grupo_id = grupos.id

WHERE grupos.id = ?;
```

Preparación de la consulta:

```php
$stmt = mysqli_prepare(

    $conexion,

    "
    SELECT

        alumnos.nombre,

        grupos.nombre AS grupo

    FROM alumnos

    JOIN grupos
    ON alumnos.grupo_id = grupos.id

    WHERE grupos.id = ?
    "

);
```

Asociación del parámetro:

```php
mysqli_stmt_bind_param(

    $stmt,

    "i",

    $grupo_id

);
```

Ejecución y recuperación del resultado:

```php
mysqli_stmt_execute(
    $stmt
);

$resultado =
    mysqli_stmt_get_result(
        $stmt
    );
```

---

## 7.11 LEFT JOIN

El tipo de JOIN utilizado hasta ahora recibe realmente el nombre de **INNER JOIN**.

Su comportamiento consiste en devolver únicamente los registros que tienen correspondencia en ambas tablas.

Otro tipo muy utilizado es **LEFT JOIN**.

Su sintaxis es:

```sql
SELECT columnas

FROM tabla1

LEFT JOIN tabla2
ON condicion;
```

LEFT JOIN recupera todos los registros de la tabla situada a la izquierda, incluso aunque no tengan correspondencia en la tabla de la derecha.

---

## 7.12 Ejemplo de LEFT JOIN

Supongamos la siguiente tabla `alumnos`.

| id | nombre | grupo_id |
|----|---------|----------|
| 1 | Ana | 1 |
| 2 | Pedro | 1 |
| 3 | Marta | 2 |
| 4 | Luis | NULL |

Si utilizamos un `JOIN` normal, Luis no aparecerá porque no pertenece a ningún grupo.

Sin embargo:

```sql
SELECT

    alumnos.nombre,

    grupos.nombre AS grupo

FROM alumnos

LEFT JOIN grupos
ON alumnos.grupo_id = grupos.id;
```

mostrará todos los alumnos.

Para aquellos que no tienen grupo asignado, la columna correspondiente aparecerá vacía (`NULL`).

---

## 7.13 Ejemplo completo

```php
<?php

include "conexion.php";

$conexion = conexion();

$sql = "

SELECT

    alumnos.nombre,

    grupos.nombre AS grupo

FROM alumnos

JOIN grupos
ON alumnos.grupo_id = grupos.id

ORDER BY alumnos.nombre

";

$resultado = mysqli_query(
    $conexion,
    $sql
);

?>

<table border="1">

<tr>

    <th>Alumno</th>

    <th>Grupo</th>

</tr>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<tr>";

    echo "<td>";
    echo $fila["nombre"];
    echo "</td>";

    echo "<td>";
    echo $fila["grupo"];
    echo "</td>";

    echo "</tr>";

}

?>

</table>

<?php

mysqli_free_result(
    $resultado
);

mysqli_close(
    $conexion
);

?>
```

---

## 7.14 Resumen

Las consultas JOIN permiten combinar información almacenada en distintas tablas relacionadas.

Los conceptos principales estudiados en este capítulo son:

- Las bases de datos relacionales distribuyen la información en múltiples tablas.
- Las claves foráneas permiten relacionar dichas tablas.
- `JOIN` combina únicamente registros relacionados.
- `LEFT JOIN` conserva todos los registros de la tabla izquierda.
- Los alias (`AS`) facilitan la lectura de los resultados.
- Los JOIN pueden utilizarse tanto con consultas normales como con sentencias preparadas.

Estas consultas constituyen una de las características más potentes de las bases de datos relacionales y son fundamentales en cualquier aplicación de cierta complejidad.