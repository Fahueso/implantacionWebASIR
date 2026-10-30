# 8. Gestión de Credenciales y Configuración

## 8.1 Introducción

Hasta ahora, para conectarnos a MariaDB hemos usado un código como este:

```php
$conexion = mysqli_connect("localhost", "app_web", "password", "empresa");
```

Funciona y es adecuado para aprender, pero tiene un problema grave: las **credenciales están escritas en el código fuente**. En aplicaciones reales debe evitarse siempre que sea posible.

Las credenciales son **configuración**, no lógica del programa, y deben mantenerse separadas.

---

## 8.2 ¿Por qué no guardar credenciales en el código?

### Exposición accidental

Si el código se publica, se comparte, se sube a un repositorio o un fallo de configuración hace que el servidor muestre el fichero PHP como texto, las credenciales quedan expuestas. Es un problema muy real: hay bots que rastrean continuamente GitHub buscando contraseñas publicadas por error.

### Mantenimiento complejo

Si la contraseña está repetida en varios sitios, cambiarla obliga a editar muchos archivos.

### Múltiples entornos

Una aplicación pasa por distintos entornos, cada uno con su configuración:

- **Desarrollo** (tu ordenador).
- **Pruebas**.
- **Producción** (el servidor real).

Si las credenciales están en el código, hay que modificar archivos cada vez que se despliega, con el riesgo de olvidar algo o subir por error la contraseña de producción.

### Separar código y configuración

| Código | Configuración |
|--------|---------------|
| Lógica de la aplicación | Valores que cambian entre instalaciones |
| Es igual en todos los entornos | Servidor, usuario, contraseña, base de datos, puerto |
| Se puede publicar | **No** se publica |

---

## 8.3 Variables de entorno

Una técnica muy extendida consiste en usar **variables de entorno**: valores definidos fuera de la aplicación (por el sistema operativo o por el servidor web) y que el programa lee al ejecutarse.

```text
DB_HOST=localhost
DB_NAME=empresa
DB_USER=app_web
DB_PASS=password_seguro
```

La ventaja: los valores **no aparecen en el código fuente** ni en el repositorio.

---

## 8.4 Configuración de Apache

Apache puede definir variables de entorno con la directiva `SetEnv`. Lo más cómodo es crear un fichero de configuración independiente:

```bash
sudo nano /etc/apache2/conf-available/database.conf
```

con el contenido:

```apache
SetEnv DB_HOST localhost
SetEnv DB_NAME empresa
SetEnv DB_USER app_web
SetEnv DB_PASS password_seguro
```

Como este fichero contiene una contraseña, **restringe sus permisos**. Apache lee su configuración como `root` al arrancar, así que no necesita que otros usuarios puedan leerlo:

```bash
sudo chown root:root /etc/apache2/conf-available/database.conf
sudo chmod 600 /etc/apache2/conf-available/database.conf
```

---

## 8.5 Activación de la configuración

Activamos el fichero y recargamos Apache:

```bash
sudo a2enconf database
```

Salida:

```text
Enabling conf database.
```

```bash
sudo systemctl reload apache2
```

(`restart` también vale; `reload` aplica la configuración sin cortar las conexiones activas.)

A partir de ese momento las variables están disponibles para las aplicaciones PHP del servidor.

> **Si usas más de un sitio virtual**, puedes poner los `SetEnv` dentro del `<VirtualHost>` de la aplicación concreta en lugar de hacerlos globales, para que otras aplicaciones del mismo servidor no vean estas credenciales.

---

## 8.6 Acceso a las variables desde PHP

Hay varias formas de leer una variable de entorno en PHP. Las dos más habituales son:

```php
getenv("DB_HOST");
$_SERVER["DB_HOST"];
```

Según cómo esté montado el servidor (módulo `mod_php` o PHP-FPM) las variables definidas con `SetEnv` pueden aparecer en una, en otra o en ambas. Para que el código funcione en cualquier configuración, creamos una función que prueba las dos:

```php
function config($nombre)
{

    if (isset($_SERVER[$nombre])) {

        return $_SERVER[$nombre];

    }

    $valor = getenv($nombre);

    if ($valor !== false) {

        return $valor;

    }

    error_log("Falta la variable de configuración: " . $nombre);

    return null;

}
```

> **Sobre `apache_getenv()`:** aparece en mucha documentación antigua, pero solo existe cuando PHP se ejecuta como módulo de Apache (`mod_php`). En un servidor con PHP-FPM (muy habitual hoy) da el error *Call to undefined function apache_getenv()*. Por eso usamos `$_SERVER` / `getenv()`.

Para comprobar que la configuración funciona, crea una página de prueba **temporal**:

```php
<?php

echo $_SERVER["DB_HOST"] ?? getenv("DB_HOST") ?: "variable no definida";

?>
```

Si todo va bien, el navegador mostrará `localhost`. **Borra el fichero después.**

---

## 8.7 Conexión utilizando variables de entorno

Con los valores leídos, la conexión no contiene ninguna credencial:

```php
$conexion = mysqli_connect(
    config("DB_HOST"),
    config("DB_USER"),
    config("DB_PASS"),
    config("DB_NAME")
);
```

---

## 8.8 Centralización de la conexión

El fichero `conexion.php` queda así (resume todo lo visto en la unidad):

```php
<?php

mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

function config($nombre)
{

    if (isset($_SERVER[$nombre])) {

        return $_SERVER[$nombre];

    }

    $valor = getenv($nombre);

    return ($valor !== false) ? $valor : null;

}

function conexion()
{

    try {

        $conexion = mysqli_connect(
            config("DB_HOST"),
            config("DB_USER"),
            config("DB_PASS"),
            config("DB_NAME")
        );

        mysqli_set_charset($conexion, "utf8mb4");

    } catch (mysqli_sql_exception $e) {

        error_log("Error de conexión: " . $e->getMessage());

        http_response_code(500);
        die("Error interno: no se pudo conectar con la base de datos.");

    }

    return $conexion;

}

function h($texto)
{

    return htmlspecialchars((string) $texto, ENT_QUOTES, "UTF-8");

}

?>
```

Cualquier cambio de configuración se hace ahora **fuera del código** (en `database.conf`), sin tocar ni un fichero PHP.

---

## 8.9 Organización recomendada del proyecto

Hasta ahora todos los ficheros estaban en la misma carpeta, y todo lo que está en la carpeta pública del servidor es accesible desde el navegador. Una estructura más segura y ordenada:

```text
proyecto/
├── config/
│   └── conexion.php
├── public/
│   ├── index.php
│   ├── clientes.php
│   ├── insertar.php
│   ├── actualizar.php
│   └── eliminar.php
└── recursos/
```

- `config/`: configuración y código de conexión. **Fuera** de la carpeta pública.
- `public/`: las páginas a las que accede el usuario. Es la única carpeta que Apache sirve.
- `recursos/`: ficheros auxiliares.

Para que funcione, el `DocumentRoot` de Apache debe apuntar a `public/`:

```apache
<VirtualHost *:80>
    ServerName empresa.local
    DocumentRoot /var/www/proyecto/public
</VirtualHost>
```

Así, si algún día Apache o PHP se configuran mal y sirven el código como texto, el atacante no podrá llegar a los ficheros de `config/`, porque no están dentro del `DocumentRoot`.

### Cambiar los `require_once`

Al mover `conexion.php` a `config/`, hay que modificar la ruta en cada página. Usamos `__DIR__` (la carpeta del fichero actual) para que la ruta funcione siempre, sin depender de desde dónde se ejecute el script:

```php
require_once __DIR__ . "/../config/conexion.php";
```

`..` significa «subir una carpeta»: desde `public/` subimos a `proyecto/` y bajamos a `config/`.

---

## 8.10 Seguridad y repositorios Git

Es habitual guardar el proyecto en un repositorio Git. Si las credenciales aparecen en los ficheros y el repositorio se hace público (o se filtra), cualquiera tendrá acceso. Y **borrar la contraseña en un commit posterior no basta**: sigue en el historial del repositorio. Si ocurre, hay que **cambiar la contraseña** inmediatamente.

Recomendaciones:

- Separar configuración y código (variables de entorno).
- No guardar contraseñas en el repositorio.
- Si algún fichero de configuración local contiene datos sensibles, excluirlo con `.gitignore`:

```text
# .gitignore
config.local.php
.env
```

- Documentar en el repositorio qué variables hacen falta, **sin valores reales** (por ejemplo, un fichero `README` con `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`).
- Usar un usuario de base de datos específico para cada aplicación.
- Mostrar errores técnicos solo en desarrollo: en producción, `display_errors = Off` en `php.ini` y los errores van al log.

### Principio de mínimo privilegio

Una aplicación web **no debe conectarse con el usuario administrador** (`root`) de MariaDB. Si hubiera una vulnerabilidad, el atacante tendría el control total del servidor de bases de datos.

Se crea un usuario específico, **restringido al equipo desde el que se conecta**:

```sql
CREATE USER 'app_web'@'localhost'
IDENTIFIED BY 'password_seguro';
```

y se le conceden **solo los permisos necesarios sobre solo la base de datos necesaria**:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.*
TO 'app_web'@'localhost';
```

Fíjate en `@'localhost'`: **un usuario creado sin indicar el host (`'app_web'`) equivale a `'app_web'@'%'`, es decir, puede conectarse desde cualquier máquina de la red.** Restringirlo a `localhost` reduce mucho la superficie de ataque.

Para comprobar los permisos concedidos:

```sql
SHOW GRANTS FOR 'app_web'@'localhost';
```

Con este usuario la aplicación puede leer, insertar, modificar y borrar datos, pero **no** puede borrar tablas (`DROP`), crear usuarios ni acceder a otras bases de datos. Los errores de programación o las vulnerabilidades tienen un impacto mucho menor.

---

## 8.11 Resumen

- Las credenciales **no se escriben en el código**: son configuración.
- Las **variables de entorno** (`SetEnv` en Apache) permiten sacar la configuración del código y tener una distinta por entorno.
- Se leen en PHP con `$_SERVER[...]` / `getenv()`. `apache_getenv()` solo funciona con `mod_php`.
- El fichero de configuración de Apache debe tener permisos restrictivos (`chmod 600`).
- La conexión se centraliza en un único fichero, ubicado **fuera** de la carpeta pública (`config/`), y se carga con `require_once __DIR__ . "/../config/conexion.php"`.
- No se suben contraseñas a Git; si ocurre, se cambia la contraseña.
- **Mínimo privilegio:** usuario propio de la aplicación (`'app_web'@'localhost'`) con solo `SELECT, INSERT, UPDATE, DELETE` sobre su base de datos.

Con este capítulo finalizamos los fundamentos para desarrollar aplicaciones PHP que se conectan a MariaDB, consultan, insertan, modifican y eliminan información, y organizan de forma segura su configuración.

## Actividades

1. Configura las variables de entorno con `SetEnv` y comprueba que PHP las lee. Cambia la contraseña en `database.conf`, recarga Apache y comprueba que ningún fichero PHP necesita modificarse.
2. Reorganiza tu aplicación de clientes con las carpetas `config/` y `public/`, apuntando el `DocumentRoot` a `public/`. Prueba a acceder desde el navegador a `/conexion.php`: ¿qué ocurre?
3. Crea el usuario `app_web`@`localhost` con permisos solo de `SELECT` y prueba a insertar un cliente desde la aplicación. ¿Qué error obtienes? ¿Dónde lo ves? Después concédele `INSERT`.
4. Explica por qué es peligroso que el usuario de la aplicación sea `root` y por qué no es suficiente con borrar la contraseña de un commit antiguo.
5. **Repaso de la unidad:** haz una lista de todas las medidas de seguridad estudiadas (sentencias preparadas, lista blanca, `htmlspecialchars`, validación en servidor, POST para modificar datos, excepciones sin detalle al usuario, credenciales fuera del código, mínimo privilegio) y, para cada una, indica de qué amenaza protege.
