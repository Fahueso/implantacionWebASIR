
# PARTE 2: EL BACKEND - EL MOTOR EN EL SERVIDOR

## 11. El Paradigma del Lado del Servidor

Para comprender la arquitectura de la web moderna, es fundamental distinguir entre los dos entornos donde reside la lógica de una aplicación: el cliente (*Client-side*) y el servidor (*Server-side*). Hasta este punto, el desarrollo se ha centrado en el frontend, donde el código se ejecuta en el navegador del usuario. Sin embargo, para construir aplicaciones que gestionen datos, usuarios y seguridad, es imperativo trasladar la inteligencia al servidor.

### 11.1 La Necesidad del Backend
El entorno del cliente es, por definición, un espacio público y vulnerable. Cualquier código JavaScript, por complejo que sea, es descargado al ordenador del usuario, donde puede ser inspeccionado, modificado o manipulado mediante las herramientas de desarrollo del navegador. Si almacenáramos la lógica de validación de un pago o la contraseña de un administrador en el frontend, estaríamos exponiendo la seguridad de la aplicación.

El servidor, en cambio, es un entorno privado y controlado. Cuando un navegador solicita una página, el servidor no envía el código fuente de la lógica de programación; en su lugar, ejecuta ese código internamente, interactúa con bases de datos y procesa reglas de negocio. El resultado de este proceso es un documento HTML ya renderizado que se envía al cliente. De este modo, el usuario final recibe la información necesaria para visualizar la página, pero el "cerebro" de la aplicación permanece oculto y protegido detrás del cortafuegos del servidor.

### 11.2 Clasificación de los Lenguajes de Servidor: Scripts vs. Compilados

No todos los lenguajes de servidor operan bajo la misma arquitectura. La principal diferencia radica en cómo el ordenador traduce las instrucciones escritas por el programador en acciones ejecutables.

**Los Lenguajes de Script (Interpretados)**
Los lenguajes de script, como PHP, Node.js, Python y Ruby, no requieren una fase de traducción previa. Utilizan un programa llamado **intérprete**, que lee el código fuente línea por línea y lo ejecuta en tiempo real. 
Esta característica es la que los hace ideales para la web: permiten un flujo de trabajo extremadamente ágil. Si un desarrollador detecta un error en una línea de código, puede corregirlo, guardar el archivo y refrescar el navegador para ver el resultado instantáneamente. No hay tiempo de espera entre la escritura y la ejecución.

**Los Lenguajes Compilados**
En el otro extremo encontramos lenguajes como Java o C#. Estos requieren un proceso llamado **compilación**. Antes de que el programa pueda ejecutarse, un compilador analiza todo el código fuente y lo traduce íntegramente a un lenguaje de bajo nivel (binario o *bytecode*) que la máquina entiende directamente. 
Aunque este proceso añade un paso adicional al desarrollo, ofrece dos ventajas críticas: la velocidad de ejecución es significativamente mayor y el compilador detecta la mayoría de los errores de sintaxis y tipos antes de que el programa llegue al usuario. Por ello, son la elección predilecta para sistemas de escala masiva, como la infraestructura de un banco o la gestión de un ERP corporativo.


## 12. Infraestructura y Configuración del Entorno

Para desarrollar aplicaciones backend, es necesario transformar la máquina local en un servidor. En la industria, el sistema operativo estándar es **Linux** (específicamente distribuciones basadas en Ubuntu), ya que la gran mayoría de los servidores en la nube corren sobre este núcleo debido a su eficiencia, seguridad y estabilidad.

### 12.1 Administración de Software mediante APT
En Linux, la gestión de software no se realiza descargando instaladores aislados, sino a través de un gestor de paquetes llamado `apt` (*Advanced Package Tool*). Este sistema conecta el ordenador con repositorios oficiales, asegurando que el software instalado sea seguro y que todas sus dependencias (librerías necesarias para que el programa funcione) se instalen automáticamente.

Para utilizar `apt`, es indispensable el comando `sudo` (*SuperUser Do*). Dado que instalar un servidor afecta a la configuración global del sistema, se requieren privilegios de administrador. El flujo de trabajo comienza siempre con la actualización de los índices del sistema para evitar conflictos de versiones:

```bash
sudo apt update
```

### 12.2 Implementación de los Entornos de Scripting
Dependiendo del lenguaje elegido, la forma de levantar un servidor varía. Para facilitar el desarrollo, se suelen utilizar **microframeworks**, que son bibliotecas ligeras que proporcionan las herramientas básicas para gestionar rutas y peticiones HTTP sin añadir la complejidad de un framework empresarial.

**A. Python y Flask**
Python es valorado por su legibilidad. Para convertirlo en servidor, se utiliza **Flask**. Flask permite definir "rutas", que son básicamente instrucciones que dicen: *"Si el usuario pide la URL /contacto, ejecuta la función de contacto"*.
Instalación: `sudo apt install -y python3 python3-flask`.
Ejecución: `sudo python3 app.py`.

**B. Node.js y Express**
Node.js es una tecnología disruptiva que permite ejecutar JavaScript en el servidor. Para gestionar sus dependencias, utiliza `npm` (*Node Package Manager*), un sistema que descarga librerías externas en una carpeta local del proyecto (`node_modules`).
Instalación: `sudo apt install -y nodejs npm`.
Flujo de trabajo: Primero se inicializa el proyecto con `npm init -y` y luego se instala el framework Express con `npm install express`.
Ejecución: `sudo node app.js`.

**C. Ruby y Sinatra**
Ruby se enfoca en la productividad y la elegancia. **Sinatra** es su microframework de referencia, permitiendo crear aplicaciones web con una cantidad mínima de código.
Instalación: `sudo apt install -y ruby-full ruby-sinatra`.
Ejecución: `sudo ruby app.rb -o 0.0.0.0 -p 80`.

**D. PHP**
PHP fue creado específicamente para la web. Su arquitectura es la más sencilla de todas, ya que el servidor PHP puede procesar archivos `.php` directamente sin necesidad de configurar un servidor de aplicación externo complejo.
Instalación: `sudo apt install -y php`.
Ejecución: `sudo php -S 0.0.0.0:80`.

### 12.3 Gestión de Puertos y el Conflicto con Apache
Un concepto fundamental en redes es el de **puerto**. El puerto es como una puerta numerada en el servidor por la que entra la información. El puerto 80 es la puerta estándar para el tráfico web (HTTP). 

El problema surge porque solo un programa puede "escuchar" en un puerto a la vez. Al instalar PHP en Ubuntu, el sistema suele instalar automáticamente **Apache2**, un servidor web profesional que se inicia solo y ocupa el puerto 80. Si intentamos levantar un servidor de Node.js o Python en ese mismo puerto, el sistema devolverá el error *"Address already in use"*. 

Para solucionar esto, debemos liberar la puerta apagando el servicio de Apache:
```bash
sudo systemctl stop apache2
```

## 13. Análisis Comparativo de Implementaciones

Para consolidar estos conceptos, analizaremos cómo cada lenguaje resuelve la misma tarea: crear una página que detecte el día de la semana actual y lo muestre al usuario.

### 13.1 Implementación en Python (Flask)
En Python, utilizamos un concepto llamado **decorador** (`@app.route("/")`). Un decorador es una función que "envuelve" a otra para añadirle una propiedad. En este caso, le dice a Flask que la función `home()` debe ejecutarse cada vez que alguien acceda a la raíz del sitio.

**Código de implementación (`app.py`):**
```python
from flask import Flask
import datetime

app = Flask(__name__)

@app.route("/") 
def home():
    # Obtenemos la fecha y la formateamos para extraer el nombre del día
    today = datetime.datetime.now().strftime("%A")
    # Devolvemos un string que el navegador interpretará como HTML
    return f"<html><body><h1>Hoy es {today}</h1></body></html>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

### 13.2 Implementación en Node.js (Express)
Node.js utiliza un modelo basado en eventos. Definimos un manejador de rutas mediante `app.get`. El primer parámetro es la ruta (`/`) y el segundo es una función que recibe dos objetos: la petición (`req`) y la respuesta (`res`).

**Código de implementación (`app.js`):**
```javascript
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  const today = new Date().toLocaleDateString("es-ES", { weekday: "long" });
  res.send(`<html><body><h1>Hoy es ${today}</h1></body></html>`);
});

app.listen(80, () => {
  console.log("Servidor activo en http://localhost:80");
});
```

### 13.3 Implementación en Ruby (Sinatra)
Ruby busca la máxima simplicidad. En Sinatra, no necesitamos envolver la lógica en funciones complejas; definimos la acción (`get '/'`) y el bloque de código devuelve directamente el contenido.

**Código de implementación (`app.rb`):**
```ruby
require 'sinatra'
require 'date'

get '/' do
  today = Date.today.strftime("%A")
  "<html><body><h1>Hoy es #{today}</h1></body></html>"
end
```

### 13.4 Implementación en PHP
PHP es el único de estos lenguajes que no requiere un servidor de aplicación externo para ejecutar lógica simple. El código PHP se incrusta directamente en el HTML. El servidor procesa la etiqueta `<?php ... ?>` y la sustituye por el resultado antes de enviar el archivo al cliente.

**Código de implementación (`index.php`):**
```php
<!DOCTYPE html>
<html lang="es">
<body>
    <h1>Hoy es <?php echo date("l"); ?></h1>
</body>
</html>
```
