# 6. Modificación y Eliminación de Registros

## 6.1 Introducción

Ya sabemos **leer** (`SELECT`) e **insertar** (`INSERT`). Una aplicación real también debe permitir **actualizar** datos existentes y **eliminar** los que ya no son necesarios:

- Corregir el teléfono de un cliente.
- Cambiar el estado de una incidencia.
- Eliminar registros duplicados.
- Borrar usuarios dados de baja.

Se hace con las sentencias `UPDATE` y `DELETE`, siempre con sentencias preparadas.

---

# 6.2 Modificación de registros

## 6.2.1 La sentencia UPDATE

```sql
UPDATE tabla
SET columna = valor
WHERE condicion;
```

```sql
UPDATE clientes
SET telefono = '600999999'
WHERE id = 3;
```

Modifica el teléfono del cliente cuyo identificador es 3.

## 6.2.2 La importancia de WHERE

`WHERE` determina **qué registros** se modifican. Esta consulta afecta a un solo cliente:

```sql
UPDATE clientes SET telefono = '600999999' WHERE id = 3;
```

pero esta, sin `WHERE`, **modifica todos los registros de la tabla**:

```sql
UPDATE clientes SET telefono = '600999999';
```

Es uno de los errores más comunes (y más dolorosos) al trabajar con bases de datos.

> **Antes de ejecutar un `UPDATE` o un `DELETE`, comprueba siempre la cláusula `WHERE`.** Un buen hábito al practicar es ejecutar primero un `SELECT` con la misma condición para ver qué filas se verán afectadas.

## 6.2.3 UPDATE con sentencias preparadas

```sql
UPDATE clientes
SET nombre = ?, telefono = ?
WHERE id = ?;
```

```php
$stmt = mysqli_prepare(
    $conexion,
    "UPDATE clientes
     SET nombre = ?,
         telefono = ?
     WHERE id = ?"
);

mysqli_stmt_bind_param($stmt, "ssi", $nombre, $telefono, $id);

mysqli_stmt_execute($stmt);
```

Los tipos: `nombre` → `s`, `telefono` → `s`, `id` → `i`. De ahí `"ssi"`. Recuerda que el orden de las letras sigue el orden de los `?`.

## 6.2.4 Validar el identificador

El `id` llega desde la URL o desde un formulario, es decir, **del usuario**. Si es texto no numérico y lo asociamos con `"i"`, PHP lo convierte silenciosamente a `0` y la consulta se ejecuta igualmente sin sentido. Hay que validarlo antes:

```php
$id = filter_var($_GET["id"] ?? "", FILTER_VALIDATE_INT);

if ($id === false || $id < 1) {

    // id no válido: volver al listado
    header("Location: clientes.php");
    exit;

}
```

`filter_var(..., FILTER_VALIDATE_INT)` devuelve el entero si el valor es un entero válido, y `false` en caso contrario.

## 6.2.5 Flujo habitual de edición

Editar un registro suele hacerse en dos pasos dentro de `actualizar.php`:

1. **GET** `actualizar.php?id=3` → se consulta el cliente y se muestra el formulario **con sus datos actuales**.
2. **POST** → se validan los datos, se ejecuta el `UPDATE` y se redirige (patrón PRG del capítulo anterior).

Primero añadimos enlaces en el listado (`clientes.php`):

```php
<td>
    <a href="actualizar.php?id=<?= h($fila["id"]) ?>">Editar</a>
</td>
```

Y el fichero completo:

```php
<?php

session_start();

require_once "conexion.php";

$conexion = conexion();

// El id viaja por la URL en el GET inicial y como campo oculto en el POST
$id = filter_var($_GET["id"] ?? $_POST["id"] ?? "", FILTER_VALIDATE_INT);

if ($id === false || $id < 1) {

    header("Location: clientes.php");
    exit;

}

$errores = [];

if ($_SERVER["REQUEST_METHOD"] === "POST") {

    // --- Procesar el formulario ---
    $nombre   = trim($_POST["nombre"] ?? "");
    $telefono = trim($_POST["telefono"] ?? "");

    if ($nombre === "") {
        $errores[] = "El nombre es obligatorio.";
    }

    if (!ctype_digit($telefono) || strlen($telefono) !== 9) {
        $errores[] = "El teléfono debe tener 9 dígitos.";
    }

    if (count($errores) === 0) {

        $stmt = mysqli_prepare(
            $conexion,
            "UPDATE clientes SET nombre = ?, telefono = ? WHERE id = ?"
        );

        mysqli_stmt_bind_param($stmt, "ssi", $nombre, $telefono, $id);

        try {

            mysqli_stmt_execute($stmt);

            $filas = mysqli_stmt_affected_rows($stmt);

            mysqli_stmt_close($stmt);
            mysqli_close($conexion);

            $_SESSION["mensaje"] = ($filas > 0)
                ? "Cliente actualizado."
                : "No se han producido cambios.";

            header("Location: clientes.php");
            exit;

        } catch (mysqli_sql_exception $e) {

            error_log("Error al actualizar cliente: " . $e->getMessage());

            $errores[] = "No se pudo actualizar el cliente.";

        }

    }

} else {

    // --- Primera carga: leer los datos actuales ---
    $stmt = mysqli_prepare(
        $conexion,
        "SELECT nombre, telefono FROM clientes WHERE id = ?"
    );

    mysqli_stmt_bind_param($stmt, "i", $id);
    mysqli_stmt_execute($stmt);

    $resultado = mysqli_stmt_get_result($stmt);
    $cliente   = mysqli_fetch_assoc($resultado);

    mysqli_free_result($resultado);
    mysqli_stmt_close($stmt);

    if (!$cliente) {

        mysqli_close($conexion);

        $_SESSION["mensaje"] = "El cliente solicitado no existe.";

        header("Location: clientes.php");
        exit;

    }

    $nombre   = $cliente["nombre"];
    $telefono = $cliente["telefono"];

}

mysqli_close($conexion);

?>
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Editar cliente</title>
</head>

<body>

<h1>Editar cliente</h1>

<?php foreach ($errores as $error) { ?>
    <p style="color: red;"><?= h($error) ?></p>
<?php } ?>

<form method="POST" action="actualizar.php">

    <input type="hidden" name="id" value="<?= h($id) ?>">

    <p>
        Nombre<br>
        <input type="text" name="nombre" value="<?= h($nombre) ?>">
    </p>

    <p>
        Teléfono<br>
        <input type="text" name="telefono" value="<?= h($telefono) ?>">
    </p>

    <input type="submit" value="Actualizar">

</form>

<p><a href="clientes.php">Cancelar</a></p>

</body>
</html>
```

> Ten en cuenta que el `id` va en un campo `hidden`, pero **un campo oculto lo puede modificar cualquiera** desde las herramientas del navegador. Por eso el servidor lo vuelve a validar.

## 6.2.6 Comprobación del resultado: `affected_rows`

```php
mysqli_stmt_affected_rows($stmt);
```

devuelve cuántas filas **han cambiado**. Hay un detalle importante en `UPDATE`:

- Si el registro existe y los nuevos valores son **distintos**, devuelve `1`.
- Si el registro existe pero los valores son **idénticos a los que ya tenía**, devuelve `0` (MariaDB no cambia nada).
- Si el registro no existe, también devuelve `0`.

Por eso **`0` no significa necesariamente «no existe»**. Mostrar «No se encontró el registro» cuando devuelve 0 puede ser falso: el usuario pudo guardar sin cambiar nada. En el ejemplo anterior comprobamos la existencia con un `SELECT` en la primera carga y, cuando devuelve 0, mostramos «No se han producido cambios».

---

# 6.3 Eliminación de registros

## 6.3.1 La sentencia DELETE

```sql
DELETE FROM tabla
WHERE condicion;
```

```sql
DELETE FROM clientes
WHERE id = 3;
```

## 6.3.2 Riesgos de DELETE

El borrado es **irreversible**: el dato desaparece. Sin `WHERE`:

```sql
DELETE FROM clientes;
```

se eliminan **todos** los registros de la tabla.

> En aplicaciones reales es frecuente no borrar físicamente, sino marcar el registro como inactivo (por ejemplo, con una columna `activo` o `fecha_baja`) para poder recuperarlo. Se conoce como **borrado lógico** (*soft delete*).

## 6.3.3 DELETE con sentencias preparadas

```sql
DELETE FROM clientes WHERE id = ?;
```

```php
$stmt = mysqli_prepare(
    $conexion,
    "DELETE FROM clientes WHERE id = ?"
);

mysqli_stmt_bind_param($stmt, "i", $id);

mysqli_stmt_execute($stmt);
```

## 6.3.4 Eliminación desde el listado: usar POST, no GET

Es tentador poner en el listado un enlace `eliminar.php?id=3`. **No lo hagas**: un enlace es una petición GET y las peticiones GET las pueden lanzar cosas que no controlas: un navegador que precarga páginas, un buscador que rastrea la web, o una imagen maliciosa en otra página (`<img src="…/eliminar.php?id=3">`).

**Regla:** las operaciones que modifican datos se hacen con **POST**. En el listado, cada fila lleva un pequeño formulario con botón:

```php
<td>
    <form method="POST" action="eliminar.php"
          onsubmit="return confirm('¿Desea eliminar este cliente?');">

        <input type="hidden" name="id" value="<?= h($fila["id"]) ?>">
        <input type="submit" value="Eliminar">

    </form>
</td>
```

La función `confirm()` de JavaScript reduce los borrados accidentales. Ojo: es una ayuda para el usuario, no una medida de seguridad; la lógica real siempre está en el servidor.

> Más adelante (en unidades de seguridad y autenticación) veremos que incluso un POST puede ser falsificado desde otra web (ataque **CSRF**) y cómo defenderse con *tokens*. Y, por supuesto, que solo los usuarios autorizados deberían poder borrar.

## 6.3.5 Fichero `eliminar.php`

```php
<?php

session_start();

require_once "conexion.php";

// Solo aceptamos POST
if ($_SERVER["REQUEST_METHOD"] !== "POST") {

    header("Location: clientes.php");
    exit;

}

$id = filter_var($_POST["id"] ?? "", FILTER_VALIDATE_INT);

if ($id === false || $id < 1) {

    $_SESSION["mensaje"] = "Identificador no válido.";

    header("Location: clientes.php");
    exit;

}

$conexion = conexion();

$stmt = mysqli_prepare($conexion, "DELETE FROM clientes WHERE id = ?");

mysqli_stmt_bind_param($stmt, "i", $id);

try {

    mysqli_stmt_execute($stmt);

    if (mysqli_stmt_affected_rows($stmt) > 0) {

        $_SESSION["mensaje"] = "Cliente eliminado.";

    } else {

        $_SESSION["mensaje"] = "No existe ningún cliente con ese identificador.";

    }

} catch (mysqli_sql_exception $e) {

    error_log("Error al eliminar cliente: " . $e->getMessage());

    $_SESSION["mensaje"] = "No se pudo eliminar el cliente.";

}

mysqli_stmt_close($stmt);
mysqli_close($conexion);

header("Location: clientes.php");
exit;
```

En `DELETE`, `affected_rows` sí es fiable: es el número de filas eliminadas, de modo que `0` significa que no existía ese registro.

---

# 6.4 Borrado y claves foráneas

Cuando una tabla está relacionada con otra, MariaDB **impide borrar un registro si hay otros que dependen de él**. Con las tablas `grupos` y `alumnos` del capítulo 7, intentar borrar un grupo que tiene alumnos asignados produce este error:

```text
Cannot delete or update a parent row: a foreign key constraint fails
```

En PHP se trata de una excepción cuyo código es `1451`. Podemos capturarla para dar un mensaje útil:

```php
try {

    mysqli_stmt_execute($stmt);

} catch (mysqli_sql_exception $e) {

    if ($e->getCode() === 1451) {

        $_SESSION["mensaje"] = "No se puede eliminar el grupo porque tiene alumnos asignados.";

    } else {

        error_log($e->getMessage());
        $_SESSION["mensaje"] = "No se pudo eliminar el grupo.";

    }

}
```

El comportamiento se define al crear la clave foránea con `ON DELETE`:

| Opción | Efecto al borrar el registro «padre» (el grupo) |
|--------|-------------------------------------------------|
| `RESTRICT` (por defecto) | Impide el borrado si hay registros relacionados |
| `CASCADE` | Borra también los registros relacionados (¡los alumnos!) |
| `SET NULL` | Deja `NULL` en la clave foránea de los alumnos |

```sql
FOREIGN KEY (grupo_id) REFERENCES grupos(id) ON DELETE SET NULL
```

La elección depende de las reglas de negocio: borrar un grupo ¿debe borrar a sus alumnos? Casi nunca. `CASCADE` es potente pero peligroso: úsalo con mucho cuidado.

---

## 6.5 Resumen

| Operación | Sentencia SQL | Método HTTP | `affected_rows` |
|-----------|---------------|-------------|-----------------|
| Update | `UPDATE` | POST | Filas **modificadas** (0 si no hay cambios *o* no existe) |
| Delete | `DELETE` | POST | Filas **eliminadas** (0 si no existe) |

- Valida el `id` con `filter_var(..., FILTER_VALIDATE_INT)`.
- **Siempre** `WHERE` en `UPDATE` y `DELETE`.
- Editar = leer los datos (GET) + formulario + `UPDATE` (POST) + redirección.
- Borrar siempre con POST, nunca con un enlace GET.
- Un `DELETE` puede fallar por claves foráneas (error `1451`): se captura y se informa.
- Todas las operaciones con datos variables se realizan con sentencias preparadas.

## Actividades

1. Completa la aplicación de clientes: listado con botones **Editar** y **Eliminar**, formulario de alta y de edición.
2. Edita un cliente y pulsa «Actualizar» sin cambiar nada. ¿Qué mensaje sale y por qué?
3. Abre `actualizar.php?id=abc` y `actualizar.php?id=99999`. ¿Cómo se comporta la aplicación? Modifica el campo oculto `id` desde las herramientas del navegador (F12) y envía el formulario: ¿qué ocurre?
4. Quita el `WHERE` de un `UPDATE` en una copia de pruebas de la tabla y observa el resultado (¡nunca en la tabla real!).
5. Con las tablas `grupos` y `alumnos`, intenta borrar el grupo `ASIR` y gestiona el error. Cambia la clave foránea a `ON DELETE SET NULL` y repite la prueba.
