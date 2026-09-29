!!! info "Criterios de Evaluación (RA5)"
    En este apartado se trabajan los siguientes puntos del Resultado de Aprendizaje:

    - **g)** SSe ha identificado y asegurado a los usuarios que acceden al documento Web.
    - **h)** Se ha verificado el aislamiento del entorno específico de cada usuario.

# 1 Sesiones
Realiza un fichero PHP que muestre la primera y la última hora en la que se accedió al fichero en la sesión actual.
Además incorpora un botón llamado reiniciar que limpie la sesión, que reaccione por POST.

NOTA: puedes obtener la hora actual con el siguiente snippet
```php
date("Y-m-d H:i:s");
```
# 2 Sesiones
Realiza una página en PHP que muestre una página con color de fondo y un botón. El color de fondo no cambiará hasta que se pulse el botón, de manera que el color de fondo siga el siguiente patrón:
Rojo - Verde - Azul - Rojo - Verde, etc

# 3 Autenticación y seguridad
Genera una página en PHP que permita el login de cualquier usuario, introduciendo un nombre. La contraseña correcta de todos los usuarios es 1234. Si existe una autenticación satisfactoria saltaremos siempre a la segunda página. 
Esta página debe generar mensaje ante una autenticación incorrecta

Genera una segunda página en la que aparecerá el nombre del usuario y su número de sesion. Además, dispondrá un botón de desconexión que volvería al programa anterior.

Si no hay usuario autenticado, esta segunda página deberá redirigir a la primera.
La primera página deberá mostrar un mensaje de usuario desconectado tras ser redirigido a ella.



# 4 Autenticación y seguridad

Amplia el programa anterior para tener en cuenta que la sesión debe finalizar tras 10 minutos de inactividad, redirigiendo a la primera página en caso de que la petición caiga sobre la segunda página.
En cualquier caso la primera página mostrará un mensaje informando si se ha dado una expiración de sesión.

# 5. Entregables y Evaluación
Sube a aules una carpeta comprimida con el código fuente ycaptura del navegador