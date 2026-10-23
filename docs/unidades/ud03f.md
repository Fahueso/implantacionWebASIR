# 6. Modificación y Eliminación de Registros

## 6.1 Introducción

Hasta este momento hemos aprendido a:

- Recuperar información mediante consultas `SELECT`.
- Insertar nuevos registros mediante consultas `INSERT`.

Sin embargo, una aplicación real también debe permitir actualizar información existente y eliminar registros que ya no sean necesarios.

Por ejemplo:

- Corregir el teléfono de un cliente.
- Modificar una dirección de correo electrónico.
- Cambiar el estado de una incidencia.
- Eliminar registros duplicados.
- Borrar usuarios dados de baja.

Estas operaciones se realizan mediante las sentencias SQL `UPDATE` y `DELETE`.

Al igual que en los capítulos anteriores, utilizaremos sentencias preparadas para trabajar de forma segura.

---

# 6.2 Modificación de Registros

## 6.2.1 La sentencia UPDATE

La sentencia `UPDATE` permite modificar registros existentes en una tabla.

Su sintaxis general es:

```sql
UPDATE tabla
SET columna = valor
WHERE condicion;
```

Por ejemplo:

```sql
UPDATE clientes
SET telefono = '600999999'
WHERE id = 3;
```

Esta consulta modifica el teléfono del cliente cuyo identificador es 3.

---

## 6.2.2 La importancia de WHERE

La cláusula `WHERE` determina qué registros serán modificados.

Por ejemplo:

```sql
UPDATE clientes
SET telefono = '600999999'
WHERE id = 3;
```

afecta únicamente a un cliente.

Sin embargo:

```sql
UPDATE clientes
SET telefono = '600999999';
```

modificará todos los registros de la tabla.

Este es uno de los errores más habituales cuando se trabaja con bases de datos.

> Antes de ejecutar una sentencia `UPDATE`, conviene revisar siempre que la cláusula `WHERE` sea correcta.

---

## 6.2.3 Actualización mediante sentencias preparadas

Supongamos que queremos modificar el nombre y el teléfono de un cliente.

La consulta SQL será:

```sql
UPDATE clientes
SET nombre = ?, telefono = ?
WHERE id = ?;
```

En PHP:

```php
$stmt = mysqli_prepare(

    $conexion,

    "UPDATE clientes
     SET nombre = ?,
         telefono = ?
     WHERE id = ?"

);

mysqli_stmt_bind_param(

    $stmt,

    "ssi",

    $nombre,
    $telefono,
    $id

);

mysqli_stmt_execute(
    $stmt
);
```

Observa los tipos utilizados:

- `nombre` → `s` (string)
- `telefono` → `s` (string)
- `id` → `i` (integer)

Por ello utilizamos:

```php
"ssi"
```

---

## 6.2.4 Actualización desde formularios

En aplicaciones reales los nuevos valores suelen proceder de formularios HTML.

Por ejemplo:

```html
<form method="POST"
      action="actualizar.php">

    ID<br>

    <input type="number"
           name="id">

    <br><br>

    Nombre<br>

    <input type="text"
           name="nombre">

    <br><br>

    Teléfono<br>

    <input type="text"
           name="telefono">

    <br><br>

    <input type="submit"
           value="Actualizar">

</form>
```

Los datos enviados pueden recuperarse mediante:

```php
$id = $_POST["id"];

$nombre = $_POST["nombre"];

$telefono = $_POST["telefono"];
```

y utilizarse posteriormente en la sentencia preparada.

---

## 6.2.5 Comprobación del resultado

Después de ejecutar una modificación podemos comprobar cuántos registros han sido afectados.

```php
mysqli_stmt_affected_rows(
    $stmt
);
```

Por ejemplo:

```php
if (mysqli_stmt_affected_rows($stmt) > 0) {

    echo "Registro actualizado";

} else {

    echo "No se encontró el registro";

}
```

---

## 6.2.6 Ejemplo completo de actualización

```php
<?php

include "conexion.php";

$conexion = conexion();

$id = $_POST["id"];

$nombre = $_POST["nombre"];

$telefono = $_POST["telefono"];

$stmt = mysqli_prepare(

    $conexion,

    "UPDATE clientes
     SET nombre = ?,
         telefono = ?
     WHERE id = ?"

);

mysqli_stmt_bind_param(

    $stmt,

    "ssi",

    $nombre,
    $telefono,
    $id

);

if (mysqli_stmt_execute($stmt)) {

    echo "Registro actualizado";

} else {

    echo "Error al actualizar";

}

mysqli_stmt_close(
    $stmt
);

mysqli_close(
    $conexion
);

?>
```

---

# 6.3 Eliminación de Registros

## 6.3.1 La sentencia DELETE

La sentencia `DELETE` permite eliminar registros de una tabla.

Su sintaxis general es:

```sql
DELETE FROM tabla
WHERE condicion;
```

Por ejemplo:

```sql
DELETE FROM clientes
WHERE id = 3;
```

Esta consulta elimina el cliente cuyo identificador es 3.

---

## 6.3.2 Riesgos de DELETE

La operación de borrado es irreversible.

Una vez eliminado un registro, la información desaparecerá de la base de datos.

Por este motivo es especialmente importante revisar las condiciones utilizadas.

Por ejemplo:

```sql
DELETE FROM clientes;
```

eliminaría todos los registros de la tabla.

En una aplicación real esto podría provocar una pérdida importante de información.

---

## 6.3.3 Eliminación mediante sentencias preparadas

La forma recomendada de eliminar registros consiste en utilizar sentencias preparadas.

Consulta SQL:

```sql
DELETE FROM clientes
WHERE id = ?;
```

Implementación en PHP:

```php
$stmt = mysqli_prepare(

    $conexion,

    "DELETE FROM clientes
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
```

---

## 6.3.4 Eliminación desde formularios

Podemos solicitar el identificador mediante un formulario sencillo.

```html
<form method="POST"
      action="eliminar.php">

    ID del cliente<br>

    <input type="number"
           name="id">

    <br><br>

    <input type="submit"
           value="Eliminar">

</form>
```

En aplicaciones reales suele incluirse una confirmación para reducir eliminaciones accidentales.

```html
<input type="submit"
       value="Eliminar"
       onclick="
       return confirm(
       '¿Desea eliminar este registro?'
       );
">
```

---

## 6.3.5 Comprobación del resultado

Del mismo modo que ocurre con `UPDATE`, podemos comprobar si se ha eliminado algún registro.

```php
if (mysqli_stmt_affected_rows($stmt) > 0) {

    echo "Registro eliminado";

} else {

    echo "No existe ningún registro con ese identificador";

}
```

---

## 6.3.6 Ejemplo completo de eliminación

```php
<?php

include "conexion.php";

$conexion = conexion();

$id = $_POST["id"];

$stmt = mysqli_prepare(

    $conexion,

    "DELETE FROM clientes
     WHERE id = ?"

);

mysqli_stmt_bind_param(

    $stmt,

    "i",

    $id

);

if (mysqli_stmt_execute($stmt)) {

    echo "Registro eliminado";

} else {

    echo "Error al eliminar";

}

mysqli_stmt_close(
    $stmt
);

mysqli_close(
    $conexion
);

?>
```

---

## 6.4 Resumen

En este capítulo hemos estudiado las dos operaciones restantes del CRUD:

| Operación | Sentencia SQL | Finalidad |
|------------|------------|------------|
| Update | `UPDATE` | Modificar registros existentes |
| Delete | `DELETE` | Eliminar registros |

Al igual que ocurría con las inserciones, todas las operaciones que incorporan datos variables deben realizarse utilizando sentencias preparadas para mejorar la seguridad y mantener una clara separación entre consultas y datos.