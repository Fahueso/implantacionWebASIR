# 5. Inserción de Registros mediante Sentencias Preparadas

## 5.1 Introducción

Hasta este momento hemos aprendido a recuperar información de una base de datos mediante consultas `SELECT`.

Sin embargo, una aplicación web no debe limitarse únicamente a mostrar información. En la mayoría de los casos también necesitaremos almacenar nuevos datos generados por los usuarios.

Algunos ejemplos habituales son:

- Dar de alta un cliente.
- Registrar un nuevo usuario.
- Crear una incidencia.
- Añadir un producto a un catálogo.
- Registrar una reserva.

Todas estas operaciones tienen algo en común: implican insertar nuevos registros en una tabla.

En SQL esta operación se realiza mediante la sentencia `INSERT INTO`.

A partir de este capítulo utilizaremos siempre sentencias preparadas cuando la consulta incluya datos variables.

---

## 5.2 La sentencia INSERT

La sintaxis básica de una inserción es:

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

Esta instrucción crea un nuevo registro en la tabla `clientes`.

Si la tabla utiliza una clave primaria autoincremental, no será necesario indicar el identificador.

Por ejemplo:

```sql
CREATE TABLE clientes (

    id INT AUTO_INCREMENT PRIMARY KEY,

    nombre VARCHAR(100),

    telefono VARCHAR(20)

);
```

MariaDB generará automáticamente el valor de la columna `id`.

---

## 5.3 Inserciones y sentencias preparadas

Aunque es posible construir una consulta concatenando variables dentro de una cadena SQL, actualmente no es una práctica recomendable.

Por ejemplo:

```php
$sql = "
INSERT INTO clientes
(nombre, telefono)
VALUES
('$nombre', '$telefono')
";
```

Este enfoque mezcla datos y SQL, dificulta el mantenimiento y puede introducir vulnerabilidades cuando los datos proceden de formularios.

La forma recomendada consiste en utilizar marcadores de posición:

```sql
INSERT INTO clientes
(nombre, telefono)
VALUES
(?, ?);
```

Posteriormente asociaremos los valores desde PHP.

---

## 5.4 Flujo de una inserción

Las inserciones mediante sentencias preparadas siguen siempre el mismo proceso:

1. Preparar la consulta.
2. Asociar los parámetros.
3. Ejecutar la sentencia.
4. Comprobar el resultado.
5. Liberar recursos.

Visualmente:

```text
Preparar
    ↓
Asociar parámetros
    ↓
Ejecutar
    ↓
Comprobar resultado
    ↓
Cerrar
```

---

## 5.5 Inserción de un registro

Supongamos que queremos almacenar un nuevo cliente.

```php
$nombre = "Carlos";

$telefono = "600444444";
```

La inserción se realizaría así:

```php
$stmt = mysqli_prepare(

    $conexion,

    "INSERT INTO clientes
     (nombre, telefono)
     VALUES (?, ?)"

);

mysqli_stmt_bind_param(

    $stmt,

    "ss",

    $nombre,
    $telefono

);

mysqli_stmt_execute(
    $stmt
);
```

Observa que ambos campos son cadenas de texto y por eso utilizamos:

```php
"ss"
```

---

## 5.6 Comprobación del resultado

La función:

```php
mysqli_stmt_execute()
```

devuelve un valor lógico indicando si la operación se ha realizado correctamente.

Por ejemplo:

```php
if (mysqli_stmt_execute($stmt)) {

    echo "Registro insertado correctamente";

} else {

    echo "Error al insertar el registro";

}
```

Siempre que una operación modifique la base de datos conviene comprobar su resultado.

---

## 5.7 Inserción desde formularios

En aplicaciones reales los datos suelen proceder de formularios HTML.

Por ejemplo:

```html
<form method="POST"
      action="insertar.php">

    Nombre<br>

    <input type="text"
           name="nombre">

    <br><br>

    Teléfono<br>

    <input type="text"
           name="telefono">

    <br><br>

    <input type="submit"
           value="Guardar">

</form>
```

Cuando el usuario envía el formulario, PHP recupera los datos mediante `$_POST`.

```php
$nombre = $_POST["nombre"];

$telefono = $_POST["telefono"];
```

Posteriormente podemos almacenarlos utilizando una sentencia preparada.

```php
<?php

include "conexion.php";

$conexion = conexion();

$nombre = $_POST["nombre"];

$telefono = $_POST["telefono"];

$stmt = mysqli_prepare(

    $conexion,

    "INSERT INTO clientes
     (nombre, telefono)
     VALUES (?, ?)"

);

mysqli_stmt_bind_param(

    $stmt,

    "ss",

    $nombre,
    $telefono

);

if (mysqli_stmt_execute($stmt)) {

    echo "<p>Cliente insertado correctamente.</p>";

} else {

    echo "<p>Error al insertar el cliente.</p>";

}

mysqli_stmt_close($stmt);

mysqli_close($conexion);

?>
```

Este programa constituye la base de la mayoría de formularios de alta utilizados en aplicaciones web.

---

## 5.8 Recuperación del identificador generado

Muchas tablas utilizan claves primarias autoincrementales.

Tras realizar una inserción podemos recuperar el identificador generado automáticamente mediante:

```php
mysqli_insert_id(
    $conexion
);
```

Por ejemplo:

```php
$id = mysqli_insert_id(
    $conexion
);

echo $id;
```

Salida:

```text
15
```

Este valor resulta útil cuando la aplicación necesita seguir trabajando con el registro recién creado.

---

## 5.9 Número de filas afectadas

También es posible comprobar cuántos registros han sido afectados por la operación.

Para ello utilizamos:

```php
mysqli_stmt_affected_rows(
    $stmt
);
```

Por ejemplo:

```php
echo mysqli_stmt_affected_rows(
    $stmt
);
```

Una inserción correcta devolverá normalmente:

```text
1
```

porque se habrá añadido un único registro.

---

## 5.10 Validación de los datos

Las sentencias preparadas mejoran la seguridad, pero no verifican que la información recibida sea correcta.

Por ejemplo, un usuario podría enviar un formulario vacío.

```php
$nombre = "";

$telefono = "";
```

Por este motivo es recomendable validar los datos antes de realizar la inserción.

```php
if (empty($nombre)) {

    die("El nombre es obligatorio");

}
```

También podemos comprobar la longitud de un campo:

```php
if (strlen($telefono) < 9) {

    die("El teléfono no es válido");

}
```

Estas validaciones permiten detectar errores antes de almacenarlos en la base de datos.

---

## 5.11 Ejemplo completo

```php
<?php

include "conexion.php";

$conexion = conexion();

$nombre = $_POST["nombre"];

$telefono = $_POST["telefono"];

if (empty($nombre)) {

    die("El nombre es obligatorio");

}

$stmt = mysqli_prepare(

    $conexion,

    "INSERT INTO clientes
     (nombre, telefono)
     VALUES (?, ?)"

);

mysqli_stmt_bind_param(

    $stmt,

    "ss",

    $nombre,
    $telefono

);

if (mysqli_stmt_execute($stmt)) {

    $id = mysqli_insert_id(
        $conexion
    );

    echo "<p>";
    echo "Cliente insertado correctamente.";
    echo "</p>";

    echo "<p>";
    echo "ID generado: " . $id;
    echo "</p>";

} else {

    echo "<p>";
    echo "Error al insertar el registro.";
    echo "</p>";

}

mysqli_stmt_close($stmt);

mysqli_close($conexion);

?>
```

Este ejemplo refleja una situación típica en aplicaciones web:

1. El usuario rellena un formulario.
2. PHP recibe la información.
3. Se validan los datos.
4. Se prepara la consulta.
5. Se ejecuta la inserción.
6. Se informa del resultado.
7. Se liberan los recursos utilizados.
