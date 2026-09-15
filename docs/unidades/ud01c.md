# Bloque 3: El Motor de Aplicación y la Base de Datos (PHP y MariaDB)

## 3.1. Fundamentos de PHP y su Integración con el Servidor

Hasta ahora, hemos utilizado Apache para servir recursos estáticos, como páginas HTML o imágenes. Sin embargo, las aplicaciones web modernas requieren capacidad dinámica: procesar formularios, gestionar sesiones de usuario o consultar bases de datos en tiempo real. Para lograr esto, necesitamos un lenguaje de programación que se ejecute en el servidor, y en este módulo utilizaremos **PHP** (*PHP: Hypertext Preprocessor*).

PHP es la base de plataformas masivas como Moodle, WordPress, Nextcloud y phpMyAdmin. Su característica fundamental es que es un lenguaje de **lado del servidor**. Esto significa que el código PHP se ejecuta íntegramente en el servidor y el navegador del usuario nunca recibe el código fuente, sino únicamente el resultado generado en formato HTML. 

**Ejemplo de flujo:**
Si tenemos un archivo con este código:
```php
<?php
echo "<h1>Hola ASIR</h1>";
?>
```
El usuario nunca ve las etiquetas `<?php ?>`, sino que recibe únicamente:
```html
<h1>Hola ASIR</h1>
```

### El flujo de procesamiento en el servidor
Cuando un usuario solicita un archivo con extensión `.php`, Apache no puede enviarlo directamente. En su lugar, Apache delega el archivo al **intérprete de PHP**. Este procesa el código, genera una respuesta en HTML y se la devuelve a Apache, quien finalmente la entrega al cliente. Este mecanismo permite generar contenido dinámico adaptado a cada usuario, gestionar sesiones y acceder a bases de datos.


## 3.2. Instalación y Configuración del Entorno PHP

En sistemas Debian y Ubuntu, la instalación se realiza a través del gestor de paquetes APT. Primero actualizamos los repositorios:
```bash
sudo apt update
```
E instalamos el metapaquete principal:
```bash
sudo apt install php
```
Tras la instalación, es obligatorio reiniciar Apache para que reconozca la integración:
```bash
sudo systemctl restart apache2
```

### Paquetes y componentes clave
Durante la instalación se añaden varios componentes esenciales:
*   **php:** Metapaquete principal con la versión recomendada.
*   **php-cli:** Permite ejecutar scripts desde la terminal: `php fichero.php`.
*   **php-common:** Bibliotecas comunes utilizadas por PHP.
*   **libapache2-mod-php:** El módulo `mod_php` que permite a Apache procesar archivos PHP.

Para verificar la instalación, usamos `php -v` para ver la versión o `sudo apache2ctl -M | grep php` para confirmar que el `php_module` está cargado.

### Configuración y ubicación de archivos
El archivo principal de configuración es el **php.ini**. En Debian se encuentra en una ruta similar a `/etc/php/8.x/apache2/php.ini`. Desde aquí configuramos el límite de memoria, la zona horaria, el tamaño máximo de subida de archivos y la visualización de errores.

Los archivos PHP se almacenan en el DocumentRoot de Apache, habitualmente en `/var/www/html`, con nombres como `index.php` o `login.php`.


## 3.3. Extensiones y Arquitecturas de Ejecución

PHP se amplía mediante extensiones para añadir funcionalidades específicas. Las más habituales son:
*   **php-mysql:** Para conectar con MariaDB/MySQL: `sudo apt install php-mysql`.
*   **php-gd:** Para trabajar con imágenes: `sudo apt install php-gd`.
*   **php-xml:** Para procesar documentos XML: `sudo apt install php-xml`.
*   **php-curl:** Para conexiones a servicios externos: `sudo apt install php-curl`.

### Modos de ejecución: mod_php frente a PHP-FPM
Existen dos formas de integrar Apache y PHP:
1.  **mod_php:** PHP está integrado directamente en Apache. Es la opción más sencilla y la que usaremos en las primeras prácticas.
2.  **PHP-FPM (FastCGI Process Manager):** Se instala con `sudo apt install php-fpm`. PHP corre como un servicio independiente. Es la opción profesional por su mejor rendimiento y escalabilidad.


## 3.4. El Sistema Gestor de Bases de Datos: MariaDB

MariaDB es el componente del stack LAMP encargado del almacenamiento permanente de datos. Se instala con:
```bash
sudo apt install mariadb-server
```
Se gestiona mediante `systemctl` (`start`, `stop`, `restart`, `status`) y se comprueba la versión con `mariadb --version`.

### Seguridad Inicial: mysql_secure_installation

Para asegurar el servidor de base de datos, es obligatorio ejecutar el script de seguridad. El objetivo no es configurar una cuenta para el uso diario, sino **blindar el acceso administrativo** y eliminar vulnerabilidades por defecto:

```bash
sudo mysql_secure_installation
```

Este asistente permite:

- **Establecer una contraseña para root:** Para evitar que cualquier usuario con acceso al sistema pueda entrar fácilmente al SGBD.
- **Eliminar usuarios anónimos:** Para que nadie pueda entrar sin identificarse.
- **Deshabilitar el acceso remoto de root:** Para que la cuenta administrativa solo sea accesible desde el propio servidor (localhost).
- **Borrar la base de datos de prueba:** Para eliminar datos innecesarios que podrían dar pistas sobre la estructura del sistema.

**Importante:** Una vez ejecutado este script, la cuenta de `root` queda reservada únicamente para tareas de mantenimiento crítico. Para el funcionamiento de cualquier aplicación web, es obligatorio crear un usuario específico con privilegios limitados (ver apartado 3.5)


## 3.5. Usuarios, Privilegios y Herramientas Gráficas

### Gestión de Usuarios y Seguridad
Nunca debemos usar el usuario `root` en las aplicaciones. Creamos usuarios específicos:
1. **Crear usuario:** `CREATE USER 'appweb'@'localhost' IDENTIFIED BY 'contraseña_segura';`
2. **Asignar permisos:** `GRANT ALL PRIVILEGES ON tienda.* TO 'appweb'@'localhost';`
3. **Aplicar cambios:** `FLUSH PRIVILEGES;`
4. **Consultar:** `SHOW GRANTS FOR 'appweb'@'localhost';`
5. **Eliminar:** `DROP USER 'appweb'@'localhost';`

### Administración Visual con phpMyAdmin
Se instala con `sudo apt install phpmyadmin`. Durante la instalación, seleccionamos `apache2` y configuramos `dbconfig-common`.
Si no se activa automáticamente, usamos:
```bash
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin
sudo systemctl reload apache2
```
Acceso: `http://localhost/phpmyadmin`.


## 3.6. Integración Final: Conectando PHP con MariaDB (Estilo Procedimental)

Para conectar PHP con MariaDB, instalamos `php-mysql` y reiniciamos Apache. En este módulo utilizaremos el **acceso no basado en objetos (procedimental)**, utilizando la extensión `mysqli`.

### Conexión a la Base de Datos
Para abrir una conexión, utilizamos la función `mysqli_connect()`.

```php
<?php
$servidor = "localhost";
$usuario  = "appweb";
$password = "contraseña_segura";
$bd       = "tienda";

// Establecer la conexión
$conexion = mysqli_connect($servidor, $usuario, $password, $bd);

// Comprobar si la conexión ha fallado
if (!$conexion) {
    die("Error de conexión: " . mysqli_connect_error());
}

echo "Conexión establecida correctamente.";

// Cerrar la conexión al final
mysqli_close($conexion);
?>
```

### Consulta de Datos
Para obtener datos, utilizamos `mysqli_query()` para ejecutar la sentencia SQL y `mysqli_fetch_assoc()` para recorrer los resultados fila por fila.

```php
<?php
$conexion = mysqli_connect("localhost", "appweb", "contraseña_segura", "tienda");

$sql = "SELECT nombre, precio FROM productos";
$resultado = mysqli_query($conexion, $sql);

// Recorrer los resultados
while ($fila = mysqli_fetch_assoc($resultado)) {
    echo "<p>" . $fila['nombre'] . " - " . $fila['precio'] . " €</p>";
}

mysqli_close($conexion);
?>
```


## 3.7. Seguridad y Sentencias Preparadas (Estilo Procedimental)

Para evitar la **Inyección SQL**, nunca concatenamos variables directamente en la consulta. Usamos sentencias preparadas, que separan el comando SQL de los datos.

**Ejemplo de consulta segura con MySQLi procedimental:**
```php
<?php
$conexion = mysqli_connect("localhost", "appweb", "contraseña_segura", "tienda");

// 1. Preparar la sentencia con un marcador '?'
$stmt = mysqli_prepare($conexion, "SELECT nombre, precio FROM productos WHERE nombre = ?");

// 2. Vincular el parámetro ('s' indica que el dato es un string)
$nombreBuscado = "Teclado";
mysqli_stmt_bind_param($stmt, "s", $nombreBuscado);

// 3. Ejecutar la sentencia
mysqli_stmt_execute($stmt);

// 4. Obtener el resultado
$resultado = mysqli_stmt_get_result($stmt);

while ($fila = mysqli_fetch_assoc($resultado)) {
    echo "<p>" . $fila['nombre'] . " - " . $fila['precio'] . " €</p>";
}

mysqli_stmt_close($stmt);
mysqli_close($conexion);
?>
```


## 3.8. Metodología de Pruebas y Diagnóstico

### Pruebas Sistemáticas
1.  **`phpinfo()`:** Crear `/var/www/html/info.php` con `<?php phpinfo(); ?>`. Confirma la instalación y extensiones.
2.  **Salida de texto:** Crear `prueba.php` con `echo "PHP funciona correctamente";`.
3.  **Contenido Dinámico:** Usar `echo date("d/m/Y H:i:s");` para comprobar la ejecución en el servidor.
4.  **Operaciones:** Probar variables y sumas: `$a=10; $b=20; echo $a+$b;`.
5.  **Entorno:** Usar `$_SERVER['SERVER_NAME']` para ver datos de Apache.
6.  **Consola:** Ejecutar `php prueba.php` desde la terminal.

### Diagnóstico de Errores
*   **Código PHP visible:** El módulo `mod_php` no está cargado. Revisar `sudo apache2ctl -M | grep php`.
*   **Página en blanco / Error 500:** Revisar `sudo tail /var/log/apache2/error.log`.
*   **MariaDB no arranca:** Revisar `sudo systemctl status mariadb` y `sudo journalctl -u mariadb`.
*   **Acceso denegado:** Revisar privilegios con `SHOW GRANTS FOR 'usuario'@'localhost';`.
*   **Puerto 3306 ocupado:** Revisar con `sudo ss -tulpn | grep :3306`.


## 3.9. Prueba de Integración Completa (Full Stack)

Para validar todo el sistema:
1.  **SQL:** Crear BD `prueba_lamp`, tabla `mensajes` e insertar un registro.
2.  **Usuario:** Crear `lampuser` con privilegios sobre `prueba_lamp`.
3.  **PHP (Procedimental):** Crear `/var/www/html/prueba_lamp.php`:
```php
<?php
$conexion = mysqli_connect("localhost", "lampuser", "lamp1234", "prueba_lamp");

if (!$conexion) {
    die("Error de conexión: " . mysqli_connect_error());
}

$resultado = mysqli_query($conexion, "SELECT texto FROM mensajes");

while ($fila = mysqli_fetch_assoc($resultado)) {
    echo "<h1>" . htmlspecialchars($fila['texto']) . "</h1>";
}

mysqli_close($conexion);
?>
```
Si el mensaje aparece en el navegador, el flujo **Cliente $\rightarrow$ Apache $\rightarrow$ PHP $\rightarrow$ MariaDB** es correcto.