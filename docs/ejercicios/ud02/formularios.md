!!! info "Criterios de Evaluación (RA5)"
    En este apartado se trabajan los siguientes puntos del Resultado de Aprendizaje:

    - **f)** Se han establecido y utilizado mecanismos para asegurar la persistencia de la información entre distintos documentos Web relacionados.

# 1 Actividad formularios GET
Genera un único fichero PHP que tenga un formulario, que solicite el nombre y un apellido del usuario en sendos inputs. Al recibir ambos parámetros del formulario por GET imprimirá el texto "Bienvenido + $nombre + $apellido".
En esta circunstancia el formulario deberá aparece también relleno con los datos, y el texto del botón del formulario cambiará a modificar. De esta manera el programa permitirá continuamente actualizar el texto de bienvenida.

Utiliza htmlspecialchars para limpiar siempre los datos recibidos del formulario.

NOTA: a este tipo de formularios, que mantienen los datos tras ser enviados, se llaman 'sticky form' 

# 2 Actividad formularios POST
Genera dos archivos PHP. 
* Fichero 1: Formulario de introducción de notas por POST. Incluye el campo nombre y el campo fecha. El envío del formulario carga el fichero 2
* Fichero 2: Muestra el nombre del alumno en cuestión y su nota. Además, imprime los siguientes elementos
    Si la nota < 7 Imprimirá en rojo "No Apto"
    Si la nota >=7 Imprimirá en negro "Apto"
El fichero 2 dispondrá de un enlace para voler al fichero 1, con los elementos del formulario en blanco.

Utiliza htmlspecialchars para limpiar siempre los datos recibidos del formulario. Valida los tipos de datos, ya que la nota ha de ser numérica.

# 2. Entregables y Evaluación
Sube a aules una carpeta comprimida con el código fuente y captura del navegador web corriendo el ejercicio. 