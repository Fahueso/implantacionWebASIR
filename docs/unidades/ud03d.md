# 4. Sentencias Preparadas

## 4.1 Introducción

En el capítulo anterior vimos que concatenar datos del usuario dentro de una consulta SQL permite la **inyección SQL**:

```php
// ❌ VULNERABLE
$nombre = $_GET["nombre"] ?? "";

$sql = "SELECT id, nombre FROM clientes WHERE nombre = '$nombre'";
```

La causa es que el dato del usuario se incorpora a la cadena SQL y MariaDB no puede distinguir qué parte es «código» y qué parte es «dato». Para evitarlo usaremos **sentencias preparadas** (*prepared statements*), el mecanismo recomendado para trabajar con bases de datos desde PHP.

---

## 4.2 ¿Qué es una sentencia preparada?

Es una consulta SQL definida previamente que contiene **marcadores de posición** (`?`) en lugar de los valores. Los valores se envían **por separado**, después.

En lugar de:

```sql
SELECT id, nombre FROM clientes WHERE id = 5;
```

escribimos:

```sql
SELECT id, nombre FROM clientes WHERE id = ?;
```

Más adelante indicaremos qué valor ocupa ese `?`.

### ¿Por qué es seguro?

1. PHP envía a MariaDB la consulta con los `?`. MariaDB la analiza y fija su estructura.
2. Después PHP envía los valores. MariaDB los trata **siempre como datos**, nunca como parte del SQL.

Si el usuario escribe `' OR '1'='1`, MariaDB buscará literalmente un cliente cuyo nombre sea el texto `' OR '1'='1`. No encontrará ninguno, y no habrá inyección posible.

Ventajas:

- Seguridad frente a inyección SQL.
- Menos errores con comillas y caracteres especiales (un cliente llamado `O'Brien` ya no rompe la consulta).
- Código más legible.
- Posibles optimizaciones internas si la misma consulta se ejecuta varias veces.

---

## 4.3 Flujo de trabajo

Siempre es el mismo proceso:

```text
Preparar la consulta     mysqli_prepare()
          ↓
Asociar parámetros       mysqli_stmt_bind_param()
          ↓
Ejecutar                 mysqli_stmt_execute()
          ↓
Obtener resultados       mysqli_stmt_get_result()
```

Parece más largo que `mysqli_query()`, pero una vez aprendido el patrón es siempre igual.

---

## 4.4 Preparación de la consulta

```php
$stmt = mysqli_prepare(
    $conexion,
    "SELECT id, nombre, telefono
     FROM clientes
     WHERE id = ?"
);
```

- `$conexion`: la conexión activa.
- `$stmt`: almacenará la sentencia preparada.
- `?`: indica que más adelante se proporcionará un valor.

En este punto la consulta **todavía no se ha ejecutado**.

> **Los `?` no llevan comillas.** Se escribe `WHERE nombre = ?`, nunca `WHERE nombre = '?'`. MariaDB se encarga de poner el valor correctamente.

---

## 4.5 Asociación de parámetros

Con `mysqli_stmt_bind_param()` indicamos qué variable ocupará cada marcador:

```php
$id = 1;

mysqli_stmt_bind_param($stmt, "i", $id);
```

El segundo argumento es una cadena con **una letra por cada `?`**, que indica el tipo de dato:

| Código | Tipo | Ejemplo |
|--------|------|---------|
| `i` | Entero (*integer*) | `id`, edad, cantidad |
| `d` | Decimal (*double*) | precio, nota media |
| `s` | Cadena de texto (*string*) | nombre, teléfono, fechas |
| `b` | Binario (*blob*) | imágenes, ficheros |

Reglas que **siempre** hay que cumplir (son los errores más habituales):

1. **El número de letras debe coincidir con el número de `?`** y con el número de variables. Si no, PHP lanza un error.
2. **El orden de las letras debe coincidir con el orden de los `?`**.
3. **Hay que pasar variables, no valores literales.** `mysqli_stmt_bind_param($stmt, "i", 1)` produce un error; debe ser `$id = 1; mysqli_stmt_bind_param($stmt, "i", $id);`. (Los parámetros se pasan por referencia.)

Un número de teléfono se declara como `s` aunque tenga solo dígitos: no vamos a hacer operaciones con él, y los ceros iniciales o el prefijo `+34` se perderían si fuera un entero.

---

## 4.6 Ejecución y recuperación de resultados

```php
mysqli_stmt_execute($stmt);

$resultado = mysqli_stmt_get_result($stmt);
```

El objeto devuelto por `mysqli_stmt_get_result()` funciona **exactamente igual** que el que devolvía `mysqli_query()`, así que lo recorremos igual:

```php
while ($fila = mysqli_fetch_assoc($resultado)) {

    echo h($fila["nombre"]);

}
```

> `mysqli_stmt_get_result()` requiere el driver nativo `mysqlnd`, que viene incluido en las instalaciones normales de PHP.

---

## 4.7 Ejemplo completo: obtener un cliente por su id

```php
<?php

require_once "conexion.php";

$conexion = conexion();

$id = 1;

$stmt = mysqli_prepare(
    $conexion,
    "SELECT id, nombre, telefono
     FROM clientes
     WHERE id = ?"
);

mysqli_stmt_bind_param($stmt, "i", $id);

mysqli_stmt_execute($stmt);

$resultado = mysqli_stmt_get_result($stmt);

$fila = mysqli_fetch_assoc($resultado);

if ($fila) {

    echo h($fila["nombre"]);

} else {

    echo "Cliente no encontrado.";

}

mysqli_free_result($resultado);
mysqli_stmt_close($stmt);
mysqli_close($conexion);

?>
```

---

## 4.8 Parámetros de texto y múltiples parámetros

Con cadenas de texto:

```php
$nombre = "Ana";

$stmt = mysqli_prepare(
    $conexion,
    "SELECT id, nombre, telefono
     FROM clientes
     WHERE nombre = ?"
);

mysqli_stmt_bind_param($stmt, "s", $nombre);
```

Con varios parámetros:

```sql
SELECT id, nombre, telefono
FROM clientes
WHERE nombre = ?
AND telefono = ?;
```

```php
mysqli_stmt_bind_param($stmt, "ss", $nombre, $telefono);
```

La primera `s` corresponde a `$nombre` y la segunda a `$telefono`. El orden de los tipos debe coincidir exactamente con el orden de los marcadores.

Con tipos mezclados, por ejemplo `WHERE nombre = ? AND id > ?`:

```php
mysqli_stmt_bind_param($stmt, "si", $nombre, $id_minimo);
```

---

## 4.9 Uso con formularios

Aquí es donde las sentencias preparadas demuestran su utilidad. Reescribimos la búsqueda del capítulo anterior, ahora de forma segura:

```php
<?php

require_once "conexion.php";

$conexion = conexion();

$nombre = trim($_GET["nombre"] ?? "");

$stmt = mysqli_prepare(
    $conexion,
    "SELECT id, nombre, telefono
     FROM clientes
     WHERE nombre = ?"
);

mysqli_stmt_bind_param($stmt, "s", $nombre);

mysqli_stmt_execute($stmt);

$resultado = mysqli_stmt_get_result($stmt);

?>
```

El valor proporcionado por el usuario **nunca se incorpora directamente a la sentencia SQL**.

---

## 4.10 Recuperación de múltiples resultados

Una sentencia preparada puede devolver varias filas. El resultado se recorre igual que cualquier otro:

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

## 4.11 Búsquedas parciales con `LIKE`

El buscador exacto resulta poco útil: nadie escribe el nombre completo. Para buscar «que contenga» combinamos `LIKE` con sentencias preparadas.

**Los `%` van en el valor, no en la consulta.**

```php
$busqueda = trim($_GET["nombre"] ?? "");

// Los comodines se añaden al VALOR
$patron = "%" . $busqueda . "%";

$stmt = mysqli_prepare(
    $conexion,
    "SELECT id, nombre, telefono
     FROM clientes
     WHERE nombre LIKE ?
     ORDER BY nombre"
);

mysqli_stmt_bind_param($stmt, "s", $patron);

mysqli_stmt_execute($stmt);

$resultado = mysqli_stmt_get_result($stmt);
```

Un error muy común es escribir `LIKE '%?%'`: dentro de las comillas el `?` ya no es un marcador, sino un carácter literal, y la consulta fallará o no devolverá nada.

---

## 4.12 Lo que las sentencias preparadas NO protegen

Los marcadores `?` solo pueden sustituir **valores** (los datos que van en `WHERE`, `VALUES`, `SET`, `LIMIT`…). **No se pueden usar para nombres de tablas, nombres de columnas ni la dirección de ordenación (`ASC` / `DESC`).**

Esto **no funciona**:

```php
// ❌ No funciona: no se puede parametrizar el nombre de una columna
"SELECT * FROM clientes ORDER BY ?"
```

Si necesitas que el usuario elija el orden (por ejemplo, haciendo clic en la cabecera de una columna), no concatenes lo que llega por `$_GET`. Usa una **lista blanca** (*whitelist*): comprueba que el valor está entre los permitidos y utiliza tú el texto SQL:

```php
$columnas_permitidas = ["id", "nombre", "telefono"];

$orden = $_GET["orden"] ?? "nombre";

if (!in_array($orden, $columnas_permitidas, true)) {

    $orden = "nombre";   // valor por defecto seguro

}

$sql = "SELECT id, nombre, telefono FROM clientes ORDER BY " . $orden;
```

Aquí se concatena `$orden`, pero es seguro porque solo puede valer uno de los tres textos de la lista, nunca algo que haya escrito el usuario.

> **Resumen de seguridad:** sentencias preparadas para los **valores**, listas blancas para **identificadores**, y `htmlspecialchars()` para lo que se **muestra**.

---

## 4.13 Liberación de recursos

Además de liberar el resultado y cerrar la conexión, se cierra la sentencia:

```php
mysqli_free_result($resultado);

mysqli_stmt_close($stmt);

mysqli_close($conexion);
```

A partir de ahora **utilizaremos sentencias preparadas siempre que una consulta incluya datos variables**.

---

## Resumen

- Una sentencia preparada separa la **consulta** (con `?`) de los **datos**, eliminando la inyección SQL.
- Flujo: `mysqli_prepare()` → `mysqli_stmt_bind_param()` → `mysqli_stmt_execute()` → `mysqli_stmt_get_result()`.
- Tipos: `i` entero, `d` decimal, `s` texto, `b` binario. Una letra por cada `?`, en el mismo orden.
- `bind_param` necesita **variables**, no literales.
- Para `LIKE`, los `%` se añaden al valor (`"%" . $texto . "%"`).
- Los `?` no sirven para nombres de columnas o tablas: usa una **lista blanca**.
- Orden de cierre: `free_result` → `stmt_close` → `close`.

## Actividades

1. Reescribe `buscar.php` de forma segura y comprueba que `' OR '1'='1` ya no devuelve todos los clientes.
2. Añade búsqueda parcial con `LIKE`. ¿Qué devuelve si el campo está vacío? ¿Te parece correcto?
3. Haz que las cabeceras de la tabla de `clientes.php` sean enlaces (`?orden=nombre`) que ordenen por esa columna usando una lista blanca. Prueba a escribir en la URL `?orden=nombre;DROP TABLE clientes` y comprueba que no tiene efecto.
4. Encuentra el error en cada uno:
   a) `mysqli_stmt_bind_param($stmt, "s", "Ana");`
   b) `WHERE nombre = ? AND telefono = ?` con `mysqli_stmt_bind_param($stmt, "s", $nombre, $telefono);`
   c) `WHERE nombre LIKE '%?%'`
