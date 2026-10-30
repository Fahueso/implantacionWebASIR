# 3. Recuperación de Datos mediante SELECT

## 3.1 Introducción

Una vez establecida la conexión entre PHP y MariaDB ya podemos recuperar la información almacenada. La lectura es una de las operaciones más frecuentes de cualquier aplicación web:

- Mostrar el catálogo de productos de una tienda online.
- Listar los usuarios registrados de una plataforma.
- Consultar incidencias de un sistema de soporte.
- Visualizar reservas de una aplicación de gestión.

Todas necesitan obtener datos de una base de datos y mostrarlos al usuario. Para ello usaremos consultas SQL de tipo `SELECT`.

> **Antes de empezar:** en esta unidad trabajamos con la base de datos `empresa` del Anexo A, con la tabla `clientes` (`id`, `nombre`, `telefono`).

---

## 3.2 Ejecución de consultas con `mysqli_query()`

Con una conexión activa podemos ejecutar consultas con `mysqli_query()`:

```php
mysqli_query($conexion, $consulta);
```

- `$conexion`: la conexión abierta con MariaDB.
- `$consulta`: una sentencia SQL escrita como cadena de texto.

```php
<?php

require_once "conexion.php";

$conexion = conexion();

$sql = "SELECT id, nombre, telefono FROM clientes";

$resultado = mysqli_query($conexion, $sql);

?>
```

> Si la consulta tiene un error (por ejemplo, un nombre de tabla mal escrito), `mysqli_query()` lanza una excepción `mysqli_sql_exception`, igual que vimos con la conexión. En un servidor real, `display_errors` debe estar en `Off` para que el usuario no vea ese detalle, y los errores quedan registrados en el log.

---

## 3.3 El resultado de una consulta

`mysqli_query()` **no devuelve directamente los registros**: devuelve un objeto *resultado* que representa el conjunto de filas obtenido. Para leer cada fila usamos:

```php
mysqli_fetch_assoc($resultado)
```

Devuelve **una fila** como array asociativo (las claves son los nombres de las columnas). Cada nueva llamada devuelve la siguiente fila. Si la tabla contiene:

| id | nombre | telefono |
|----|--------|----------|
| 1 | Ana | 600111111 |

```php
$fila = mysqli_fetch_assoc($resultado);
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

y podremos acceder a cada dato por el nombre de su columna:

```php
echo $fila["nombre"];   // Ana
```

> **Importante:** cuando ya no quedan filas por leer (o cuando la consulta no devolvió ninguna), `mysqli_fetch_assoc()` devuelve `null`. Intentar usar `$fila["nombre"]` sobre `null` genera un *warning*. Siempre hay que comprobarlo (lo vemos en la sección siguiente).

---

## 3.4 Recuperación de registros

### Un único registro

Si esperamos como mucho una fila, la leemos una sola vez **y comprobamos que existe**:

```php
<?php

require_once "conexion.php";

$conexion = conexion();

$sql = "SELECT id, nombre, telefono FROM clientes WHERE id = 1";

$resultado = mysqli_query($conexion, $sql);

$fila = mysqli_fetch_assoc($resultado);

if ($fila) {

    echo $fila["nombre"];

} else {

    echo "No existe ningún cliente con ese identificador.";

}

mysqli_free_result($resultado);
mysqli_close($conexion);

?>
```

### Múltiples registros

Si la consulta devuelve varias filas usamos un bucle `while`:

```php
while ($fila = mysqli_fetch_assoc($resultado)) {

    echo $fila["nombre"];

}
```

Cada vuelta del bucle lee una fila nueva. Cuando no quedan más, `mysqli_fetch_assoc()` devuelve `null`, la condición del `while` se evalúa como falsa y el bucle termina por sí solo.

### ¿Cuántas filas ha devuelto la consulta?

```php
$total = mysqli_num_rows($resultado);
```

Es útil para mostrar un mensaje del tipo «No hay clientes registrados»:

```php
if (mysqli_num_rows($resultado) === 0) {

    echo "<p>No hay clientes registrados.</p>";

}
```

---

## 3.5 Mostrar resultados en HTML

### Escapar los datos: `htmlspecialchars()`

Antes de mostrar en una página un dato que viene de la base de datos hay que **escaparlo** con `htmlspecialchars()`.

¿Por qué? Imagina que un usuario se registra con el nombre:

```html
<script>alert("Hola")</script>
```

Si hacemos `echo $fila["nombre"]`, el navegador de **todos los usuarios que vean el listado** ejecutará ese código JavaScript. Es un ataque llamado **XSS** (*Cross-Site Scripting*), que permite robar sesiones, redirigir a webs falsas o alterar la página.

`htmlspecialchars()` convierte los caracteres especiales de HTML (`<`, `>`, `&`, `"`, `'`) en entidades inofensivas (`&lt;`, `&gt;`…), de modo que el navegador los **muestra como texto** en vez de interpretarlos como código:

```php
echo htmlspecialchars($fila["nombre"], ENT_QUOTES, "UTF-8");
```

Como lo vamos a escribir muchísimas veces, definimos una función corta. Añádela al final de `conexion.php` (antes del `?>`, si lo usas):

```php
function h($texto)
{

    return htmlspecialchars((string) $texto, ENT_QUOTES, "UTF-8");

}
```

A partir de ahora, **todo dato que viene de la base de datos o del usuario y se imprime en HTML se escapa con `h()`**.

> **Regla de oro:** `htmlspecialchars()` se aplica al **mostrar** (salida), no al guardar. En la base de datos se guarda el dato original, tal cual lo escribió el usuario.

### Lista HTML

```php
<ul>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<li>" . h($fila["nombre"]) . "</li>";

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

Una de las formas más habituales de presentar datos:

```php
<table border="1">

<tr>
    <th>ID</th>
    <th>Nombre</th>
    <th>Teléfono</th>
</tr>

<?php

while ($fila = mysqli_fetch_assoc($resultado)) {

    echo "<tr>";
    echo "<td>" . h($fila["id"]) . "</td>";
    echo "<td>" . h($fila["nombre"]) . "</td>";
    echo "<td>" . h($fila["telefono"]) . "</td>";
    echo "</tr>";

}

?>

</table>
```

---

## 3.6 Liberación de recursos

Al terminar de trabajar con un resultado conviene liberarlo y después cerrar la conexión:

```php
mysqli_free_result($resultado);

mysqli_close($conexion);
```

---

## 3.7 Ejemplo completo: `clientes.php`

```php
<?php

require_once "conexion.php";

$conexion = conexion();

$sql = "SELECT id, nombre, telefono FROM clientes ORDER BY nombre";

$resultado = mysqli_query($conexion, $sql);

?>
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>Clientes</title>
</head>

<body>

<h1>Listado de clientes</h1>

<?php if (mysqli_num_rows($resultado) === 0) { ?>

    <p>No hay clientes registrados.</p>

<?php } else { ?>

    <table border="1">

        <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Teléfono</th>
        </tr>

        <?php while ($fila = mysqli_fetch_assoc($resultado)) { ?>

            <tr>
                <td><?= h($fila["id"]) ?></td>
                <td><?= h($fila["nombre"]) ?></td>
                <td><?= h($fila["telefono"]) ?></td>
            </tr>

        <?php } ?>

    </table>

<?php } ?>

</body>
</html>
<?php

mysqli_free_result($resultado);
mysqli_close($conexion);

?>
```

> `<?= expresion ?>` es la forma abreviada de `<?php echo expresion; ?>`. Combinada con la sintaxis de llaves, permite mezclar PHP y HTML de forma legible.

Este ejemplo recoge el flujo completo de lectura:

1. Se abre la conexión.
2. Se ejecuta la consulta.
3. Se recorren los registros.
4. Se genera el HTML (escapando los datos).
5. Se liberan los recursos.
6. Se cierra la conexión.

---

## 3.8 Filtrado de resultados con `WHERE`

Normalmente solo queremos una parte de los datos. Para ello usamos las cláusulas `WHERE`, `ORDER BY` y `LIMIT`.

`WHERE` selecciona únicamente los registros que cumplen una condición:

```sql
SELECT id, nombre, telefono
FROM clientes
WHERE id = 1;
```

También podemos filtrar por texto. Los valores de texto van **entre comillas simples** dentro de la sentencia SQL:

```sql
SELECT id, nombre, telefono
FROM clientes
WHERE nombre = 'Ana';
```

Operadores de comparación habituales: `=`, `<>` (distinto), `<`, `>`, `<=`, `>=`, y combinaciones con `AND`, `OR` y `NOT`.

### Búsquedas parciales con `LIKE`

Para buscar textos que **contengan** una cadena (en lugar de coincidir exactamente) se usa `LIKE` con el comodín `%` (cualquier número de caracteres):

```sql
-- Nombres que empiezan por "A"
SELECT id, nombre FROM clientes WHERE nombre LIKE 'A%';

-- Nombres que contienen "ar" en cualquier posición
SELECT id, nombre FROM clientes WHERE nombre LIKE '%ar%';
```

---

## 3.9 Ordenación con `ORDER BY`

```sql
SELECT id, nombre FROM clientes ORDER BY nombre;        -- ascendente (por defecto)
SELECT id, nombre FROM clientes ORDER BY nombre DESC;   -- descendente
```

Resultado de la segunda consulta (aproximado):

```text
Pedro
Marta
Ana
```

Se pueden indicar varios criterios: `ORDER BY apellidos, nombre`.

---

## 3.10 Limitación con `LIMIT`

`LIMIT` devuelve solo una parte de los registros:

```sql
SELECT id, nombre FROM clientes ORDER BY id LIMIT 5;
```

Devuelve únicamente los cinco primeros. Con dos números, `LIMIT desplazamiento, cantidad`, saltamos los primeros registros:

```sql
SELECT id, nombre FROM clientes ORDER BY id LIMIT 5, 5;   -- del 6.º al 10.º
```

Es la base de los sistemas de **paginación**. Si usas `LIMIT` conviene añadir siempre `ORDER BY`; de lo contrario el orden de las filas no está garantizado.

---

## 3.11 Búsquedas desde formularios

Queremos que el usuario pueda buscar un cliente por su nombre:

```html
<form method="GET" action="buscar.php">

    Nombre:
    <input type="text" name="nombre">

    <input type="submit" value="Buscar">

</form>
```

Al enviar el formulario, el dato llega en `$_GET["nombre"]`. **La primera vez que se carga la página, el formulario todavía no se ha enviado y esa clave no existe**, así que hay que comprobarlo con `isset()` (o con el operador `??`):

```php
$nombre = $_GET["nombre"] ?? "";
```

`??` significa: «usa `$_GET["nombre"]` si existe; si no, usa `""`».

Una primera versión (¡**incorrecta**, la veremos enseguida por qué!) construiría la consulta así:

```php
//  NUNCA HAGAS ESTO 
$nombre = $_GET["nombre"] ?? "";

$sql = "SELECT id, nombre, telefono FROM clientes WHERE nombre = '$nombre'";

$resultado = mysqli_query($conexion, $sql);
```

Funciona perfectamente cuando el usuario escribe `Ana`. El problema aparece cuando el usuario no escribe lo que esperamos.

---

## 3.12 Un grave problema de seguridad: la inyección SQL

Con el código anterior, el valor escrito por el usuario pasa **directamente a formar parte de la consulta SQL**. Veamos qué ocurre.

**Entrada normal:** el usuario escribe `Ana`.

```sql
SELECT id, nombre, telefono FROM clientes WHERE nombre = 'Ana'
```

**Entrada maliciosa:** el usuario escribe `' OR '1'='1`.

```sql
SELECT id, nombre, telefono FROM clientes WHERE nombre = '' OR '1'='1'
```

La condición `'1'='1'` es siempre verdadera, así que la consulta devuelve **todos los clientes**. El usuario ha conseguido **cambiar el significado de la consulta**.

Es un ejemplo inofensivo. Con la misma técnica un atacante puede:

- Saltarse formularios de login (`' OR '1'='1' -- ` en el campo de usuario).
- Leer tablas que no deberían ser accesibles (por ejemplo, la de usuarios y contraseñas).
- Modificar o borrar datos.

Esta vulnerabilidad se llama **inyección SQL** (*SQL Injection*) y lleva décadas entre las más graves y frecuentes de las aplicaciones web (aparece de forma recurrente en el *OWASP Top 10*).

> **Idea clave:** el problema no es qué escribe el usuario, sino que **datos y código SQL se mezclan en la misma cadena**. La solución es mantenerlos separados.

Eso es exactamente lo que hacen las **sentencias preparadas**, que estudiamos en el capítulo siguiente. Desde este momento, **nunca concatenaremos variables procedentes del usuario dentro de una consulta SQL**.

---

## Resumen

| Función | Para qué sirve |
|---------|----------------|
| `mysqli_query()` | Ejecuta una consulta SQL |
| `mysqli_fetch_assoc()` | Lee una fila como array asociativo (`null` si no hay más) |
| `mysqli_num_rows()` | Cuenta las filas devueltas por un `SELECT` |
| `mysqli_free_result()` | Libera la memoria del resultado |
| `htmlspecialchars()` / `h()` | Escapa texto antes de mostrarlo en HTML (evita XSS) |

- `WHERE` filtra, `ORDER BY` ordena y `LIMIT` limita; `LIKE` con `%` permite búsquedas parciales.
- Hay que comprobar que la fila existe antes de usarla y que `$_GET` / `$_POST` contienen la clave esperada (`isset` o `??`).
- **Todo dato que se imprime en HTML se escapa** (XSS).
- **Nunca se concatenan datos del usuario en una consulta SQL** (inyección SQL).

## Actividades

1. Crea `clientes.php` con el listado completo y comprueba que las tildes se ven bien.
2. Inserta (con el cliente de MariaDB) un cliente cuyo nombre sea `<b>Hacker</b>`. Muestra el listado con y sin `h()` y explica la diferencia.
3. Crea `buscar.php` con el formulario de búsqueda **vulnerable** (solo en tu entorno de pruebas) y comprueba qué devuelve si escribes `' OR '1'='1`. Anota la consulta SQL final que se ejecuta.
4. Escribe consultas SQL para: los 3 últimos clientes dados de alta; los clientes cuyo teléfono empieza por `600`; los clientes ordenados por nombre de Z a A.
