# PARTE 3: PHP - PROGRAMACIÓN DEL LADO DEL SERVIDOR

## Módulo I: Fundamentos y Arquitectura de Renderizado

### Capítulo 1. El Modelo de Ejecución de PHP
PHP (*Hypertext Preprocessor*) es un lenguaje de programación del lado del servidor. Esto significa que el código escrito en PHP no se ejecuta en el navegador del usuario, sino en el servidor. El servidor procesa las instrucciones y devuelve al navegador HTML listo para mostrar. Esta arquitectura lo hace ideal para crear páginas **dinámicas**: mostrar el nombre de un usuario, procesar formularios o conectarse a una base de datos.

**1.1 Requisitos Básicos y Entorno**
Para desarrollar en PHP se requiere:
1.  **Un servidor con PHP instalado.** Para el aprendizaje, se recomienda el **servidor embebido de PHP**, que evita la necesidad de configurar servidores complejos como Apache o Nginx.
2.  **Un editor de texto.** Herramientas como VS Code, Sublime Text o Notepad++ son el estándar.

**1.2 Guardar y Ejecutar Archivos PHP**
Los programas deben guardarse con la extensión `.php` (ej. `index.php`). El flujo de trabajo profesional es el siguiente:
1.  **Crear una carpeta de proyecto:** (ej. `miweb/`).
2.  **Guardar el archivo:** Colocar el código en `index.php` dentro de dicha carpeta.
3.  **Levantar el servidor:** Desde la terminal, situados en la carpeta, ejecutar:
    ```bash
    php -S localhost:8000
    ```
4.  **Acceso:** Abrir el navegador en `http://localhost:8000`. El servidor buscará automáticamente el archivo `index.php`. También se puede acceder a otros archivos directamente (ej. `http://localhost:8000/prueba.php`).

**1.3 Comprobación de la Instalación**
Para validar que el servidor PHP está funcionando correctamente y conocer su configuración, se puede crear un archivo llamado `php_info.php` con el siguiente contenido:
```php
<?php
  phpinfo();
?>
```
Al invocar este archivo desde el navegador, PHP desplegará una página detallada con toda la configuración del servidor, las extensiones activas y la versión instalada.

**1.4 Depuración y Ejecución Online**
Al utilizar el servidor embebido, los errores se manifiestan de dos formas: en el navegador (indicando la línea del fallo) y en la terminal (mostrando advertencias y trazas).

Es importante notar que si ejecutamos un script directamente desde la consola (`php archivo.php`), el resultado de las instrucciones `echo` se mostrará en la terminal y no en el navegador. Esto es extremadamente útil para pruebas rápidas de lógica.

Para quienes deseen probar fragmentos de código (*snippets*) rápidamente sin configurar un servidor local, existen compiladores online como el de **W3Schools**. Sin embargo, una vez que el proyecto requiera múltiples archivos, gestión de sesiones o formularios complejos, el uso del servidor embebido o un servidor web como Apache es obligatorio.

---

## Módulo II: Sintaxis Básica y Gestión de Datos

### Capítulo 2. Sintaxis y Estructura de PHP
PHP se incrusta dentro de páginas HTML. Todo el código PHP debe ir encerrado entre las etiquetas `<?php` y `?>`.

**2.1 El Rol de PHP frente al HTML**
PHP no reemplaza al HTML; su función es **generar** contenido que normalmente será HTML. Por tanto, la buena práctica es trabajar siempre con documentos HTML completos.

**Ejemplo mínimo con HTML completo:**
```php
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Ejemplo PHP</title>
</head>
<body>
  <?php
    echo "<h1>Hola Mundo!</h1><p>Ésta es mi primera página en PHP.</p>";
  ?>
</body>
</html>
```
*   `<?php`: Abre el bloque de ejecución.
*   `echo`: Instrucción para mostrar contenido en pantalla.
*   `?>`: Cierra el bloque de ejecución.

**2.2 Comentarios en PHP**
Los comentarios son esenciales para documentar la lógica del código y son ignorados por el servidor.
*   **Una línea:** Se utiliza `// comentario`.
*   **Varias líneas:** Se utiliza `/* comentario */`.

```php
<?php
  // Este es un comentario de una sola línea
  echo "Hola mundo"; // También se puede comentar al final de una línea

  /* 
     Este es un comentario de varias líneas. 
     Se usa para explicar bloques complejos o dejar notas extensas.
  */
  $nombre = "Francisco";
  echo "Bienvenido $nombre";
?>
```

---

### Capítulo 3. Variables y Constantes

**3.1 Tipado Dinámico**
En PHP, las variables no tienen un tipado estricto; se adaptan automáticamente al tipo de dato asignado.

```php
<?php
  $nombre = "Carlos"; // string
  $edad = 25;        // int
  $activo = true;     // bool

  echo "<p>Nombre: $nombre</p>";
  echo "<p>Edad: $edad</p>";
  echo "<p>Activo: $activo</p>";
?>
```

**3.2 Tipos de Datos Disponibles**
PHP soporta diversos tipos: `string` (texto), `int` (entero), `float` (decimal), `bool` (booleano), `array` (lista) y `null`. Se puede comprobar el tipo de una variable con `gettype()`.

```php
<?php
  $texto = "Hola";         // string
  $numero = 10;            // int
  $decimal = 2.5;          // float
  $activo = true;          // bool
  $lista = [1, 2, 3];      // array
  $nada = null;            // null

  echo gettype($texto);    // Muestra: string
?>
```

**3.3 Ejemplo Detallado de Cambio de Tipo (Casting)**
A continuación, se muestra cómo una misma variable puede cambiar de tipo y cómo forzar dicha conversión:

```php
<?php
  $dato = "123"; // Inicialmente es un string
  echo "<p>Tipo inicial: " . gettype($dato) . "</p>";

  $dato = (int)$dato; // Conversión explícita a entero (Casting)
  echo "<p>Convertido a entero: " . gettype($dato) . "</p>";

  $dato = 123.45; // Ahora cambia a float
  echo "<p>Cambiado a float: " . gettype($dato) . "</p>";

  $dato = true; // Ahora cambia a boolean
  echo "<p>Cambiado a boolean: " . gettype($dato) . "</p>";
?>
```

**3.4 Constantes**
Para valores que permanecen invariables, se utiliza `define()`.
```php
<?php
    define("PAIS", "España");
    echo "<p>País: " . PAIS . "</p>";
?>
```

---

## Módulo III: Operadores y Control de Flujo

### Capítulo 4. Operadores

**4.1 Operadores Aritméticos y de Asignación**
PHP soporta la suma, resta, multiplicación, división, módulo (`%`) y exponenciación (`**`). Además, permite la asignación compuesta: `$c += 2` (equivale a `$c = $c + 2`).

**Operadores aritméticos:**
```php
<?php
  $a = 10;
  $b = 3;

  echo "<p>Suma: " . ($a + $b) . "</p>";
  echo "<p>Resta: " . ($a - $b) . "</p>";
  echo "<p>Multiplicación: " . ($a * $b) . "</p>";
  echo "<p>División: " . ($a / $b) . "</p>";
  echo "<p>Módulo: " . ($a % $b) . "</p>";
  echo "<p>Exponenciación: " . ($a ** $b) . "</p>";
?>

```

**Operadores de asignación:**
```php
<?php
  $c = 5;
  $c += 2; // equivale a $c = $c + 2
  echo "<p>Asignación compuesta (+=): " . $c . "</p>";
?>
``

**4.2 Comparación y Lógica**
*   **Igual (`==`) vs Idéntico (`===`):** El primero compara el valor, el segundo compara valor y tipo.
*   **Lógicos:** `&&` (AND), `||` (OR), `!` (NOT).

**Operadores de asignación**
```php
<?php
  $a = 10;
  $b = 3;

  echo "<p>Igual (==): " . ($a == $b ? "true" : "false") . "</p>";
  echo "<p>Idéntico (===): " . ($a === $b ? "true" : "false") . "</p>";
  echo "<p>Diferente (!=): " . ($a != $b ? "true" : "false") . "</p>";
  echo "<p>No idéntico (!==): " . ($a !== $b ? "true" : "false") . "</p>";
  echo "<p>Mayor que (>): " . ($a > $b ? "true" : "false") . "</p>";
  echo "<p>Menor o igual (<=): " . ($a <= $b ? "true" : "false") . "</p>";
?>
```

**Operadores lógicos**:
```php
<?php
  $x = true;
  $y = false;

  echo "<p>AND (&&): " . ($x && $y ? "true" : "false") . "</p>";
  echo "<p>OR (||): " . ($x || $y ? "true" : "false") . "</p>";
  echo "<p>NOT (!): " . (!$x ? "true" : "false") . "</p>";
?>
```

**4.3 Incremento y Decremento**
Existen dos formas de alterar un valor en 1 unidad:
*   **Pre-incremento (`++$z`):** Incrementa primero, devuelve después.
*   **Post-incremento (`$z++`):** Devuelve el valor actual, incrementa después.

```php
<?php
  $z = 5;

  echo "<p>Pre-incremento (++z): " . (++$z) . "</p>";
  echo "<p>Post-incremento (z++): " . ($z++) . "</p>";
  echo "<p>Valor actual: " . $z . "</p>";
  echo "<p>Pre-decremento (--z): " . (--$z) . "</p>";
  echo "<p>Post-decremento (z--): " . ($z--) . "</p>";
  echo "<p>Valor final: " . $z . "</p>";
?>
```

**4.4 Concatenación y Escapado de Caracteres**
La unión de textos se realiza con el punto (`.`).
*   **Comillas dobles:** Permiten interpolación automática de variables.
*   **Comillas simples:** No interpolan; tratan todo como texto literal.

**Ejemplo de escapado y formato:**
```php
<?php
  $nombre = "Francisco";
  $edad = 40;

  // Comillas dobles: interpolación automática
  echo "<p>Interpolación con dobles: Hola $nombre, tienes $edad años.</p>";

  // Comillas simples: no hay interpolación
  echo '<p>Sin interpolación con simples: Hola $nombre, tienes $edad años.</p>';

  // Concatenación con comillas simples
  echo '<p>Concatenación con simples: Hola ' . $nombre . ', tienes ' . $edad . ' años.</p>';

  // Concatenación con comillas dobles
  echo "<p>Concatenación con dobles: Hola " . $nombre . ", tienes " . $edad . " años.</p>";
?>
```
```php
<?php
  $nombre = "Francisco";

  // Comillas simples: solo se escapan \' y \\
  echo 'Ruta en Windows: C:\\Usuarios\\Francisco<br>';

  // Comillas dobles: se pueden usar secuencias como \n, \t, \", \\ (usar pre)
  echo "<pre>Hola \"$nombre\"\n\tBienvenido al sistema.</pre>";

  // Mostrar salto de línea en HTML
  echo "<pre>Primera línea\nSegunda línea</pre>";

  // Tabulación con \t (visible en <pre>)
  echo "<pre>Nombre:\t$nombre</pre>";
?>
``

---

### Capítulo 5. Estructuras de Control

**5.1 Condicionales (`if`, `elseif`, `else` y `switch`)**
El `switch` es ideal para evaluar una variable contra múltiples casos. También puede usarse `switch(true)` para evaluar condiciones lógicas complejas.

```php
<?php
  $edad = 25;
  switch (true) {
    case ($edad >= 0 && $edad < 18):
      echo "<p>Menor de edad</p>";
      break;
    case ($edad >= 18 && $edad < 65):
      echo "<p>Adulto</p>";
      break;
    default:
      echo "<p>Adulto Mayor</p>";
      break;
  }
?>
```

**5.2 Bucles (`for`, `while`, `do-while`)**
*   `for`: Repeticiones predeterminadas por un índice.
*   `while`: Repite mientras la condición sea verdadera.
*   `do-while`: Ejecuta el código al menos una vez antes de evaluar la condición.

**Control de bucles:** `break` termina el bucle; `continue` salta a la siguiente iteración.

---

## Módulo IV: Estructuras de Datos y Modularidad

### Capítulo 6. Arrays y JSON

**6.1 Tipos de Arrays**
1.  **Indexados:** Claves numéricas. `echo $colores[1];`
2.  **Asociativos:** Claves personalizadas. `echo $persona["nombre"];`

**6.2 Recorridos con Foreach**
Es la forma más eficiente de iterar colecciones, ya sea extrayendo solo el valor o el par clave $\rightarrow$ valor.

**6.3 Generación de JSON**
Para enviar datos a una aplicación frontend, se utiliza `json_encode()`. Es imperativo añadir la cabecera `Content-Type: application/json`.

---

### Capítulo 7. Funciones y Reutilización de Código

**7.1 Definición de Funciones**
Permiten encapsular lógica para evitar la redundancia.
```php
<?php
  function saludo($nombre) {
    return "Hola, $nombre!";
  }
  echo saludo("María");
?>
```

**7.2 Inclusión de Archivos (`include` y `require`)**
Permiten fragmentar el sitio en partes reutilizables (cabecera, pie).
*   `include`: Lanza un aviso si el archivo no existe, pero el script continúa.
*   `require`: Lanza un error fatal y detiene la ejecución si el archivo no existe.

---

## Módulo V: Interacción, Persistencia y Errores

### Capítulo 8. Superglobales y Formularios
Las superglobales son arrays disponibles en cualquier ámbito del script.

**8.1 Método GET (`$_GET`)**
Envía datos a través de la URL. Ideal para búsquedas.
```php
<?php
  if (isset($_GET['nombre'])) {
      $nombre = htmlspecialchars($_GET['nombre']);
      echo "Hola, $nombre!";
  }
?>
```

**8.2 Método POST (`$_POST`)**
Envía datos ocultos. Obligatorio para datos sensibles.
```php
<?php
  if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['nombre'])) {
      $nombre = htmlspecialchars($_POST['nombre']);
      echo "Hola, $nombre!";
  }
?>
```

---

### Capítulo 9. Sesiones y Autenticación (`$_SESSION`)

HTTP es *stateless*. Las sesiones permiten que el servidor recuerde al usuario.
1.  `session_start()`: Inicia la sesión.
2.  `$_SESSION['usuario'] = "Carlos"`: Almacena datos.
3.  `session_destroy()`: Cierra la sesión.

**Seguridad:** Se recomienda usar `session_regenerate_id(true)` para evitar la fijación de sesión y controlar la expiración mediante el sello de tiempo `time()`.

---

### Capítulo 10. Gestión de Archivos y Errores

**10.1 Manejo de Archivos**
*   `file_get_contents()`: Lectura rápida de todo el archivo.
*   `fopen()` / `fgets()` / `fclose()`: Lectura eficiente línea por línea.

**10.2 Control de Excepciones**
El bloque `try-catch` captura errores sin detener la aplicación.
```php
<?php
try {
    if (!file_exists("config.txt")) {
        throw new Exception("El archivo no existe.");
    }
} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
?>
```

---

### Anexo: El archivo `php.ini`
Configuración global del servidor.
*   `display_errors`: `On` en desarrollo, `Off` en producción.
*   `upload_max_filesize`: Límite de subida de archivos.
*   `max_execution_time`: Tiempo máximo de ejecución de un script.

---

