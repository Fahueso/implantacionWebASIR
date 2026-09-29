!!! info "Criterios de Evaluación (RA5)"
    En este apartado se trabajan los siguientes puntos del Resultado de Aprendizaje:

    - **e)** Se han definido y utilizado funciones.

# 1. Funciones

Realiza un fichero en PHP que incluya las siguientes funciones. Se facilita el prototipo de las mismas y el tipo de retorno:

* Función que devuelve TRUE o FALSE en función de si un número es par o impar
```php
function esPar(int $n): bool
```
* Función que devuelve el cuadrado de un número entero
```php
function cuadrado(int $n): int
```
* Función que devuelve el máximo de dos números enteros
```php
function max2(int $a, int $b): string
```
* Función que imprime un menú de opciones. La opción se elige de manera automática en el rango 0-3, mediante números aleatorios. La opción 0 es la opción de salida del programa. El resto de funciones, desde la 1 a la 3, corresponden a las funciones anteriores.  

```php
function menu(): int
```
* Realiza el cuerpo del programa en PHP para que imprima de manera indefinida el menú, hasta que se decida por la opción de salir. En caso contrario ejecutará uno de las funciones anteriores. Los parámetros serán números aleatorios entre 1 y 100.


# 2. Entregables y Evaluación
Sube a aules una carpeta comprimida con el código fuente, captura de consola y captura del navegador