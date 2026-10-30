# Anexo B. Errores frecuentes y cómo resolverlos

## B.1 Errores de conexión y configuración

| Síntoma | Causa probable | Solución |
|---------|----------------|----------|
| `Access denied for user 'app_web'@'localhost'` | Usuario o contraseña incorrectos, o el usuario no existe | Revisa `SetEnv` y ejecuta `SHOW GRANTS FOR 'app_web'@'localhost';` |
| `Unknown database 'empresa'` | La base de datos no está creada | Ejecuta el script del Anexo A |
| `Class "mysqli" not found` / `Call to undefined function mysqli_connect()` | Extensión `mysqli` no instalada | `sudo apt install php-mysql` y reinicia Apache |
| `Connection refused` | MariaDB está parado | `sudo systemctl status mariadb` / `start mariadb` |
| `Call to undefined function apache_getenv()` | PHP no se ejecuta como módulo de Apache (PHP-FPM) | Usa `$_SERVER[...]` / `getenv()` (capítulo 8) |
| `config("DB_HOST")` devuelve `null` | Falta `a2enconf`, no se recargó Apache o la variable está mal escrita | `sudo a2enconf database && sudo systemctl reload apache2` |
| Pantalla en blanco | Error fatal con `display_errors` desactivado | Mira `/var/log/apache2/error.log` |
| Tildes y ñ salen como `Ã±`, `Ã³` | Falta `mysqli_set_charset` o las páginas no son UTF-8 | `mysqli_set_charset($conexion, "utf8mb4")` y `<meta charset="UTF-8">` |

## B.2 Errores de `require_once` y rutas

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `Failed opening required 'conexion.php'` | Ruta incorrecta | Usa `__DIR__ . "/../config/conexion.php"` |
| `Cannot redeclare conexion()` | El fichero se carga dos veces con `include` | Usa `require_once` |
| `Cannot modify header information - headers already sent` | Se imprimió algo (HTML, `echo`, un espacio antes de `<?php`) antes de `header()` | Haz las redirecciones **antes** de cualquier salida; no cierres con `?>` los ficheros solo-PHP |

## B.3 Errores con sentencias preparadas

| Mensaje | Causa | Solución |
|---------|-------|----------|
| `mysqli_stmt_bind_param(): Argument #3 ($var) cannot be passed by reference` | Se ha pasado un literal (`"Ana"`, `1`) | Guarda el valor en una variable y pásala |
| `The number of elements in the type definition string must match the number of bind variables` | Letras de tipo distintas del número de variables | Una letra por cada `?` y por cada variable |
| La consulta no devuelve nada con `LIKE '%?%'` | El `?` dentro de comillas no es un marcador | `LIKE ?` y pasar `"%" . $texto . "%"` |
| `Call to undefined function mysqli_stmt_get_result()` | Falta el driver `mysqlnd` | Instala `php-mysql` del repositorio oficial de la distribución |
| `You have an error in your SQL syntax` | Error en la consulta (coma de más, palabra reservada...) | Prueba la consulta en el cliente `mariadb` antes de ponerla en PHP |
| Se intenta parametrizar `ORDER BY ?` o un nombre de tabla | Los `?` solo sustituyen valores | Lista blanca (capítulo 4) |

## B.4 Avisos (*warnings*) habituales

| Mensaje | Causa | Solución |
|---------|-------|----------|
| `Undefined array key "nombre"` en `$_GET` / `$_POST` | El formulario aún no se ha enviado | `$_POST["nombre"] ?? ""` |
| `Trying to access array offset on value of type null` | `mysqli_fetch_assoc()` devolvió `null` (no había filas) y se usó el resultado | `if ($fila) { ... }` |
| `Undefined array key "nombre"` en un JOIN | Columnas con el mismo nombre sin alias, o alias mal escrito | Alias (`AS`) en todas las columnas repetidas |

## B.5 Errores de MariaDB que verás en el log

| Código | Mensaje | Significado |
|--------|---------|-------------|
| 1044 / 1142 | `command denied` | El usuario `app_web` no tiene ese permiso (`GRANT`) |
| 1045 | `Access denied` | Credenciales incorrectas |
| 1048 | `Column 'nombre' cannot be null` | Se intenta guardar `NULL` en una columna `NOT NULL` |
| 1062 | `Duplicate entry` | Valor duplicado en una columna `UNIQUE` o `PRIMARY KEY` |
| 1451 | `Cannot delete or update a parent row` | Se borra un registro del que dependen otros (clave foránea) |
| 1452 | `Cannot add or update a child row` | Se inserta una clave foránea que no existe (p. ej. grupo inexistente) |
| 1406 | `Data too long for column` | El texto es más largo que la columna |

## B.6 Lista de comprobación antes de dar por buena una página

- [ ] ¿Se usa `require_once` para cargar la conexión?
- [ ] ¿Todas las consultas con datos del usuario son sentencias preparadas?
- [ ] ¿Todo lo que se imprime en HTML pasa por `h()` / `htmlspecialchars()`?
- [ ] ¿Se comprueban `$_GET` / `$_POST` con `??` o `isset()`?
- [ ] ¿Se valida en el servidor (vacíos, longitud, formato, `id` entero)?
- [ ] ¿`UPDATE` y `DELETE` llevan `WHERE`?
- [ ] ¿Las operaciones que modifican datos usan POST y terminan con redirección?
- [ ] ¿Los errores técnicos van al log y no a la pantalla?
- [ ] ¿No hay contraseñas en el código ni en el repositorio?
- [ ] ¿La aplicación usa un usuario con los mínimos permisos?
