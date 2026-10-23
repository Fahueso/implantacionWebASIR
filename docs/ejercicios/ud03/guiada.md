# Práctica Guiada: Gestión de Videojuegos

## Objetivo

Desarrollar una pequeña aplicación web en PHP y MariaDB para gestionar una colección de videojuegos.

Al finalizar la práctica serás capaz de:

- Conectarte a una base de datos desde PHP.
- Recuperar información mediante consultas `SELECT`.
- Insertar registros mediante sentencias preparadas.
- Modificar registros existentes.
- Eliminar registros.
- Utilizar formularios HTML.
- Mostrar información en tablas HTML.
- Aplicar sentencias preparadas para evitar inyecciones SQL.

---

# Parte 1. Creación de la base de datos

## Paso 1. Crear la base de datos

```sql
CREATE DATABASE videojuegos;
```

---

## Paso 2. Seleccionar la base de datos

```sql
USE videojuegos;
```

---

## Paso 3. Crear la tabla

```sql
CREATE TABLE videojuegos (

    id INT AUTO_INCREMENT PRIMARY KEY,

    titulo VARCHAR(100),

    genero VARCHAR(50),

    plataforma VARCHAR(50)

);
```

---

## Paso 4. Insertar datos de prueba

```sql
INSERT INTO videojuegos
(titulo, genero, plataforma)
VALUES
('Minecraft', 'Sandbox', 'PC'),

('FIFA 25', 'Deportes', 'PlayStation'),

('Mario Kart', 'Carreras', 'Nintendo');
```

Comprobar los datos:

```sql
SELECT *
FROM videojuegos;
```

---

# Parte 2. Preparación del proyecto

Crear la siguiente estructura:

```text
videojuegos/

├── conexion.php
├── index.php
├── listar.php
├── nuevo.php
├── insertar.php
├── editar.php
├── actualizar.php
└── eliminar.php
```

---

# Parte 3. Conexión a la base de datos

## Paso 5. Crear el archivo conexion.php

```php
<?php

function conexion()
{
    $conexion = mysqli_connect(
        "localhost",
        "usuario",
        "password",
        "videojuegos"
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

---

# Parte 4. Página principal

## Paso 6. Crear index.php

```php
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>Gestión de Videojuegos</title>
</head>

<body>

<h1>Gestión de videojuegos</h1>

<ul>

    <li>
        <a href="listar.php">
            Listar videojuegos
        </a>
    </li>

    <li>
        <a href="nuevo.php">
            Añadir videojuego
        </a>
    </li>

</ul>

</body>
</html>
```

Abre la página en el navegador y comprueba que aparecen los enlaces.

---

# Parte 5. Mostrar videojuegos

## Paso 7. Crear listar.php

```php
<?php

include "conexion.php";

$conexion = conexion();

$sql = "
SELECT *
FROM videojuegos
ORDER BY titulo
";

$resultado = mysqli_query(
    $conexion,
    $sql
);

?>

<!DOCTYPE html>

<html>

<head>
    <meta charset="UTF-8">
    <title>Listado</title>
</head>

<body>

<h1>Listado de videojuegos</h1>

<table border="1">

<tr>
    <th>ID</th>
    <th>Título</th>
    <th>Género</th>
    <th>Plataforma</th>
</tr>

<?php

while (
    $fila = mysqli_fetch_assoc($resultado)
) {

    echo "<tr>";

    echo "<td>";
    echo $fila["id"];
    echo "</td>";

    echo "<td>";
    echo $fila["titulo"];
    echo "</td>";

    echo "<td>";
    echo $fila["genero"];
    echo "</td>";

    echo "<td>";
    echo $fila["plataforma"];
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

Comprueba que aparecen los tres videojuegos insertados anteriormente.

---

# Parte 6. Formulario de alta

## Paso 8. Crear nuevo.php

```php
<!DOCTYPE html>

<html>

<head>
    <meta charset="UTF-8">
    <title>Nuevo videojuego</title>
</head>

<body>

<h1>Nuevo videojuego</h1>

<form method="POST"
      action="insertar.php">

    Título<br>

    <input type="text"
           name="titulo">

    <br><br>

    Género<br>

    <input type="text"
           name="genero">

    <br><br>

    Plataforma<br>

    <input type="text"
           name="plataforma">

    <br><br>

    <input type="submit"
           value="Guardar">

</form>

</body>

</html>
```

Comprueba que el formulario aparece correctamente.

---

# Parte 7. Insertar videojuegos

## Paso 9. Crear insertar.php

```php
<?php

include "conexion.php";

$conexion = conexion();

$titulo =
    $_POST["titulo"];

$genero =
    $_POST["genero"];

$plataforma =
    $_POST["plataforma"];

$stmt = mysqli_prepare(

    $conexion,

    "INSERT INTO videojuegos
    (titulo, genero, plataforma)
    VALUES (?, ?, ?)"

);

mysqli_stmt_bind_param(

    $stmt,

    "sss",

    $titulo,
    $genero,
    $plataforma

);

if (
    mysqli_stmt_execute($stmt)
) {

    echo "Videojuego añadido correctamente";

}
else {

    echo "Error al insertar";

}

mysqli_stmt_close(
    $stmt
);

mysqli_close(
    $conexion
);

?>
```

Añade varios videojuegos y comprueba que se almacenan correctamente.

---

# Parte 8. Añadir acciones a la tabla

## Paso 10. Modificar listar.php

Añade dos columnas nuevas:

```html
<th>Editar</th>
<th>Eliminar</th>
```

Dentro del bucle añade:

```php
echo "<td>";

echo "<a href='editar.php?id=";
echo $fila["id"];
echo "'>";

echo "Editar";

echo "</a>";

echo "</td>";
```

y:

```php
echo "<td>";

echo "<a href='eliminar.php?id=";
echo $fila["id"];
echo "'>";

echo "Eliminar";

echo "</a>";

echo "</td>";
```

Comprueba que aparecen los enlaces.

---

# Parte 9. Editar videojuegos

## Paso 11. Crear editar.php

Recuperar el videojuego seleccionado:

```php
$id = $_GET["id"];
```

Consulta:

```sql
SELECT *
FROM videojuegos
WHERE id = ?
```

Mostrar un formulario con los valores recuperados para que puedan modificarse.

**Reto guiado:**
utiliza como modelo el formulario de `nuevo.php`.

---

# Parte 10. Actualizar videojuegos

## Paso 12. Crear actualizar.php

Utilizar la consulta:

```sql
UPDATE videojuegos
SET titulo = ?,
    genero = ?,
    plataforma = ?
WHERE id = ?
```

Recuperar:

```php
$id
$titulo
$genero
$plataforma
```

desde `$_POST`.

Utilizar:

```php
mysqli_stmt_bind_param(
    $stmt,
    "sssi",
    ...
);
```

Mostrar un mensaje indicando si la actualizacirte 11. Eliminar videojuegos

## Paso 13. Crear eliminar.php

Recuperar:

```php
$id = $_GET["id"];
```

Preparar la consulta:

```sql
DELETE FROM videojuegos
WHERE id = ?
```

Utilizar una sentencia preparada para eliminar el registro.

Mostrar:

```text
Videojuego eliminado correctamente
```

si la operación tiene éxito.

---

# Parte 12. Mejoras opcionales

## Nivel 1

Añadir un enlace:

```text
Volver al listado
```

en todas las páginas.

---

## Nivel 2

Añadir validaciones:

- El título es obligatorio.
- El género es obligatorio.
- La plataforma es obligatoria.

---

## Nivel 3

Añadir un buscador por título.

Formulario:

```html
<input type="text" name="titulo">
```

Consulta:

```sql
SELECT *
FROM videojuegos
WHERE titulo = ?
```

usando una sentencia preparada.

---

## Nivel 4

Permitir ordenar los videojuegos por:

- título
- género
- plataforma

---

# Resultado final

La aplicación deberá permitir:

* Listar videojuegos
* Añadir videojuegos
* Modificar videojuegos
* Eliminar videojuegos
* Utilizar sentencias preparadas
* Trabajar con formularios HTML
* Mostrar resultados obtenidos desde MariaDB

Cuando finalices habrás implementado un CRUD completo aplicando todos los conceptos estudiados en la unidad.