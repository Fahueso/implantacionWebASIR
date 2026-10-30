# 1. Introducción a las Aplicaciones Dinámicas

## 1.1 El problema de la persistencia de los datos

Durante las unidades anteriores hemos aprendido a crear páginas web dinámicas con PHP: variables, estructuras de control, funciones, formularios y sesiones.

Sin embargo, todo lo que hemos hecho hasta ahora tiene una limitación: **los datos desaparecen**.

```php
<?php

$nombre = "Francisco";

echo "Bienvenido " . $nombre;

?>
```

Cuando el navegador solicita la página, PHP ejecuta el código y genera:

```text
Bienvenido Francisco
```

Al terminar la ejecución del script, las variables desaparecen. Para conservar información **entre distintas páginas** podemos usar sesiones:

```php
<?php
// pagina1.php
session_start();
$_SESSION["usuario"] = "Francisco";
?>
```

```php
<?php
// pagina2.php
session_start();
echo $_SESSION["usuario"];
?>
```

Las sesiones mantienen información asociada a un usuario **mientras dura su visita**: quién ha iniciado sesión, sus permisos o el contenido de su carrito. Pero no están pensadas para guardar datos de forma permanente: la sesión caduca, el servidor puede reiniciarse o el usuario puede borrar las cookies.

Las aplicaciones reales necesitan guardar datos durante días, meses o años:

- Clientes de una empresa.
- Productos de una tienda online.
- Reservas de un hotel.
- Calificaciones de estudiantes.
- Historial de pedidos.

## 1.2 Persistencia de datos

> **Persistencia** es la capacidad de almacenar información de forma permanente para poder recuperarla posteriormente, incluso cuando el programa termina o el servidor se apaga.

Para conseguirla utilizaremos **bases de datos**.

| Mecanismo | Duración de los datos | Ejemplo de uso |
|-----------|-----------------------|----------------|
| Variables PHP | Hasta que termina el script | Cálculos intermedios |
| Sesiones | Mientras dura la visita del usuario | Usuario autenticado, carrito |
| Base de datos | Permanente | Clientes, pedidos, notas |

## 1.3 ¿Qué es una base de datos?

Una base de datos es un sistema diseñado para almacenar y organizar información de forma estructurada.

Gracias a una base de datos podemos:

- Guardar información.
- Consultarla.
- Modificarla.
- Eliminarla.
- Compartirla entre múltiples usuarios a la vez.

Las bases de datos modernas gestionan millones de registros de forma rápida y segura.

## 1.4 Bases de datos relacionales

Las más utilizadas en aplicaciones web son las **bases de datos relacionales**, que organizan la información en **tablas**.

Por ejemplo, una tabla `clientes`:

| id | nombre | telefono |
|----|--------|----------|
| 1 | Ana | 600111111 |
| 2 | Pedro | 600222222 |
| 3 | María | 600333333 |

En una tabla:

- Cada **fila** representa un registro.
- Cada **columna** representa una característica del registro.
- Cada fila posee un **identificador único** (clave primaria).

Recuerda a una hoja de cálculo, pero está optimizada para almacenar y consultar enormes cantidades de datos, y permite relacionar unas tablas con otras (lo veremos en el capítulo 7).

## 1.5 Sistemas Gestores de Bases de Datos (SGBD)

Las bases de datos necesitan un software que las gestione: el **Sistema Gestor de Bases de Datos (SGBD)**. Un SGBD permite:

- Crear bases de datos y tablas.
- Gestionar usuarios y permisos.
- Ejecutar consultas.
- Garantizar la integridad de la información.

Ejemplos: MariaDB, MySQL, PostgreSQL, Oracle Database y Microsoft SQL Server.

### MariaDB y MySQL

MariaDB nació como una bifurcación (*fork*) de MySQL, por lo que comparten gran parte de su funcionamiento:

- Usan prácticamente el mismo lenguaje SQL.
- La mayoría de consultas son compatibles.
- PHP se conecta a ambos con las mismas funciones.

En este curso utilizaremos **MariaDB**, aunque casi todos los ejemplos funcionarían igual en MySQL.

> **Nota:** PHP ofrece dos formas principales de acceder a bases de datos: la extensión `mysqli` (específica de MySQL/MariaDB) y **PDO** (genérica, válida para varios SGBD). En este curso usaremos `mysqli` en su estilo **procedural**. Cuando trabajes con código de terceros es probable que encuentres también `mysqli` orientado a objetos (`$conexion->query(...)`) o PDO: los conceptos son los mismos y solo cambia la sintaxis.

## 1.6 Operaciones básicas sobre datos: CRUD

Con una base de datos realizamos cuatro operaciones fundamentales, conocidas como **CRUD**:

| Operación | Descripción | Sentencia SQL | Ejemplo |
|-----------|-------------|---------------|---------|
| **C**reate | Crear registros | `INSERT` | Insertar un nuevo cliente |
| **R**ead | Consultar información | `SELECT` | Mostrar todos los clientes |
| **U**pdate | Modificar registros | `UPDATE` | Cambiar el teléfono de un cliente |
| **D**elete | Eliminar registros | `DELETE` | Borrar un cliente |

Estas cuatro operaciones son la base de la mayoría de aplicaciones empresariales. Las estudiaremos en los capítulos 3, 5 y 6.

## 1.7 Comunicación entre PHP y MariaDB

PHP actúa como intermediario entre el navegador y la base de datos. **El usuario nunca accede directamente a MariaDB**: interactúa con una página web.

```text
Usuario
   │
   ▼
Navegador
   │
   ▼
Servidor web (Apache)
   │
   ▼
PHP
   │
   ▼
MariaDB
```

Por ejemplo:

1. El usuario accede a `clientes.php`.
2. Apache ejecuta el código PHP.
3. PHP solicita información a MariaDB.
4. MariaDB devuelve los datos.
5. PHP genera el HTML.
6. El navegador muestra el resultado.

Todo este proceso suele completarse en fracciones de segundo.

## 1.8 SQL: el lenguaje de las bases de datos

Para comunicarnos con una base de datos usamos **SQL** (*Structured Query Language*).

### Crear tablas

```sql
CREATE TABLE clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    telefono VARCHAR(20)
);
```

### Insertar datos

```sql
INSERT INTO clientes (nombre, telefono)
VALUES ('Ana', '600111111');
```

### Consultar información

```sql
SELECT id, nombre, telefono
FROM clientes;
```

### Modificar registros

```sql
UPDATE clientes
SET nombre = 'Ana López'
WHERE id = 1;
```

### Eliminar registros

```sql
DELETE FROM clientes
WHERE id = 1;
```

> **Buenas prácticas desde el principio**
> - En los `INSERT`, indica siempre las columnas (`INSERT INTO clientes (nombre, telefono) VALUES ...`). Si algún día cambia el orden o el número de columnas de la tabla, tu código no se romperá.
> - En los `SELECT`, evita `SELECT *` en aplicaciones reales y escribe las columnas que necesitas. En los ejemplos de estos apuntes usaremos `SELECT *` solo cuando sea para simplificar.

## 1.9 Preparar el entorno de trabajo

Para seguir los ejemplos necesitas:

- Apache, PHP (8.1 o superior) y MariaDB funcionando.
- La extensión `mysqli` activada (viene activada en la mayoría de instalaciones).
- La base de datos `empresa` con sus tablas y datos de ejemplo.

En el **Anexo A** tienes un script SQL que crea todo lo necesario. Ejecútalo antes de empezar el capítulo 2:

```bash
sudo mariadb < anexo_a_empresa.sql
```

Para comprobar que `mysqli` está disponible, crea un fichero `info.php` con `<?php phpinfo(); ?>` y busca la sección *mysqli* (bórralo después: no debe quedar en un servidor real).

A lo largo de la unidad combinaremos SQL y PHP para construir aplicaciones capaces de almacenar información de manera permanente.

---

## Resumen

- Las variables y las sesiones no son suficientes para guardar datos de forma permanente.
- La **persistencia** se consigue con una **base de datos**.
- Las bases de datos relacionales organizan los datos en **tablas** de filas y columnas.
- Un **SGBD** (en nuestro caso MariaDB) gestiona las bases de datos.
- **CRUD** = Create, Read, Update, Delete = `INSERT`, `SELECT`, `UPDATE`, `DELETE`.
- PHP es el intermediario entre el navegador y MariaDB, y se comunica con ella mediante **SQL**.

## Actividades

1. Indica qué mecanismo (variable, sesión o base de datos) usarías para guardar: a) el nombre del usuario conectado, b) las notas de los alumnos, c) el resultado de una suma, d) los productos del carrito antes de pagar, e) los pedidos ya pagados.
2. Escribe las sentencias SQL para: crear una tabla `productos` (id, nombre, precio), insertar dos productos, listarlos, subir 1 € el precio de uno de ellos y borrar el otro.
3. Dibuja el recorrido completo de una petición cuando un usuario abre `clientes.php`, indicando qué hace cada componente.
