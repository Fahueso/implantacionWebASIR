# 7. Consultas Multitabla y JOIN

## 7.1 Introducción

Hasta ahora hemos trabajado con consultas sobre **una sola tabla**:

```sql
SELECT id, nombre FROM clientes;
```

Sin embargo, la gran potencia de las bases de datos relacionales está en la capacidad de **relacionar tablas entre sí**:

- Un alumno pertenece a un grupo.
- Un pedido pertenece a un cliente.
- Una incidencia pertenece a un usuario.
- Un libro pertenece a una categoría.

Para recuperar información repartida entre varias tablas relacionadas usamos consultas **JOIN**.

---

## 7.2 ¿Por qué dividir la información en varias tablas?

Supongamos que guardamos a los alumnos en una única tabla:

| id | nombre | grupo |
|----|--------|-------|
| 1 | Ana | ASIR |
| 2 | Pedro | ASIR |
| 3 | Marta | DAW |

Funciona, pero tiene inconvenientes:

- Se **repite** información («ASIR» aparece varias veces).
- Es difícil mantener la **consistencia**: «ASIR», «asir» y «Asir » serían tres grupos distintos.
- Aumenta el riesgo de **errores al modificar**: si cambia el nombre del grupo, hay que actualizar todos los registros.

La solución es separar la información en varias tablas y relacionarlas.

---

## 7.3 Claves primarias y claves foráneas

### Tabla `grupos`

```sql
CREATE TABLE grupos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);
```

| id | nombre |
|----|--------|
| 1 | ASIR |
| 2 | DAW |
| 3 | SMR |

### Tabla `alumnos`

```sql
CREATE TABLE alumnos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    grupo_id INT NULL,
    FOREIGN KEY (grupo_id) REFERENCES grupos(id)
);
```

| id | nombre | grupo_id |
|----|--------|----------|
| 1 | Ana | 1 |
| 2 | Pedro | 1 |
| 3 | Marta | 2 |
| 4 | Luis | NULL |

En la tabla `alumnos`:

- `id` es la **clave primaria** del alumno.
- `grupo_id` es una **clave foránea**: contiene el `id` de un registro de la tabla `grupos`.
- Gracias a la clave foránea, MariaDB **no permite** asignar un alumno a un grupo que no existe, ni borrar un grupo que tiene alumnos (ver capítulo 6).
- `Luis` no tiene grupo: `grupo_id` vale `NULL` (la columna lo permite).

> `NULL` significa «sin valor / desconocido». No es ni el número 0 ni una cadena vacía, y para compararlo se usa `IS NULL`, no `= NULL`.

---

## 7.4 El problema que resuelve JOIN

Queremos este resultado:

| Alumno | Grupo |
|--------|-------|
| Ana | ASIR |
| Pedro | ASIR |
| Marta | DAW |

`alumnos` contiene los nombres de los alumnos pero solo el *número* del grupo; `grupos` contiene los nombres de los grupos. Hay que **combinar** ambas tablas.

---

## 7.5 La cláusula JOIN

```sql
SELECT columnas
FROM tabla1
JOIN tabla2
ON condicion;
```

La cláusula `ON` indica **cómo se relacionan** las filas de ambas tablas: normalmente, la clave foránea de una con la clave primaria de la otra.

---

## 7.6 Primer JOIN

```sql
SELECT
    alumnos.nombre,
    grupos.nombre
FROM alumnos
JOIN grupos
    ON alumnos.grupo_id = grupos.id;
```

Como ambas tablas tienen una columna `nombre`, **hay que anteponer el nombre de la tabla** (`alumnos.nombre`) para que MariaDB sepa a cuál nos referimos. El resultado:

| nombre | nombre |
|--------|--------|
| Ana | ASIR |
| Pedro | ASIR |
| Marta | DAW |

Fíjate: **hay dos columnas con el mismo nombre**. Eso causa un problema en PHP, que vemos a continuación.

---

## 7.7 Alias de columnas (`AS`)

Con `mysqli_fetch_assoc()` cada fila es un array asociativo donde la clave es el nombre de la columna. Si dos columnas se llaman igual, **la segunda sobrescribe a la primera** y el array solo tendría una clave `nombre` (con el nombre del *grupo*, perdiéndose el del alumno).

La solución es dar un **alias** a cada columna con `AS`:

```sql
SELECT
    alumnos.nombre AS alumno,
    grupos.nombre  AS grupo
FROM alumnos
JOIN grupos
    ON alumnos.grupo_id = grupos.id;
```

| alumno | grupo |
|--------|-------|
| Ana | ASIR |
| Pedro | ASIR |
| Marta | DAW |

Ahora la fila tendrá las claves `alumno` y `grupo`, sin ambigüedad.

> **Regla práctica:** en una consulta con JOIN, pon alias a todas las columnas que puedan repetirse (`id`, `nombre`...).

---

## 7.8 Alias de tablas

Escribir `alumnos.` y `grupos.` en cada columna resulta largo. También podemos dar **alias a las tablas**:

```sql
SELECT
    a.nombre AS alumno,
    g.nombre AS grupo
FROM alumnos AS a
JOIN grupos AS g
    ON a.grupo_id = g.id;
```

(`AS` es opcional en alias de tabla: `FROM alumnos a JOIN grupos g` es equivalente.)

Es la forma más habitual en la práctica, y a partir de aquí la usaremos.

---

## 7.9 Ejecución de JOIN desde PHP

Una consulta con `JOIN` se ejecuta igual que cualquier otra:

```php
$sql = "
    SELECT
        a.nombre AS alumno,
        g.nombre AS grupo
    FROM alumnos a
    JOIN grupos g
        ON a.grupo_id = g.id
    ORDER BY a.nombre
";

$resultado = mysqli_query($conexion, $sql);

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo h($fila["alumno"]) . " - " . h($fila["grupo"]) . "<br>";

}
```

Salida:

```text
Ana - ASIR
Marta - DAW
Pedro - ASIR
```

---

## 7.10 JOIN y sentencias preparadas

Si la consulta lleva datos variables, usamos sentencias preparadas como siempre. Por ejemplo, los alumnos de un grupo concreto:

```php
$grupo_id = filter_var($_GET["grupo"] ?? "", FILTER_VALIDATE_INT);

$stmt = mysqli_prepare(
    $conexion,
    "SELECT
         a.nombre AS alumno,
         g.nombre AS grupo
     FROM alumnos a
     JOIN grupos g
         ON a.grupo_id = g.id
     WHERE g.id = ?
     ORDER BY a.nombre"
);

mysqli_stmt_bind_param($stmt, "i", $grupo_id);

mysqli_stmt_execute($stmt);

$resultado = mysqli_stmt_get_result($stmt);
```

El `JOIN` no cambia nada del flujo de trabajo: preparar → asociar → ejecutar → recorrer.

---

## 7.11 INNER JOIN y LEFT JOIN

El `JOIN` que hemos usado se llama realmente **INNER JOIN** (`JOIN` es su abreviatura). Devuelve **solo las filas que tienen correspondencia en ambas tablas**.

Con los datos del apartado 7.3, **Luis no aparece** porque no tiene grupo.

**`LEFT JOIN`** devuelve **todas las filas de la tabla de la izquierda** (la que va tras `FROM`), tengan o no correspondencia en la de la derecha:

```sql
SELECT
    a.nombre AS alumno,
    g.nombre AS grupo
FROM alumnos a
LEFT JOIN grupos g
    ON a.grupo_id = g.id
ORDER BY a.nombre;
```

| alumno | grupo |
|--------|-------|
| Ana | ASIR |
| Luis | NULL |
| Marta | DAW |
| Pedro | ASIR |

Para los alumnos sin grupo, las columnas de `grupos` salen como `NULL`.

```text
INNER JOIN                      LEFT JOIN
┌───────┐   ┌───────┐           ┌───────┐   ┌───────┐
│ alum. │ ∩ │ grupos│           │ alum. │ ⟕ │ grupos│
└───────┘   └───────┘           └───────┘   └───────┘
solo filas con pareja          todas las de la izquierda
```

### Tratar el `NULL` en PHP

En PHP, un `NULL` de la base de datos llega como `null`. Si lo mostramos tal cual aparece una celda vacía. Con el operador `??` ponemos un texto alternativo:

```php
echo h($fila["grupo"] ?? "Sin grupo");
```

### Aplicación práctica: encontrar lo que *no* tiene relación

`LEFT JOIN` + `IS NULL` sirve para encontrar registros sin pareja. Por ejemplo, los **grupos sin ningún alumno**:

```sql
SELECT g.nombre
FROM grupos g
LEFT JOIN alumnos a
    ON a.grupo_id = g.id
WHERE a.id IS NULL;
```

---

## 7.12 Elegir un grupo desde un formulario

Al dar de alta un alumno hay que elegir su grupo. En lugar de pedir un número, mostramos un desplegable `<select>` con los grupos existentes (otra aplicación de las consultas que ya conocemos):

```php
$grupos = mysqli_query($conexion, "SELECT id, nombre FROM grupos ORDER BY nombre");
```

```php
<select name="grupo_id">

    <option value="">-- Sin grupo --</option>

    <?php while ($g = mysqli_fetch_assoc($grupos)) { ?>

        <option value="<?= h($g["id"]) ?>"><?= h($g["nombre"]) ?></option>

    <?php } ?>

</select>
```

Al procesar el formulario, el valor `""` (sin grupo) debe guardarse como `NULL` y no como `0` (no existe ningún grupo con `id` 0 y la clave foránea fallaría):

```php
$grupo_id = filter_var($_POST["grupo_id"] ?? "", FILTER_VALIDATE_INT);

if ($grupo_id === false) {

    $grupo_id = null;   // sin grupo

}

$stmt = mysqli_prepare(
    $conexion,
    "INSERT INTO alumnos (nombre, grupo_id) VALUES (?, ?)"
);

mysqli_stmt_bind_param($stmt, "si", $nombre, $grupo_id);

mysqli_stmt_execute($stmt);
```

`mysqli_stmt_bind_param()` admite variables con valor `null`: MariaDB guardará `NULL`. Si el usuario manipula el formulario y envía un grupo inexistente, la clave foránea lo rechazará y saltará una excepción que habrá que capturar con `try / catch`.

---

## 7.13 Ejemplo completo: `alumnos.php`

```php
<?php

require_once "conexion.php";

$conexion = conexion();

$sql = "
    SELECT
        a.id     AS alumno_id,
        a.nombre AS alumno,
        g.nombre AS grupo
    FROM alumnos a
    LEFT JOIN grupos g
        ON a.grupo_id = g.id
    ORDER BY a.nombre
";

$resultado = mysqli_query($conexion, $sql);

?>
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Alumnos</title>
</head>

<body>

<h1>Listado de alumnos</h1>

<table border="1">

    <tr>
        <th>ID</th>
        <th>Alumno</th>
        <th>Grupo</th>
    </tr>

    <?php while ($fila = mysqli_fetch_assoc($resultado)) { ?>

        <tr>
            <td><?= h($fila["alumno_id"]) ?></td>
            <td><?= h($fila["alumno"]) ?></td>
            <td><?= h($fila["grupo"] ?? "Sin grupo") ?></td>
        </tr>

    <?php } ?>

</table>

</body>
</html>
<?php

mysqli_free_result($resultado);
mysqli_close($conexion);

?>
```

---

## 7.14 Resumen

- Las bases de datos relacionales reparten la información en **varias tablas** para evitar repeticiones e inconsistencias.
- La **clave foránea** de una tabla apunta a la **clave primaria** de otra.
- `JOIN ... ON` combina filas relacionadas; `INNER JOIN` devuelve solo las que tienen pareja, `LEFT JOIN` conserva todas las de la izquierda (con `NULL` donde no hay pareja).
- Si dos columnas tienen el mismo nombre, hay que usar **alias** (`AS`) o `fetch_assoc` perderá una de ellas.
- Los **alias de tabla** (`FROM alumnos a`) abrevian las consultas.
- En PHP, `$fila["columna"] ?? "texto"` sirve para mostrar algo cuando el valor es `NULL`.
- Los JOIN se pueden usar con consultas normales y con sentencias preparadas.
- Un valor opcional de clave foránea se guarda como `NULL`, nunca como `0` o `""`.

## Actividades

1. Crea las tablas `grupos` y `alumnos` (o usa el Anexo A) y escribe en el cliente SQL el `JOIN` y el `LEFT JOIN`. Comprueba qué pasa con Luis.
2. Escribe la consulta del primer JOIN **sin alias** y muéstrala en PHP: ¿qué se ve en la columna «nombre»? ¿Por qué?
3. Crea `alumnos.php` con el listado (con `LEFT JOIN` y «Sin grupo») y un filtro por grupo mediante sentencia preparada.
4. Crea el formulario de alta de alumnos con el desplegable de grupos y la opción «Sin grupo».
5. Escribe una consulta que muestre, para cada grupo, cuántos alumnos tiene (pista: `COUNT(a.id)` y `GROUP BY g.id` con un `LEFT JOIN` desde `grupos`).
