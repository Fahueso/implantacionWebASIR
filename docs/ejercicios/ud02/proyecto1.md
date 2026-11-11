# Práctica Integrada PHP
## Calculadora con Login, Sesiones e Historial

### Objetivos

En esta práctica se trabajarán los siguientes conceptos:

- Sintaxis básica de PHP.
- Variables.
- Formularios HTML.
- Envío de datos mediante `POST`.
- Estructuras de control (`if`, `switch`, `foreach`).
- Funciones.
- Arrays.
- Sesiones.
- Redirecciones.
- Organización de código en varios ficheros.

---

# Estructura del proyecto

El proyecto deberá tener la siguiente estructura:

```text
/
│
├── login.php
├── calculadora.php
├── funciones.php
└── logout.php
```

---

# Parte 1. Inicio de sesión

Crear una página llamada `login.php`.

La página mostrará un formulario con los siguientes campos:

- Usuario
- Contraseña
- Botón de acceso

Las credenciales válidas serán:

```text
Usuario: admin
Contraseña: 1234
```

Cuando el usuario introduzca los datos correctamente:

- Se iniciará una sesión.
- Se almacenará el nombre del usuario en la sesión.
- Se almacenará la hora de acceso en la sesión.
- Se redirigirá al usuario a la página `calculadora.php`.

Si las credenciales son incorrectas se mostrará un mensaje de error.

---

# Parte 2. Control de acceso

La página `calculadora.php` solo podrá ser utilizada por usuarios autenticados.

Si un usuario intenta acceder directamente a la URL sin haber iniciado sesión, deberá ser redirigido automáticamente a `login.php`.

---

# Parte 3. Calculadora

Crear una calculadora que permita:

- Introducir dos números.
- Seleccionar una operación.
- Obtener el resultado.

Las operaciones disponibles serán:

- Suma (`+`)
- Resta (`-`)
- Multiplicación (`*`)
- División (`/`)

## Ejemplo

```text
Número 1: 8
Número 2: 2
Operación: +

Resultado: 10
```

---

# Parte 4. Funciones

Todas las operaciones matemáticas deberán realizarse mediante una función definida en el fichero `funciones.php`.

La función deberá tener el siguiente prototipo:

```php
function calcular(float $a, float $b, string $op): float
```

La operación deberá resolverse utilizando una estructura `switch`.

---

# Parte 5. Historial de operaciones

Cada vez que se realice una operación, esta deberá almacenarse en un historial asociado a la sesión del usuario.

## Ejemplo

```text
8 + 2 = 10
5 * 3 = 15
9 - 4 = 5
```

El historial deberá mostrarse debajo de la calculadora.

---

# Parte 6. Estadísticas

Mostrar el número total de operaciones realizadas durante la sesión.

## Ejemplo

```text
Operaciones realizadas: 3
```

---

# Parte 7. Información de la sesión

En la parte superior de la página deberá mostrarse:

```text
Usuario conectado: admin

Hora de acceso: 10:35:24
```

---

# Parte 8. Borrar historial

Añadir un botón que permita vaciar completamente el historial de operaciones almacenado en la sesión.

## Ejemplo

Antes:

```text
8 + 2 = 10
5 * 3 = 15
9 - 4 = 5
```

Después:

```text
Historial vacío
```

---

# Parte 9. Cerrar sesión

Crear una página `logout.php` que permita:

- Cerrar la sesión actual.
- Eliminar toda la información almacenada en ella.
- Volver automáticamente a `login.php`.

---

# Snippets de ayuda

## Iniciar una sesión

```php
session_start();
```

---

## Guardar datos en una sesión

```php
$_SESSION["usuario"] = "admin";
```

---

## Comprobar si existe una variable

```php
isset($variable)
```

---

## Redirigir a otra página

```php
header("Location: login.php");
exit;
```

---

## Obtener la hora actual

```php
date("H:i:s")
```

---

## Formulario HTML

```html
<form method="post">

</form>
```

---

## Prototipo obligatorio

```php
function calcular(float $a, float $b, string $op): float
```

---

# Entregables

Se deberá entregar:

- Código fuente completo.
- Captura de pantalla del login.
- Captura de pantalla de la calculadora funcionando.
- Captura del historial con varias operaciones.
- Captura mostrando el cierre de sesión.

---

# Ampliación (Opcional)

Implementar una o varias de las siguientes mejoras:

- Operación de potencia.
- Operación de raíz cuadrada.
- Última operación realizada.
- Contador de accesos durante la sesión.
- Tema claro/oscuro almacenado en sesión.
- Mostrar fecha y hora completas del acceso.

Estas mejoras no son obligatorias, pero servirán para subir nota.