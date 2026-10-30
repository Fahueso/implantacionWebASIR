# PHP: hoja de snippets para el examen

## 1. Ejecución y estructura básica

```bash
php -v                  # comprobar versión instalada
php -S localhost:8000   # servidor embebido (SOLO desarrollo), desde la carpeta del proyecto
php archivo.php         # ejecutar un script en consola
```

```php
<?php
phpinfo();              // versión, extensiones y config (NO dejarla en producción)
```

```php
<!doctype html>
<html lang="es">
<head><meta charset="utf-8"><title>Ejemplo</title></head>
<body>
  <?php echo "<h1>¡Hola Mundo!</h1>"; ?>
  <p>Atajo: <?= $nombre ?></p>   <!-- equivale a <?php echo $nombre; ?> -->
</body>
</html>
```

```php
<?php
// Un archivo SOLO con PHP NO lleva "?>" al final (evita espacios que rompen cabeceras)
// comentario de una línea
# también de una línea
/* varias
   líneas */
```

---

## 2. Variables, tipos y constantes

```php
<?php

$nombre = "Carlos";  // string
$edad   = 25;        // int
$precio = 2.5;       // float
$activo = true;      // bool (true se muestra "1", false se muestra "")
$nada   = null;      // null
$lista  = [1, 2, 3]; // array

// Las variables empiezan por $ y distinguen MAYÚSCULAS/minúsculas
gettype($edad);      // "integer"  (float da "double", bool da "boolean")
var_dump($edad);     // tipo + valor (ideal para depurar)
```

```php
<?php
// Casting: (int) (float) (string) (bool) (array)
$dato = "123";
$dato = (int)$dato;        // ahora es int

// Constantes
define("PAIS", "España");
const IVA = 0.21;          // alternativa moderna
echo "País: " . PAIS;      // las constantes NO llevan $
```

---

## 3. Operadores

```php
<?php
// Aritméticos: + - * / % (módulo) ** (potencia)
$a = 10; $b = 3;
echo $a + $b;   // 13
echo $a / $b;   // 3.3333...
echo $a % $b;   // 1
echo $a ** $b;  // 1000

// Asignación compuesta: += -= *= /= .=
$c = 5; $c += 2;   // 7
$txt = "Hola"; $txt .= " mundo"; // concatena sobre sí misma
```

```php
<?php
// Comparación
$a == $b     // igual en valor (convierte tipos: cuidado)
$a === $b    // igual en valor Y tipo  ← USAR POR DEFECTO
$a != $b     // distinto
$a !== $b    // no idéntico
$a <=> $b    // nave espacial: devuelve -1, 0 o 1

// Null coalescing: valor por defecto si no existe o es null
$nombre = $_GET['nombre'] ?? "invitado";
```

```php
<?php
// Lógicos: && (AND)  || (OR)  ! (NOT)   → usar símbolos, "and"/"or" tienen menor precedencia
$x = true; $y = false;
$x && $y;   // false
$x || $y;   // true
!$x;        // false
```

```php
<?php
// Incremento / decremento
$z = 5;
echo ++$z;  // 6 → PRE: incrementa y luego devuelve
echo $z++;  // 6 → POST: devuelve y luego incrementa (z pasa a 7)
echo $z;    // 7
echo --$z;  // 6
echo $z--;  // 6 (z pasa a 5)
```

---

## 4. Cadenas

```php
<?php
$nombre = "Francisco"; $edad = 40;

echo "Hola $nombre, tienes $edad años.";          // comillas DOBLES: interpolan variables y \n \t \" \\
echo 'Hola $nombre';                              // comillas SIMPLES: texto literal (solo \' y \\)
echo 'Hola ' . $nombre . ', tienes ' . $edad;     // el punto concatena
echo "Con llaves: {$nombre}s";                    // las llaves delimitan la variable
echo "Array: {$persona['edad']}";                 // en arrays asociativos dentro de "": llaves
echo "<pre>Línea1\n\tLínea2</pre>";               // <pre> hace visibles \n y \t en el navegador
```

```php
<?php
// Funciones de cadena frecuentes
strlen($t);                       // longitud
strtoupper($t); strtolower($t);   // mayúsculas / minúsculas
str_replace("a", "e", $t);        // reemplazar
substr($t, 0, 3);                 // subcadena (inicio, longitud)
trim($t);                         // quita espacios de los extremos
explode(",", "a,b,c");            // texto → array
implode(", ", $array);            // array → texto
```

---

## 5. Estructuras de control

```php
<?php
// if / elseif / else
$edad = 20; $es_estudiante = true;
if ($edad < 18) {
    echo "Menor de edad";
} elseif ($es_estudiante) {
    echo "Adulto estudiante";
} else {
    echo "Adulto";
}
```

```php
<?php
// switch: compara con varios casos; "break" evita caer al siguiente
switch ($dia) {
    case "lunes":
        echo "Empezamos la semana";
        break;
    case "sábado":     // casos apilados: comparten código
    case "domingo":
        echo "Fin de semana";
        break;
    default:
        echo "Día no reconocido";
}


```

```php
<?php
// for: repeticiones conocidas
for ($n = 1; $n <= 5; $n++) { echo $n . "<br>"; }

// for con decremento: cuenta atrás (la condición pasa a ser >= y el paso es --)
for ($n = 5; $n >= 1; $n--) { echo $n . "<br>"; }       // 5 4 3 2 1

// for con paso distinto de 1: usa asignación compuesta
for ($n = 10; $n >= 0; $n -= 2) { echo $n . "<br>"; }   // 10 8 6 4 2 0
for ($n = 0; $n <= 10; $n += 2) { echo $n . "<br>"; }   // 0 2 4 6 8 10


// while: repite MIENTRAS la condición sea cierta (puede no ejecutarse nunca)
$i = 1;
while ($i <= 5) { echo $i++ . "<br>"; }

// do-while: se ejecuta AL MENOS UNA VEZ
$n = 1;
do { echo $n++ . "<br>"; } while ($n <= 5);

// break termina el bucle; continue salta a la siguiente iteración
for ($n = 1; $n <= 10; $n++) {
    if ($n == 5) { break; }      // sale del bucle
    if ($n == 3) { continue; }   // se salta el 3
    echo $n;
}
```

---

## 6. Arrays

### Indexados (claves numéricas desde 0)

```php
<?php
$colores = ["Rojo", "Verde", "Azul"];   // [0] [1] [2]
echo $colores[0];          // Rojo
echo count($colores);      // 3

$colores[1] = "Amarillo";               // modificar
$colores[] = "Negro";                   // añadir al final
array_push($colores, "Blanco", "Gris"); // añadir varios al final
array_pop($colores);                    // elimina el ÚLTIMO
array_shift($colores);                  // elimina el PRIMERO (y reindexa)
unset($colores[1]);                     // elimina SIN reindexar (quedan huecos)
```

**Recorrer un array indexado**

```php
<?php
$colores = ["Rojo", "Verde", "Azul"];

// foreach solo con valores (lo más cómodo)
foreach ($colores as $color) {
    echo "Color: $color <br>";
}

// for clásico: el último índice es count() - 1
for ($i = 0; $i < count($colores); $i++) {
    echo $colores[$i] . "<br>";
}

// al revés
for ($i = count($colores) - 1; $i >= 0; $i--) {
    echo $colores[$i] . "<br>";             // Azul, Verde, Rojo
}
// CUIDADO: tras unset() hay huecos y el for con $colores[$i] da aviso → usa foreach o array_values()
```

### Asociativos (clave => valor)

```php
<?php
$persona = [
    "nombre" => "Ana",
    "edad"   => 25,
    "ciudad" => "Valencia",
];
echo $persona["nombre"];                // Ana
echo "Tienes {$persona['edad']} años";  // dentro de "" usar llaves

$persona["edad"]  = 26;                 // modificar
$persona["email"] = "ana@mail.com";     // añadir clave nueva
unset($persona["ciudad"]);              // eliminar
// Mantienen el orden de inserción; una clave repetida SOBRESCRIBE la anterior
```

**Recorrer un array asociativo**

```php
<?php
$persona = ["nombre" => "Ana", "edad" => 25, "ciudad" => "Valencia"];

// foreach con clave => valor (lo habitual)
foreach ($persona as $campo => $valor) {
    echo "$campo: $valor <br>";             // nombre: Ana, edad: 25, ciudad: Valencia
}

// solo valores
foreach ($persona as $valor) {
    echo "$valor <br>";
}

// solo claves
foreach (array_keys($persona) as $campo) {
    echo "$campo <br>";
}

// como tabla HTML
echo "<table border='1'>";
foreach ($persona as $campo => $valor) {
    echo "<tr><th>$campo</th><td>$valor</td></tr>";
}
echo "</table>";

// CUIDADO: un for con $persona[0] NO funciona, las claves son textos → usa foreach
```

### Comprobar si existe una clave

```php
<?php
isset($persona["telefono"]);                  // existe Y no es null
array_key_exists("edad", $persona);           // existe (aunque valga null)
$tel = $persona["telefono"] ?? "sin teléfono"; // valor por defecto
in_array("Ana", $persona);                    // busca un VALOR
array_keys($persona);                         // devuelve las claves
array_values($persona);                       // devuelve los valores
```


### Depurar arrays (echo NO imprime arrays)

```php
<?php
echo "<pre>";
print_r($notas);     // o var_dump($notas)
echo "</pre>";
```

---

## 7. Funciones, include y require

```php
<?php
// Función con tipos, valor por defecto y retorno
function saludo(string $nombre = "invitado"): string {
    return "Hola, $nombre!";
}
echo "<p>" . saludo("María") . "</p>";

// strict_types: sumar("5", 2) daría error con declare(strict_types=1)
function sumar(int $a, int $b): int { return $a + $b; }

// Ámbito local: las variables de fuera NO se ven dentro; se pasan como parámetros
```

| Instrucción | Si el archivo no existe |
|---|---|
| `include` | Aviso (*warning*); el script continúa |
| `require` | Error fatal; el script se detiene |
| `include_once` / `require_once` | Igual, pero no recarga un archivo ya incluido |

`require_once` para lo imprescindible (config, funciones); `include` para partes opcionales.

```php
<?php require "cabecera.php"; ?>   <!-- inicio del HTML, menú -->
  <h1>Página de inicio</h1>
<?php require "pie.php"; ?>        <!-- cierre del HTML -->
```

---

## 8. Formularios y superglobales

Superglobales: `$_GET`, `$_POST`, `$_SERVER`, `$_SESSION`, `$_COOKIE`, `$_FILES` (arrays accesibles desde cualquier ámbito).

### GET (datos en la URL: búsquedas, filtros)

```php
<?php
$mensaje = "";
if ($_SERVER["REQUEST_METHOD"] === "GET" && isset($_GET['nombre'])) {  // método Y campo existe
    $nombre = htmlspecialchars($_GET['nombre']);  // escapar AL MOSTRAR (evita XSS)
    $mensaje = "Hola, $nombre!";
}
?>
<form method="get">
  Nombre: <input type="text" name="nombre" required>
  <input type="submit" value="Enviar">
</form>
<p><?= $mensaje ?></p>
```

### POST (datos en el cuerpo: contraseñas, altas, modificaciones)

```php
<?php
$mensaje = "";
if ($_SERVER["REQUEST_METHOD"] === "POST" && isset($_POST['nombre'])) {  // método Y campo existe
    $nombre = htmlspecialchars($_POST['nombre']);
    $mensaje = "Hola, $nombre!";
}
?>
<form method="post">
  Nombre: <input type="text" name="nombre" required>
  <input type="submit" value="Enviar">
</form>
<p><?= $mensaje ?></p>
```

### Enviar el formulario a OTRO archivo (atributo `action`)

`action` indica a qué archivo se envían los datos. Sin `action`, el formulario se envía a sí mismo. El atributo `name` de cada campo es la clave que luego se lee en `$_GET` / `$_POST`.

**formulario.php** (solo el formulario)

```php
<!-- GET: los datos viajan en la URL → procesar.php?nombre=Ana&edad=25 -->
<form action="procesar.php" method="get">
  Nombre: <input type="text" name="nombre" required>
  Edad:   <input type="number" name="edad">
  <input type="submit" value="Enviar por GET">
</form>

<!-- POST: los datos van en el cuerpo de la petición (no se ven en la URL) -->
<form action="procesar.php" method="post">
  Nombre: <input type="text" name="nombre" required>
  Edad:   <input type="number" name="edad">
  <input type="submit" value="Enviar por POST">
</form>
```

**procesar.php** (recibe los datos del formulario)

```php
<?php
// Distinguir cómo ha llegado la petición Y comprobar con isset que el campo existe
if ($_SERVER["REQUEST_METHOD"] === "POST" && isset($_POST['nombre'])) {
    $nombre = htmlspecialchars($_POST['nombre']);         // escapar al mostrar
    $edad   = (int)($_POST['edad'] ?? 0);                 // casting: llega SIEMPRE como string
    echo "<p>POST → $nombre, $edad años</p>";

} elseif ($_SERVER["REQUEST_METHOD"] === "GET" && isset($_GET['nombre'])) {
    $nombre = htmlspecialchars($_GET['nombre']);
    $edad   = (int)($_GET['edad'] ?? 0);
    echo "<p>GET → $nombre, $edad años</p>";

} else {
    // Método correcto pero falta el campo, o se ha entrado al archivo directamente
    header("Location: formulario.php");
    exit;                                                 // siempre exit tras header
}
?>
<a href="formulario.php">Volver</a>
```

Un enlace también puede mandar datos por GET sin formulario: `<a href="procesar.php?nombre=Ana&edad=25">Enviar</a>`.

**Seguridad básica**
- POST **no cifra**: para datos sensibles hace falta **HTTPS**.
- **Nunca confíes en el usuario**: valida en el servidor (`required` del HTML se puede saltar).
- `htmlspecialchars()` **al mostrar** datos (evita XSS).
- Formularios que modifican datos: añadir **token CSRF**.

---

## 9. Sesiones y autenticación

HTTP no guarda estado (*stateless*); la sesión usa una cookie con un identificador.

```php
<?php
session_start();                       // PRIMERO, antes de cualquier salida HTML
$_SESSION['nombre'] = "Carlos";        // guardar datos
echo $_SESSION['nombre'];              // leer datos
session_unset();                       // vacía las variables de sesión
session_destroy();                     // destruye la sesión
```

### login.php

```php
<?php
session_start();

// Si ya está autenticado, al panel
if (isset($_SESSION['usuario'])) {
    header("Location: panel.php");
    exit;                               // SIEMPRE exit tras header("Location")
}

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['user'], $_POST['pass'])) {  // método Y campos existen
    $user = $_POST['user'];
    $pass = $_POST['pass'];

    if ($user === 'admin' && $pass === '1234') {   // validación de ejemplo
        session_regenerate_id(true);               // SOLO al hacer login (evita fijación de sesión)
        $_SESSION['usuario'] = $user;
        $_SESSION['ultimo_acceso'] = time();       // para controlar inactividad
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

### panel.php (página protegida + caducidad por inactividad)

```php
<?php
session_start();
$limite = 1800;                          // 30 min de inactividad máxima

// No autenticado → login
if (!isset($_SESSION['usuario'])) {
    header("Location: login.php");
    exit;
}

// Inactividad superada → cerrar sesión
if (isset($_SESSION['ultimo_acceso']) && time() - $_SESSION['ultimo_acceso'] > $limite) {
    session_unset();
    session_destroy();
    header("Location: login.php");
    exit;
}

$_SESSION['ultimo_acceso'] = time();     // renovar último acceso
?>
<!DOCTYPE html>
<html>
<body>
    <h2>Bienvenido, <?php echo htmlspecialchars($_SESSION['usuario']); ?></h2>
    <a href="logout.php">Cerrar sesión</a>
</body>
</html>
```

---

## 10. Trampas típicas de examen

- `==` convierte tipos, `===` no → usa `===`.
- `++$z` incrementa **antes** de devolver; `$z++` **después**.
- `echo` no imprime arrays → `print_r` / `var_dump`.
- Comillas simples **no** interpolan; dobles **sí**.
- `isset` es `false` si vale `null`; `array_key_exists` es `true` aunque valga `null`.
- `session_start()` antes de cualquier salida HTML.
- `session_regenerate_id(true)` solo en el login, no en cada petición.
- Tras `header("Location: ...")` siempre `exit;`.
- `htmlspecialchars()` al **mostrar**; validar siempre en el **servidor**.
