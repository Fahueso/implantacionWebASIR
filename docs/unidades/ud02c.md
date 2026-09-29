# PHP: programación del lado del servidor

## Módulo I. Fundamentos

### Capítulo 1. El modelo de ejecución

PHP (*PHP: Hypertext Preprocessor*) se ejecuta **en el servidor**, no en el navegador. El servidor procesa el código y envía al cliente solo el resultado (normalmente HTML). Por eso permite páginas **dinámicas**: mostrar datos de un usuario, procesar formularios, consultar una base de datos...

```
Navegador --> petición--> Servidor (ejecuta PHP) --> HTML--> Navegador
```

**1.1 Entorno de desarrollo**

1. PHP instalado (comprueba con `php -v`).
2. Un editor de código (VS Code, Sublime Text, Notepad++...).
3. Para aprender, el **servidor embebido** de PHP evita configurar Apache o Nginx.

**1.2 Ejecutar un proyecto**

1. Crea la carpeta `miweb/` y dentro un `index.php`.
2. Desde la terminal, en esa carpeta:
   ```bash
   php -S localhost:8000
   ```
3. Abre `http://localhost:8000`. Se sirve `index.php` por defecto; otros archivos se piden por su nombre (`/prueba.php`).

> El servidor embebido es **solo para desarrollo**. En producción se usa Apache o Nginx.

**1.3 Comprobar la instalación**

```php
<?php
phpinfo();
```

Muestra versión, extensiones y configuración. No la dejes accesible en un servidor real: revela información sensible.

**1.4 Depuración**

- Con el servidor embebido, los errores salen en el navegador y en la terminal.
- `php archivo.php` ejecuta el script en consola: el `echo` se imprime en la terminal. Útil para probar lógica rápidamente.
- Para probar fragmentos sin instalar nada existen compiladores online (W3Schools, 3v4l.org). Cuando el proyecto tenga varios archivos, sesiones o formularios, usa el servidor local.

---

## Módulo II. Sintaxis y datos

### Capítulo 2. Sintaxis básica

El código PHP va entre `<?php` y `?>`. PHP **genera** contenido, normalmente HTML, así que trabaja siempre con documentos HTML completos.

```php
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Ejemplo PHP</title>
</head>
<body>
  <?php echo "<h1>¡Hola Mundo!</h1><p>Mi primera página en PHP.</p>"; ?>
</body>
</html>
```

- Cada instrucción termina en `;`.
- `echo` muestra contenido. Atajo dentro de HTML: `<?= $nombre ?>` (equivale a `<?php echo $nombre; ?>`).
- **Un archivo con solo PHP no lleva `?>` al final**: evita espacios en blanco accidentales que rompen cabeceras.

**Comentarios**

```php
<?php
// Una línea
# También de una línea
/* Varias
   líneas */
```

---

### Capítulo 3. Variables y constantes

**3.1 Variables y tipado dinámico**

Las variables empiezan por `$`, distinguen mayúsculas de minúsculas y adoptan el tipo del valor asignado.

```php
<?php
$nombre = "Carlos"; // string
$edad = 25;         // int
$activo = true;     // bool

echo "<p>Nombre: $nombre</p>";
echo "<p>Edad: $edad</p>";
echo "<p>Activo: $activo</p>"; // true se muestra como "1"; false, como cadena vacía
```

**3.2 Tipos de datos**

| Tipo | Ejemplo |
|---|---|
| `string` | `"Hola"` |
| `int` | `10` |
| `float` | `2.5` |
| `bool` | `true` / `false` |
| `array` | `[1, 2, 3]` |
| `null` | `null` |

`gettype($x)` devuelve el tipo; `var_dump($x)` muestra tipo y valor (muy útil para depurar).

**3.3 Conversión de tipos (casting)**

```php
<?php
$dato = "123";
echo gettype($dato);   // string

$dato = (int)$dato;    // casting explícito
echo gettype($dato);   // integer

$dato = 123.45;
echo gettype($dato);   // double (es el nombre interno de float)

$dato = true;
echo gettype($dato);   // boolean
```

Casts habituales: `(int)`, `(float)`, `(string)`, `(bool)`, `(array)`.

**3.4 Tipado estricto (recomendado)**

Aunque PHP es dinámico, puedes declarar tipos en funciones y exigir que se respeten:

```php
<?php
declare(strict_types=1); // primera instrucción del archivo

function sumar(int $a, int $b): int {
    return $a + $b;
}
```

**3.5 Constantes**

```php
<?php
define("PAIS", "España");
const IVA = 0.21;           // alternativa moderna
echo "<p>País: " . PAIS . "</p>";
```

---

## Módulo III. Operadores y control de flujo

### Capítulo 4. Operadores

**4.1 Aritméticos y de asignación**

`+`, `-`, `*`, `/`, `%` (módulo), `**` (potencia). Asignación compuesta: `+=`, `-=`, `*=`, `/=`, `.=`.

```php
<?php
$a = 10;
$b = 3;
echo "<p>Suma: " . ($a + $b) . "</p>";           // 13
echo "<p>División: " . ($a / $b) . "</p>";       // 3.3333...
echo "<p>Módulo: " . ($a % $b) . "</p>";         // 1
echo "<p>Exponenciación: " . ($a ** $b) . "</p>"; // 1000

$c = 5;
$c += 2;   // $c = $c + 2
echo "<p>Asignación compuesta: $c</p>";          // 7
```

**4.2 Comparación**

- `==` compara el valor (con conversión de tipos): `0 == "a"` y `"1" == "01"` dan resultados sorprendentes según la versión.
- `===` compara valor **y** tipo. **Úsalo por defecto.**
- `!=` / `!==` son sus negaciones.
- `<`, `>`, `<=`, `>=`, y el operador nave espacial `<=>` (devuelve -1, 0 o 1).

```php
<?php
$a = 10;
$b = 3;
echo "<p>Igual (==): " . ($a == $b ? "true" : "false") . "</p>";
echo "<p>Idéntico (===): " . ($a === $b ? "true" : "false") . "</p>";
echo "<p>Diferente (!=): " . ($a != $b ? "true" : "false") . "</p>";
echo "<p>No idéntico (!==): " . ($a !== $b ? "true" : "false") . "</p>";
echo "<p>Mayor que (&gt;): " . ($a > $b ? "true" : "false") . "</p>";
echo "<p>Menor o igual (&lt;=): " . ($a <= $b ? "true" : "false") . "</p>";
```

Operadores útiles: `??` (null coalescing) da un valor por defecto si algo no existe.

```php
$nombre = $_GET['nombre'] ?? "invitado";
```

**4.3 Lógicos**

`&&` (AND), `||` (OR), `!` (NOT). También existen `and` / `or`, pero tienen menor precedencia: usa los símbolos.

```php
<?php
$x = true;
$y = false;
echo "<p>AND: " . (($x && $y) ? "true" : "false") . "</p>"; // false
echo "<p>OR: " . (($x || $y) ? "true" : "false") . "</p>";  // true
echo "<p>NOT: " . (!$x ? "true" : "false") . "</p>";        // false
```

**4.4 Incremento y decremento**

- **Pre** (`++$z`): incrementa y luego devuelve.
- **Post** (`$z++`): devuelve y luego incrementa.

```php
<?php
$z = 5;
echo ++$z; // 6  (z vale 6)
echo $z++; // 6  (se muestra 6, z pasa a 7)
echo $z;   // 7
echo --$z; // 6
echo $z--; // 6  (z pasa a 5)
echo $z;   // 5
```

**4.5 Cadenas: concatenación y escapado**

- El punto (`.`) une textos.
- **Comillas dobles**: interpolan variables y admiten `\n`, `\t`, `\"`, `\\`.
- **Comillas simples**: texto literal; solo se escapan `\'` y `\\`.

```php
<?php
$nombre = "Francisco";
$edad = 40;

echo "<p>Dobles: Hola $nombre, tienes $edad años.</p>";
echo '<p>Simples (sin interpolar): Hola $nombre.</p>';
echo '<p>Concatenando: Hola ' . $nombre . ', tienes ' . $edad . ' años.</p>';
echo "<p>Con llaves: {$nombre}s</p>"; // las llaves delimitan la variable
```

```php
<?php
$nombre = "Francisco";
echo 'Ruta: C:\\Usuarios\\Francisco<br>';
echo "<pre>Hola \"$nombre\"\n\tBienvenido al sistema.</pre>"; // <pre> hace visibles \n y \t
```

Funciones de cadena frecuentes: `strlen()`, `strtoupper()`, `strtolower()`, `str_replace()`, `substr()`, `trim()`, `explode()`, `implode()`.

---

### Capítulo 5. Estructuras de control

**5.1 Condicionales**

```php
<?php
$edad = 20;
$es_estudiante = true;

if ($edad < 18) {
    echo "<p>Eres menor de edad.</p>";
} elseif ($es_estudiante) {
    echo "<p>Eres un adulto estudiante.</p>";
} else {
    echo "<p>Eres un adulto.</p>";
}
```

`switch` compara una variable con varios casos (usa `break` para no "caer" al siguiente):

```php
<?php
$dia = "lunes";
switch ($dia) {
    case "lunes":
        echo "<p>Empezamos la semana.</p>";
        break;
    case "viernes":
        echo "<p>Último día laboral.</p>";
        break;
    case "sábado":
    case "domingo":
        echo "<p>Fin de semana.</p>";
        break;
    default:
        echo "<p>Día no reconocido.</p>";
}
```

Alternativa moderna, `match` (PHP 8): devuelve un valor y usa comparación estricta.

```php
$mensaje = match ($dia) {
    "sábado", "domingo" => "Fin de semana",
    "lunes"             => "Empezamos",
    default             => "Día laboral",
};
```

**5.2 Bucles**

- `for`: número de repeticiones conocido.
- `while`: repite mientras la condición sea cierta.
- `do-while`: se ejecuta **al menos una vez**.
- `break` termina el bucle; `continue` salta a la siguiente iteración.

```php
<?php
for ($n = 1; $n <= 5; $n++) {
    echo $n . "<br>";
}

$i = 1;
while ($i <= 5) {
    echo $i++ . "<br>";
}

$n = 1;
do {
    echo $n++ . "<br>";
} while ($n <= 5);
```

```php
<?php
for ($n = 1; $n <= 10; $n++) {
    if ($n == 5) {
        echo "Se detiene en 5<br>";
        break;
    }
    echo $n . "<br>";
}

for ($n = 1; $n <= 10; $n++) {
    if ($n == 5) {
        echo "Se omite el 5<br>";
        continue;
    }
    echo $n . "<br>";
}
```

---

## Módulo IV. Estructuras de datos y modularidad

### Capítulo 6. Arrays y JSON

**6.1 ¿Qué es un array?**

Un array es una variable que guarda **varios valores** a la vez. Cada valor se identifica con una **clave**. Según cómo sean las claves, hay dos tipos (en PHP ambos son el mismo tipo `array`, y se pueden mezclar):

| | Indexado | Asociativo |
|---|---|---|
| Claves | Números automáticos (0, 1, 2...) | Textos (u otros valores) elegidos por ti |
| Útil para | Listas de elementos del mismo tipo | Datos de una "ficha" (una persona, un producto...) |
| Ejemplo | `["Rojo", "Verde"]` | `["nombre" => "Ana", "edad" => 25]` |

Se crean con corchetes `[]` (forma moderna) o con `array()` (forma clásica, equivalente).

**6.2 Arrays indexados**

Las claves son números que **empiezan en 0**.

```php
<?php
$colores = ["Rojo", "Verde", "Azul"];
//           [0]     [1]      [2]

echo $colores[0];   // Rojo
echo $colores[2];   // Azul
echo count($colores); // 3 (número de elementos)
```

Modificar, añadir y eliminar:

```php
<?php
$colores = ["Rojo", "Verde", "Azul"];

$colores[1] = "Amarillo";   // modifica la posición 1
$colores[] = "Negro";       // añade al final (recibe la clave 3)
array_push($colores, "Blanco", "Gris"); // añade varios al final
array_pop($colores);        // elimina el último
array_shift($colores);      // elimina el primero (y reindexa)

unset($colores[1]);         // elimina esa posición SIN reindexar
```

Tras `unset()` las claves quedan con huecos (por ejemplo 0, 2, 3). Si necesitas volver a numerarlas: `$colores = array_values($colores);`.

**6.3 Arrays asociativos**

Aquí eliges tú la clave, que suele ser un texto. Se escribe `clave => valor`.

```php
<?php
$persona = [
    "nombre" => "Ana",
    "edad"   => 25,
    "ciudad" => "Valencia",
];

echo $persona["nombre"];  // Ana
echo "Tienes {$persona['edad']} años"; // dentro de comillas dobles, usa llaves
```

Modificar, añadir y eliminar:

```php
<?php
$persona["edad"] = 26;              // modifica
$persona["email"] = "ana@mail.com"; // añade una clave nueva
unset($persona["ciudad"]);          // elimina
```

**Comprobar si una clave existe** (acceder a una clave inexistente genera un aviso):

```php
<?php
if (isset($persona["telefono"])) {          // existe y no es null
    echo $persona["telefono"];
}
if (array_key_exists("edad", $persona)) { } // existe (aunque valga null)

$tel = $persona["telefono"] ?? "sin teléfono"; // valor por defecto
```

Otras funciones: `in_array("Ana", $persona)` busca un **valor**; `array_keys($persona)` devuelve las claves; `array_values($persona)`, los valores.

> Los arrays asociativos **conservan el orden** en que se insertan. Una clave repetida sobrescribe la anterior.

**6.4 Arrays multidimensionales (arrays de arrays)**

Un elemento puede ser otro array. Es la estructura típica para representar una **tabla de registros**, y es también el formato de lo que devuelve una base de datos.

```php
<?php
$alumnos = [
    ["nombre" => "Juan",  "nota" => 8.5],
    ["nombre" => "Ana",   "nota" => 9.2],
    ["nombre" => "Carlos","nota" => 7.8],
];

echo $alumnos[1]["nombre"]; // Ana  → primero la fila, luego la clave

foreach ($alumnos as $alumno) {
    echo "{$alumno['nombre']}: {$alumno['nota']}<br>";
}
```

**6.5 Recorrer con `foreach`**

Es la forma más cómoda de recorrer un array. Tiene dos variantes:

```php
<?php
// Solo valores (arrays indexados)
$colores = ["Rojo", "Verde", "Azul"];
foreach ($colores as $color) {
    echo "Color: $color <br>";
}

// Clave y valor (arrays asociativos, y también indexados si quieres la posición)
$estudiantes = ["Juan" => 8.5, "Ana" => 9.2, "Carlos" => 7.8, "María" => 6.5];
foreach ($estudiantes as $nombre => $nota) {
    echo "Estudiante: $nombre - Nota: $nota <br>";
}
```

Ejemplo práctico: mostrar un array asociativo como tabla HTML.

```php
<?php
$persona = ["nombre" => "Ana", "edad" => 25];
echo "<table border='1'>";
foreach ($persona as $campo => $valor) {
    echo "<tr><th>$campo</th><td>$valor</td></tr>";
}
echo "</table>";
```

**6.6 Funciones útiles**

| Función | Qué hace |
|---|---|
| `count($a)` | Número de elementos |
| `sort($a)` / `rsort($a)` | Ordena valores (asc./desc.) y **reindexa** |
| `asort($a)` / `arsort($a)` | Ordena por valor **manteniendo las claves** |
| `ksort($a)` / `krsort($a)` | Ordena por clave |
| `array_sum($a)` | Suma de los valores |
| `array_merge($a, $b)` | Une arrays |
| `array_slice($a, 1, 2)` | Extrae una parte |
| `array_map(fn, $a)` | Aplica una función a cada elemento |
| `array_filter($a, fn)` | Se queda con los que cumplen la condición |
| `implode(", ", $a)` / `explode(",", $t)` | Array → texto / texto → array |

```php
<?php
$notas = ["Juan" => 8.5, "Ana" => 9.2, "Carlos" => 7.8];

arsort($notas);                         // Ana, Juan, Carlos (de mayor a menor nota)
$media = array_sum($notas) / count($notas);
$aprobados = array_filter($notas, fn($n) => $n >= 8);
```

**Depurar arrays:** `echo` no puede imprimir un array. Usa `print_r()` o `var_dump()`, mejor dentro de `<pre>`:

```php
<?php
echo "<pre>";
print_r($notas);
echo "</pre>";
```

**6.7 Generar JSON**


`json_encode()` convierte un array en JSON. Hay que enviar antes la cabecera correcta:

```php
<?php
header('Content-Type: application/json; charset=utf-8');
$estudiantes = ["Juan" => 8.5, "Ana" => 9.2];
echo json_encode($estudiantes);
```

Para el camino inverso: `json_decode($texto, true)` devuelve un array asociativo.

---

### Capítulo 7. Funciones y reutilización

**7.1 Funciones**

```php
<?php
function saludo(string $nombre = "invitado"): string {
    return "Hola, $nombre!";
}
echo "<p>" . saludo("María") . "</p>";
```

Las variables de fuera **no** son visibles dentro de una función (ámbito local); se pasan como parámetros.

**7.2 `include` y `require`**

| Instrucción | Si el archivo no existe |
|---|---|
| `include` | Aviso (*warning*); el script continúa |
| `require` | Error fatal; el script se detiene |
| `include_once` / `require_once` | Igual, pero no vuelve a cargar un archivo ya incluido |

Usa `require_once` para código imprescindible (configuración, funciones) e `include` para partes opcionales.

**Ejemplo de arquitectura**

- `cabecera.php`: inicio del HTML (`<!doctype>`, `<head>`, menú).
- `pie.php`: cierre del HTML.
- `index.php`:

```php
<?php require "cabecera.php"; ?>
  <h1>Página de inicio</h1>
  <p>Bienvenido a mi primera página en PHP con includes.</p>
<?php require "pie.php"; ?>
```

---

## Módulo V. Interacción, persistencia y errores

### Capítulo 8. Formularios y superglobales

Las superglobales (`$_GET`, `$_POST`, `$_SERVER`, `$_SESSION`, `$_COOKIE`, `$_FILES`) son arrays accesibles desde cualquier ámbito.

**8.1 GET**: los datos viajan en la URL (`?nombre=Ana`). Adecuado para búsquedas y filtros; se pueden compartir y guardar en favoritos.

```php
<?php
$mensaje = "";
if (isset($_GET['nombre'])) {
    $nombre = htmlspecialchars($_GET['nombre']);
    $mensaje = "Hola, $nombre!";
}
?>
<form method="get">
  Nombre: <input type="text" name="nombre" required>
  <input type="submit" value="Enviar">
</form>
<p><?= $mensaje ?></p>
```

**8.2 POST**: los datos van en el cuerpo de la petición, no en la URL. Úsalo para contraseñas, altas y cualquier acción que modifique datos.

```php
<?php
$mensaje = "";
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $nombre = htmlspecialchars($_POST['nombre'] ?? "");
    $mensaje = "Hola, $nombre!";
}
?>
<form method="post">
  Nombre: <input type="text" name="nombre" required>
  <input type="submit" value="Enviar">
</form>
<p><?= $mensaje ?></p>
```

> **Seguridad básica**
> - POST **no cifra** nada: para proteger datos sensibles hace falta HTTPS.
> - **Nunca confíes en lo que envía el usuario.** Valida en el servidor (el atributo `required` del HTML se puede saltar).
> - Escapa con `htmlspecialchars()` **al mostrar** datos (evita XSS).
> - En formularios que modifican datos añade un token CSRF.

---

### Capítulo 9. Sesiones y autenticación

HTTP no guarda estado (*stateless*). Las sesiones permiten al servidor recordar al usuario entre peticiones mediante una cookie con un identificador.

1. `session_start()`: inicia o reanuda la sesión (antes de cualquier salida HTML).
2. `$_SESSION['usuario'] = "Carlos"`: guarda datos.
3. `session_unset()` + `session_destroy()`: cierra la sesión.

```php
<?php
session_start();
$mensaje = "";

if ($_SERVER["REQUEST_METHOD"] === "POST" && isset($_POST['nombre'])) {
    $nombre = htmlspecialchars($_POST['nombre']);
    $_SESSION['nombre'] = $nombre;
    $mensaje = "Hola, $nombre!";
} elseif (isset($_SESSION['nombre'])) {
    $mensaje = "Bienvenido de nuevo, " . $_SESSION['nombre'] . "!";
}
?>
<form method="post">
  Nombre: <input type="text" name="nombre" required>
  <input type="submit" value="Enviar">
</form>
<p><?= $mensaje ?></p>
```

**Buenas prácticas de seguridad**

- Ejecuta `session_regenerate_id(true)` **justo al iniciar sesión (login)**, no en cada petición. Evita la fijación de sesión.
- Controla la inactividad con `time()`.
- Tras `header("Location: ...")` pon siempre `exit;`.

```php
<?php
session_start();

// Si ya está autenticado, lo mandamos al panel
if (isset($_SESSION['usuario'])) {
    header("Location: panel.php");
    exit;
}

// Si llega el formulario
if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $user = $_POST['user'] ?? '';
    $pass = $_POST['pass'] ?? '';

    // Ejemplo simple de validación
    if ($user === 'admin' && $pass === '1234') {

        // Seguridad: regenerar ID
        session_regenerate_id(true);

        $_SESSION['usuario'] = $user;
        $_SESSION['ultimo_acceso'] = time();

        header("Location: panel.php");
        exit;
    }

    $error = "Credenciales incorrectas";
}
?>

<!DOCTYPE html>
<html>
<body>
    <h2>Login</h2>

    <?php if (isset($error)) echo "<p style='color:red'>$error</p>"; ?>

    <form method="post">
        Usuario: <input type="text" name="user"><br>
        Contraseña: <input type="password" name="pass"><br>
        <button type="submit">Entrar</button>
    </form>
</body>
</html>

```
**Codigo de `panel.php`:
```php
<?php
session_start();

// Tiempo máximo de inactividad (30 min)
$limite = 1800;

// Si no está autenticado → login
if (!isset($_SESSION['usuario'])) {
    header("Location: login.php");
    exit;
}

// Si existe último acceso y supera el límite → cerrar sesión
if (isset($_SESSION['ultimo_acceso']) && time() - $_SESSION['ultimo_acceso'] > $limite) {

    session_unset();
    session_destroy();

    header("Location: login.php");
    exit;
}

// Actualizar último acceso
$_SESSION['ultimo_acceso'] = time();
?>

<!DOCTYPE html>
<html>
<body>
    <h2>Bienvenido, <?php echo htmlspecialchars($_SESSION['usuario']); ?></h2>

    <p>Has iniciado sesión correctamente.</p>

    <a href="logout.php">Cerrar sesión</a>
</body>
</html>

```


---

### Capítulo 10. Archivos, errores y bases de datos

**10.1 Archivos**

- `file_get_contents()`: lee el archivo entero (devuelve `false` si falla).
- `fopen()` / `fgets()` / `fclose()`: lectura línea a línea, más eficiente en archivos grandes.
- `file_put_contents()`: escribe (con `FILE_APPEND` añade al final).

```php
<?php
$contenido = file_get_contents("ejemplo.txt");
echo "<pre>" . htmlspecialchars($contenido) . "</pre>";

$archivo = fopen("ejemplo.txt", "r");
while (($linea = fgets($archivo)) !== false) { // !== false: una línea "0" no corta el bucle
    echo "<p>" . htmlspecialchars($linea) . "</p>";
}
fclose($archivo);
```

**10.2 Excepciones**

`try-catch` captura errores sin detener la aplicación.

```php
<?php
try {
    if (!file_exists("config.txt")) {
        throw new Exception("El archivo no existe.");
    }
} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
} finally {
    echo "<p>Esto se ejecuta siempre.</p>";
}
```


---

### Anexo: `php.ini`

Configuración global del servidor. Localízalo con `php --ini` o `phpinfo()`.

| Directiva | Función |
|---|---|
| `display_errors` | `On` en desarrollo, `Off` en producción (registra en log con `log_errors`) |
| `error_reporting` | Nivel de errores a notificar (`E_ALL` en desarrollo) |
| `upload_max_filesize` / `post_max_size` | Límite de subida de archivos y de datos POST |
| `max_execution_time` | Tiempo máximo de ejecución de un script |
| `memory_limit` | Memoria máxima por script |