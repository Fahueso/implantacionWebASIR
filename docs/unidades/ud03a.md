# 1. Introducción a las Aplicaciones Dinámicas

## 1.1 El problema de la persistencia de los datos

Durante las unidades anteriores hemos aprendido a crear páginas web dinámicas utilizando PHP. Gracias al lenguaje hemos podido trabajar con variables, estructuras de control, funciones, formularios y sesiones que permiten generar contenido diferente para cada usuario.

Por ejemplo:

```php
<?php

$nombre = "Francisco";

echo "Bienvenido " . $nombre;

?>
```

Cuando el navegador solicita la página, PHP ejecuta el código y genera:

```html
Bienvenido Francisco
```

Al finalizar la ejecución, las variables normales desaparecen y dejan de estar disponibles.

Para conservar información entre distintas páginas podemos utilizar sesiones.

Por ejemplo:

```php
<?php

session_start();

$_SESSION["usuario"] = "Francisco";

?>
```

Posteriormente, otra página podrá recuperar ese valor:

```php
<?php

session_start();

echo $_SESSION["usuario"];

?>
```

Las sesiones permiten mantener información asociada a un usuario mientras dure su visita al sitio web, pero no están diseñadas para almacenar datos de forma permanente.

Por ejemplo, una sesión puede utilizarse para recordar:

- El usuario que ha iniciado sesión.
- Su nombre.
- Sus permisos.
- El contenido de un carrito de compra.

Sin embargo, si el servidor se reinicia, la sesión caduca o el usuario borra sus datos de navegación, esa información puede perderse.

Las aplicaciones reales necesitan almacenar datos durante días, meses o incluso años.

Por ejemplo:

- Clientes de una empresa.
- Productos de una tienda online.
- Reservas de un hotel.
- Calificaciones de estudiantes.
- Historial de pedidos.

Para conservar esta información de manera permanente necesitaremos utilizar bases de datos.

A esta capacidad de conservar información más allá de la ejecución de una página o de la duración de una sesión la denominamos **persistencia de datos**.

## 1.2 La necesidad de almacenar información

La mayoría de aplicaciones necesitan conservar información aunque el programa deje de ejecutarse.

Por ejemplo, una tienda online debe almacenar:

- Clientes.
- Direcciones de envío.
- Productos adquiridos.
- Pedidos realizados.
- Estado de los envíos.

Del mismo modo, una plataforma educativa debe almacenar usuarios, cursos, actividades y calificaciones; mientras que una red social necesita conservar usuarios, publicaciones, comentarios y mensajes.

En todos estos casos es necesario disponer de un mecanismo que permita conservar la información incluso cuando el servidor se apaga o la aplicación finaliza.

A esta capacidad la denominamos **persistencia de datos**.

> La persistencia es la capacidad de almacenar información de forma permanente para poder recuperarla posteriormente.

## 1.3 ¿Qué es una base de datos?

Una base de datos es un sistema diseñado para almacenar y organizar información de forma estructurada.

Gracias a una base de datos podemos:

- Guardar información.
- Consultar información.
- Modificar información.
- Eliminar información.
- Compartir información entre múltiples usuarios.

Las bases de datos modernas pueden gestionar miles o millones de registros de forma rápida y segura.

## 1.4 Bases de datos relacionales

Existen diferentes tipos de bases de datos, pero las más utilizadas en aplicaciones web son las **bases de datos relacionales**.

Una base de datos relacional organiza la información en tablas.

Por ejemplo, una empresa podría disponer de una tabla llamada `clientes`.

| id | nombre | telefono |
|----|---------|----------|
| 1 | Ana | 600111111 |
| 2 | Pedro | 600222222 |
| 3 | María | 600333333 |

En una tabla:

- Cada fila representa un registro.
- Cada columna representa una característica del registro.
- Cada fila posee un identificador único.

Las tablas recuerdan a una hoja de cálculo, pero están optimizadas para almacenar y consultar grandes cantidades de información.

## 1.5 Sistemas Gestores de Bases de Datos (SGBD)

Las bases de datos necesitan un software especializado para gestionarlas.

Este software recibe el nombre de **Sistema Gestor de Bases de Datos (SGBD)**.

Un SGBD permite:

- Crear bases de datos.
- Crear tablas.
- Gestionar usuarios.
- Ejecutar consultas.
- Garantizar la integridad de la información.

Algunos ejemplos de SGBD son:

- MariaDB.
- MySQL.
- PostgreSQL.
- Oracle Database.
- Microsoft SQL Server.

### MariaDB y MySQL

MariaDB nació como una bifurcación (*fork*) de MySQL, por lo que ambos sistemas comparten gran parte de su funcionamiento:

- Utilizan prácticamente el mismo lenguaje SQL.
- La mayoría de consultas son compatibles.
- PHP puede conectarse a ambos utilizando las mismas funciones.

En este curso utilizaremos principalmente **MariaDB**, aunque la mayoría de ejemplos funcionarían igualmente en MySQL.

## 1.6 Operaciones básicas sobre datos: CRUD

Cuando trabajamos con una base de datos realizamos cuatro operaciones fundamentales conocidas como **CRUD**.

| Operación | Descripción | Ejemplo |
|------------|------------|----------|
| Create | Crear registros | Insertar un nuevo cliente |
| Read | Consultar información | Mostrar todos los clientes |
| Update | Modificar registros | Cambiar el teléfono de un cliente |
| Delete | Eliminar registros | Borrar un cliente |

Estas cuatro operaciones forman la base de la mayoría de aplicaciones empresariales.

## 1.7 Comunicación entre PHP y MariaDB

PHP actúa como intermediario entre el navegador y la base de datos.

El usuario nunca accede directamente al servidor MariaDB. En su lugar, interactúa con una página web.

El proceso general es el siguiente:

```text
Usuario
    │
    ▼
Navegador
    │
    ▼
Servidor Web (Apache)
    │
    ▼
PHP
    │
    ▼
MariaDB
```

Por ejemplo:

1. El usuario accede a `clientes.php`.
2. El servidor ejecuta el código PHP.
3. PHP solicita información a MariaDB.
4. MariaDB devuelve los datos.
5. PHP genera HTML.
6. El navegador muestra el resultado.

Todo este proceso suele completarse en unas pocas fracciones de segundo.

## 1.8 SQL: el lenguaje de las bases de datos

Para comunicarnos con una base de datos utilizamos **SQL** (*Structured Query Language*).

SQL permite realizar todas las operaciones habituales sobre los datos.

### Crear tablas

```sql
CREATE TABLE clientes (
    id INT,
    nombre VARCHAR(100)
);
```

### Insertar datos

```sql
INSERT INTO clientes
VALUES (1, 'Ana');
```

### Consultar información

```sql
SELECT *
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

A lo largo de esta unidad aprenderemos a combinar SQL y PHP para desarrollar aplicaciones web capaces de almacenar información de manera permanente.