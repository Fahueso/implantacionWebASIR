# 5. Inserción de Registros mediante Sentencias Preparadas

## 5.1 Introducción

Hasta ahora hemos aprendido a **leer** datos con `SELECT`. Pero una aplicación web también debe **almacenar** los datos que generan los usuarios:

- Dar de alta un cliente.
- Registrar un nuevo usuario.
- Crear una incidencia.
- Añadir un producto a un catálogo.
- Registrar una reserva.

En SQL, esta operación se realiza con `INSERT INTO`. Como los datos proceden de formularios, **usaremos siempre sentencias preparadas**.

---

## 5.2 La sentencia INSERT

```sql
INSERT INTO nombre_tabla
    (columna1, columna2)
VALUES
    (valor1, valor2);
```

Por ejemplo:

```sql
INSERT INTO clientes
    (nombre, telefono)
VALUES
    ('Ana', '600111111');
```

> **Indica siempre las columnas.** `INSERT INTO clientes VALUES (...)` sin nombrarlas obliga a conocer el orden exacto de todas las columnas y se rompe si la tabla cambia.

Si la tabla usa una clave primaria autoincremental, **no se indica el `id`**: MariaDB lo genera:

```sql
CREATE TABLE clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    telefono VARCHAR(20)
);
```

---

## 5.3 Inserciones y sentencias preparadas

Construir la consulta concatenando variables es **vulnerable** además de incómodo (comillas, caracteres especiales...):

```php
// ❌ NUNCA: mezcla datos y SQL
$sql = "INSERT INTO clientes (nombre, telefono) VALUES ('$nombre', '$telefono')";
```

Si un cliente se llama `O'Brien`, la comilla rompe la sentencia; y si el nombre es `x'); DELETE FROM clientes; --`, el problema es mucho peor.

La forma correcta usa marcadores:

```sql
INSERT INTO clientes (nombre, telefono) VALUES (?, ?);
```

---

## 5.4 Flujo de una inserción

```text
Comprobar que llegan datos (POST)
          ↓
Limpiar y validar los datos
          ↓
Preparar la consulta
          ↓
Asociar parámetros
          ↓
Ejecutar (dentro de try/catch)
          ↓
Informar del resultado y redirigir
          ↓
Cerrar
```

---

## 5.5 Inserción de un registro

```php
$nombre   = "Carlos";
$telefono = "600444444";

$stmt = mysqli_prepare(
    $conexion,
    "INSERT INTO clientes (nombre, telefono) VALUES (?, ?)"
);

mysqli_stmt_bind_param($stmt, "ss", $nombre, $telefono);

mysqli_stmt_execute($stmt);
```

Ambos campos son texto, por eso `"ss"`.

---

## 5.6 Comprobación del resultado

Con el modo de excepciones que activamos en el capítulo 2, `mysqli_stmt_execute()` **no devuelve `false` cuando falla: lanza una excepción** `mysqli_sql_exception`. Por eso el clásico:

```php
if (mysqli_stmt_execute($stmt)) { ... } else { echo "Error"; }   // el else NUNCA se ejecuta
```

no sirve. Lo correcto es rodear la operación con `try / catch`:

```php
try {

    mysqli_stmt_execute($stmt);

    echo "Registro insertado correctamente";

} catch (mysqli_sql_exception $e) {

    error_log("Error al insertar cliente: " . $e->getMessage());

    echo "No se pudo guardar el cliente.";

}
```

Casos típicos en los que la ejecución falla: texto más largo que la columna, valor duplicado en una columna `UNIQUE`, columna `NOT NULL` sin valor, violación de una clave foránea...

---

## 5.7 Validación de los datos

Las sentencias preparadas evitan la inyección SQL, pero **no comprueban que los datos sean correctos**. Un usuario puede enviar un formulario vacío, un nombre de 5000 caracteres o un teléfono con letras.

La validación se hace **en el servidor, antes** de insertar. (La validación en HTML o JavaScript mejora la experiencia, pero se puede saltar fácilmente: nunca es suficiente por sí sola.)

Pasos habituales:

1. **Limpiar** con `trim()` para quitar espacios sobrantes.
2. **Comprobar** que lo obligatorio no está vacío, la longitud y el formato.
3. **Acumular los errores** en un array para mostrarlos todos a la vez.

```php
$nombre   = trim($_POST["nombre"] ?? "");
$telefono = trim($_POST["telefono"] ?? "");

$errores = [];

if ($nombre === "") {

    $errores[] = "El nombre es obligatorio.";

} elseif (mb_strlen($nombre) > 100) {

    $errores[] = "El nombre no puede superar los 100 caracteres.";

}

if (!ctype_digit($telefono) || strlen($telefono) !== 9) {

    $errores[] = "El teléfono debe tener 9 dígitos.";

}
```

> **Cuidado con `empty()`.** `empty("0")` es `true`, de modo que un nombre o valor `"0"` se consideraría vacío. Para texto es más fiable `trim($valor) === ""`.
> Usa `mb_strlen()` en lugar de `strlen()` para contar caracteres (las tildes y la ñ ocupan 2 bytes en UTF-8).

---

## 5.8 Un único fichero para formulario y proceso

Es muy habitual que `insertar.php` haga dos cosas:

- Si se accede normalmente (**GET**): muestra el formulario vacío.
- Si se envía el formulario (**POST**): valida, inserta y, si todo va bien, redirige.

```php
<?php

session_start();

require_once "conexion.php";

$nombre   = "";
$telefono = "";
$errores  = [];

if ($_SERVER["REQUEST_METHOD"] === "POST") {

    $nombre   = trim($_POST["nombre"] ?? "");
    $telefono = trim($_POST["telefono"] ?? "");

    // --- Validación ---
    if ($nombre === "") {
        $errores[] = "El nombre es obligatorio.";
    } elseif (mb_strlen($nombre) > 100) {
        $errores[] = "El nombre no puede superar los 100 caracteres.";
    }

    if (!ctype_digit($telefono) || strlen($telefono) !== 9) {
        $errores[] = "El teléfono debe tener 9 dígitos.";
    }

    // --- Inserción (solo si no hay errores) ---
    if (count($errores) === 0) {

        $conexion = conexion();

        $stmt = mysqli_prepare(
            $conexion,
            "INSERT INTO clientes (nombre, telefono) VALUES (?, ?)"
        );

        mysqli_stmt_bind_param($stmt, "ss", $nombre, $telefono);

        try {

            mysqli_stmt_execute($stmt);

            $id = mysqli_stmt_insert_id($stmt);

            mysqli_stmt_close($stmt);
            mysqli_close($conexion);

            $_SESSION["mensaje"] = "Cliente creado correctamente (ID " . $id . ").";

            header("Location: clientes.php");
            exit;

        } catch (mysqli_sql_exception $e) {

            error_log("Error al insertar cliente: " . $e->getMessage());

            $errores[] = "No se pudo guardar el cliente. Inténtalo de nuevo.";

            mysqli_stmt_close($stmt);
            mysqli_close($conexion);

        }

    }

}

?>
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Nuevo cliente</title>
</head>

<body>

<h1>Nuevo cliente</h1>

<?php if (count($errores) > 0) { ?>

    <ul style="color: red;">
        <?php foreach ($errores as $error) { ?>
            <li><?= h($error) ?></li>
        <?php } ?>
    </ul>

<?php } ?>

<form method="POST" action="insertar.php">

    <p>
        Nombre<br>
        <input type="text" name="nombre" value="<?= h($nombre) ?>">
    </p>

    <p>
        Teléfono<br>
        <input type="text" name="telefono" value="<?= h($telefono) ?>">
    </p>

    <input type="submit" value="Guardar">

</form>

<p><a href="clientes.php">Volver al listado</a></p>

</body>
</html>
```

Observa los detalles:

- `$_SERVER["REQUEST_METHOD"]` indica si la petición es GET o POST.
- Los valores se **devuelven al formulario** (`value="<?= h($nombre) ?>"`) para que el usuario no tenga que reescribirlos si hay errores. Se escapan con `h()`.
- Si la inserción funciona, se **redirige** (siguiente sección).

---

## 5.9 El patrón Post / Redirect / Get

Si después de insertar mostráramos directamente un mensaje de éxito, la página resultante sería la respuesta a una petición POST. Si el usuario pulsa **F5 (recargar)**, el navegador le preguntará si desea reenviar el formulario, y al aceptar **se insertaría el cliente por segunda vez**.

La solución es el patrón **PRG** (*Post / Redirect / Get*):

```text
1. POST   el formulario envía los datos a insertar.php
2. (el servidor inserta)
3. REDIRECT   insertar.php responde con header("Location: clientes.php")
4. GET    el navegador pide clientes.php
```

Ahora el usuario ve el listado como respuesta a una petición GET y recargar la página no repite la inserción.

```php
header("Location: clientes.php");
exit;
```

Dos reglas:

- `header()` debe ejecutarse **antes de enviar cualquier salida** (ni HTML, ni espacios, ni `echo`).
- Siempre `exit;` justo después, para que el script no siga ejecutándose.

### Mostrar un mensaje tras la redirección

Como la redirección crea una petición nueva, guardamos el mensaje en la **sesión** (un «mensaje flash»: se muestra una vez y se borra). Al principio de `clientes.php`:

```php
<?php

session_start();

require_once "conexion.php";

$mensaje = $_SESSION["mensaje"] ?? "";

unset($_SESSION["mensaje"]);   // para que no vuelva a aparecer

// ... resto del código del listado ...
```

Y en el HTML del listado:

```php
<?php if ($mensaje !== "") { ?>
    <p style="color: green;"><?= h($mensaje) ?></p>
<?php } ?>
```

---

## 5.10 Recuperación del identificador generado

Tras insertar en una tabla con clave autoincremental, podemos saber qué `id` ha recibido el nuevo registro:

```php
$id = mysqli_stmt_insert_id($stmt);
```

(También existe `mysqli_insert_id($conexion)`, equivalente en este caso.)

Es útil cuando hay que seguir trabajando con el registro recién creado, por ejemplo para insertar filas relacionadas en otra tabla o redirigir a su ficha.

---

## 5.11 Número de filas afectadas

```php
$filas = mysqli_stmt_affected_rows($stmt);
```

Tras un `INSERT` correcto devuelve `1`, porque se ha añadido un registro. En los capítulos siguientes veremos que con `UPDATE` y `DELETE` este dato es más interesante.

---

## Resumen

- `INSERT INTO tabla (columnas) VALUES (?, ?)` con sentencias preparadas.
- Los datos de formularios se obtienen con `$_POST["campo"] ?? ""` y se limpian con `trim()`.
- La validación se hace **en el servidor**: obligatorios, longitud, formato. Se acumulan los errores en un array.
- Con excepciones, el fallo de `mysqli_stmt_execute()` se captura con **`try / catch`**.
- Un único fichero puede mostrar el formulario (GET) y procesarlo (POST).
- Tras una operación que modifica datos se aplica **Post / Redirect / Get** para evitar duplicados al recargar.
- `mysqli_stmt_insert_id()` devuelve el `id` autogenerado.

## Actividades

1. Crea `insertar.php` completo. Comprueba que: (a) con el formulario vacío muestra errores, (b) un nombre con comillas (`O'Brien`) se guarda bien, (c) tras guardar y recargar con F5 no se duplica el cliente.
2. Quita la redirección y muestra directamente un mensaje de éxito. Inserta un cliente y recarga con F5. ¿Qué ocurre? ¿Por qué?
3. Añade un campo `email` a la tabla (`ALTER TABLE`) y al formulario. Valídalo con `filter_var($email, FILTER_VALIDATE_EMAIL)`.
4. Añade a la tabla una restricción `UNIQUE` en `telefono` e intenta insertar dos clientes con el mismo teléfono. Captura el error y muestra «Ya existe un cliente con ese teléfono» (pista: el código de error es `1062`; se obtiene con `$e->getCode()`).
