# 8. Gestión de Credenciales y Configuración

## 8.1 Introducción

Durante toda la unidad hemos utilizado ejemplos similares al siguiente para conectarnos a MariaDB:

```php
$conexion = mysqli_connect(
    "localhost",
    "usuario",
    "password",
    "empresa"
);
```

Este código funciona correctamente y resulta adecuado para aprender cómo se realiza una conexión. Sin embargo, presenta un problema importante: las credenciales se encuentran escritas directamente dentro del código fuente.

En aplicaciones reales esta práctica debe evitarse siempre que sea posible.

Las credenciales forman parte de la configuración de la aplicación y deberían mantenerse separadas de la lógica del programa.

---

## 8.2 ¿Por qué no debemos guardar credenciales en el código?

Supongamos la siguiente conexión:

```php
$conexion = mysqli_connect(
    "localhost",
    "admin",
    "123456",
    "empresa"
);
```

Este enfoque presenta varios inconvenientes.

### Exposición accidental

Si el código fuente se comparte o publica accidentalmente, las credenciales quedarán expuestas.

### Mantenimiento complejo

Si la contraseña cambia, será necesario modificar todos los archivos donde aparezca la conexión.

### Múltiples entornos

Una aplicación suele pasar por diferentes fases:

- Desarrollo.
- Pruebas.
- Producción.

Cada entorno suele utilizar configuraciones distintas. Si las credenciales están escritas directamente en el código, será necesario modificar los archivos cada vez que se despliegue la aplicación.

Por este motivo es recomendable separar claramente:

### Código

Contiene la lógica de la aplicación.

```php
echo "Listado de clientes";
```

### Configuración

Contiene valores que pueden variar entre instalaciones:

- Servidor de base de datos.
- Usuario.
- Contraseña.
- Nombre de la base de datos.
- Puerto de conexión.

Las credenciales pertenecen a la configuración, no al código.

---

## 8.3 Variables de entorno

Una de las técnicas más utilizadas para gestionar configuraciones consiste en emplear variables de entorno.

Las variables de entorno son valores definidos por el sistema operativo o por el servidor web que pueden recuperarse posteriormente desde la aplicación.

Por ejemplo:

```text
DB_HOST=localhost

DB_NAME=empresa

DB_USER=usuario

DB_PASS=password
```

La principal ventaja es que estos valores no aparecen directamente en el código fuente.

---

## 8.4 Configuración de Apache

Apache permite definir variables de entorno accesibles desde PHP.

Una forma cómoda de hacerlo consiste en crear un archivo independiente de configuración.

Por ejemplo:

```text
/etc/apache2/conf-available/database.conf
```

Podemos editarlo mediante:

```bash
sudo nano /etc/apache2/conf-available/database.conf
```

y añadir:

```apache
SetEnv DB_HOST localhost

SetEnv DB_NAME empresa

SetEnv DB_USER usuario

SetEnv DB_PASS password
```

Cada instrucción `SetEnv` crea una variable de entorno disponible para las aplicaciones PHP ejecutadas por Apache.

---

## 8.5 Activación de la configuración

Una vez creado el archivo, debemos indicar a Apache que queremos utilizarlo.

```bash
sudo a2enconf database
```

La salida será similar a:

```text
Enabling conf database.
```

Posteriormente reiniciamos Apache:

```bash
sudo systemctl restart apache2
```

A partir de ese momento las variables estarán disponibles para todas las aplicaciones PHP ejecutadas por el servidor.

---

## 8.6 Acceso a las variables desde PHP

PHP permite recuperar estas variables mediante:

```php
apache_getenv()
```

Por ejemplo:

```php
$db_host = apache_getenv(
    "DB_HOST"
);

$db_name = apache_getenv(
    "DB_NAME"
);

$db_user = apache_getenv(
    "DB_USER"
);

$db_pass = apache_getenv(
    "DB_PASS"
);
```

Cada variable contendrá el valor definido en la configuración de Apache.

Para verificar su funcionamiento podemos utilizar un pequeño script de prueba:

```php
<?php

echo apache_getenv(
    "DB_HOST"
);

?>
```

Si la configuración es correcta, el navegador mostrará:

```text
localhost
```

---

## 8.7 Conexión utilizando variables de entorno

Una vez recuperados los valores, podemos utilizarlos para crear la conexión.

```php
<?php

$db_host = apache_getenv(
    "DB_HOST"
);

$db_name = apache_getenv(
    "DB_NAME"
);

$db_user = apache_getenv(
    "DB_USER"
);

$db_pass = apache_getenv(
    "DB_PASS"
);

$conexion = mysqli_connect(

    $db_host,

    $db_user,

    $db_pass,

    $db_name

);

if (!$conexion) {

    die(
        "Error de conexión: "
        . mysqli_connect_error()
    );

}

?>
```

Observa que ya no aparecen valores sensibles escritos directamente dentro del código.

---

## 8.8 Centralización de la conexión

Durante toda la unidad hemos utilizado un archivo llamado:

```text
conexion.php
```

Este archivo constituye un punto central desde el que toda la aplicación accede a la base de datos.

Por ejemplo:

```php
<?php

function conexion()
{

    $db_host = apache_getenv(
        "DB_HOST"
    );

    $db_name = apache_getenv(
        "DB_NAME"
    );

    $db_user = apache_getenv(
        "DB_USER"
    );

    $db_pass = apache_getenv(
        "DB_PASS"
    );

    $conexion = mysqli_connect(

        $db_host,

        $db_user,

        $db_pass,

        $db_name

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

Gracias a este enfoque, cualquier cambio futuro en la configuración deberá realizarse únicamente en un único lugar.

---

## 8.9 Organización recomendada del proyecto

Una estructura habitual para aplicaciones PHP podría ser:

```text
proyecto/

├── config/
│   └── conexion.php
│
├── public/
│   ├── index.php
│   ├── clientes.php
│   ├── insertar.php
│   ├── actualizar.php
│   └── eliminar.php
│
└── recursos/
```

En esta organización:

- `config/` contiene la configuración.
- `public/` contiene las páginas accesibles por el usuario.
- `recursos/` almacena archivos auxiliares.

Esta separación mejora la claridad y facilita el mantenimiento.

---

## 8.10 Seguridad y repositorios Git

Una situación muy habitual consiste en almacenar el proyecto en un repositorio Git.

Si las credenciales aparecen dentro de los archivos fuente y el repositorio se publica accidentalmente, cualquier persona tendrá acceso a ellas.

Por este motivo es recomendable:

- Separar configuración y código.
- No almacenar contraseñas en el repositorio.
- Excluir archivos sensibles mediante `.gitignore`.
- Utilizar usuarios específicos para cada aplicación.

Otro principio fundamental es el **principio de mínimo privilegio**.

Una aplicación web no debería conectarse utilizando el usuario administrador de MariaDB.

Lo recomendable es crear un usuario específico:

```sql
CREATE USER
'app_web'
IDENTIFIED BY 'password_seguro';
```

y conceder únicamente los permisos necesarios:

```sql
GRANT

    SELECT,
    INSERT,
    UPDATE,
    DELETE

ON empresa.*

TO 'app_web';
```

Este enfoque limita los posibles daños producidos por errores de programación o vulnerabilidades de seguridad.

---

## 8.11 Resumen

A lo largo de este capítulo hemos aprendido que:

- Las credenciales no deberían almacenarse directamente en el código.
- La configuración debe mantenerse separada de la lógica de la aplicación.
- Las variables de entorno permiten almacenar información sensible de forma más segura.
- Apache puede proporcionar dichas variables a PHP mediante `apache_getenv()`.
- La conexión debe centralizarse en un único archivo reutilizable.
- Los repositorios Git no deberían contener contraseñas ni configuraciones sensibles.
- Las aplicaciones deben utilizar únicamente los permisos de base de datos estrictamente necesarios.

Con este capítulo finalizamos los fundamentos necesarios para desarrollar aplicaciones PHP capaces de conectarse a MariaDB, consultar información, insertar registros, modificarlos, eliminarlos y organizar de forma segura la configuración del proyecto.