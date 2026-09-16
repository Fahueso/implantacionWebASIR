# Frontend Web - Repaso lenguaje de Marcas

## 1. Los Tres Pilares del Desarrollo Web


### 1.1 HTML: La Estructura
El HTML define qué elementos hay en la página. Si queremos un botón, ponemos un botón. Si queremos un título, ponemos un título.



**Ejemplo rápido:**
```html
<h1>Mi Tienda de Libros</h1>
<p>Bienvenidos a la mejor librería online.</p>
<button>Comprar ahora</button>
```
*Aquí el navegador sabe que hay un título principal, un párrafo y un botón, pero no sabe si son rojos, azules, grandes o pequeños.*

### 1.2 CSS: La Presentación
El CSS toma esos elementos de HTML y les da estilo.

**Ejemplo rápido:**
```css
h1 { color: navy; font-family: Arial; }
button { background-color: green; color: white; border-radius: 5px; }
```
*Ahora, el título es azul marino y el botón es verde con bordes redondeados.*

### 1.3 JavaScript: La Interactividad
JavaScript hace que las cosas "pasen".

**Ejemplo rápido:**
```javascript
document.querySelector('button').onclick = function() {
    alert('¡Gracias por tu compra!');
};
```
*Ahora, cuando el usuario hace clic en el botón, aparece un mensaje de alerta.*

## 
## 2. El Lenguaje HTML: Fundamentos y Anatomía

Un documento HTML no es solo una lista de etiquetas; es un árbol organizado. Todo comienza con una estructura base obligatoria.

### 2.1 La Estructura Base
Cualquier página profesional comienza así:

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Mi Primera Página | Aprendiendo HTML</title>
  </head>
  <body>
    <h1>¡Hola, Mundo!</h1>
    <p>Este es el inicio de mi camino como desarrollador.</p>
  </body>
</html>
```

**Explicación del código:**
*   `lang=Identifica a navegadores y robots que la página está en español.
*   `charset="utf-8"`: Permite que las tildes y la "ñ" se vean correc, utilizando el conjunto de caracteres típico uft-8tamente.
*   `viewport`: Es la instrútil mágica para que la se adapte a dispositivos móvilesn móvil.

### 2.2 Identificadores y Clases: ¿Cómo diferenciar elementos?
Imagina que tienes tres párrafosquieres tenerlos clasificados para operar con ellos en bloquee.a rojo. Para eso usamos `id` y `class`.

**Ejemplo práctico:**
```html
<!-- El ID es único, como un DNI -->
<p id="parrafo-especial">Este párrafo es único en toda la página.</p>

<!-- La Clase es grupal, como un uniforme -->
<p class="texto-azul">Este texto será azul.</p>
<p class="texto-azul">Este también será azul porque comparte la clase.</p>
<p class="texto-azul destacado">Este es azul y además tiene un estilo de destacado.</p>


Más adelante veremos como asignar el estilo deseado mediante CSS.
```

## 3. Estructuración de Contenido: Texto, Enlaces y Listas

### 3.1 Jerarquía de Títulos
No uses el `<h1>` solo para que el texto sea grande; úsalo para organizar la  Esta jerarquía la tendrán en cuenta motores de búsqueda, entre otros.importancia.

**Ejemplo de estructura real:**
```html
<h1>Curso de Desarrollo Web</h1> <!-- Título del libro -->
  <h2>Módulo 1: Frontend</h2>      <!-- Capítulo -->
    <h3>Lección 1: HTML</h3>       <!-- Sub-lección -->
      <p>Aquí aprendemos etiquetas.</p>
    <h3>Lección 2: CSS</h3>        <!-- Sub-lección -->
      <p>Aquí aprendemos colores.</p>
  <h2>Módulo 2: Backend</h2>       <!-- Capítulo -->
```

### 3.2 Énfasis y Formato de Texto
Para resaltar ideas dentro de un párrafo:

```html
<p>El lenguaje <strong>JavaScript</strong> es fundamental, 
   pero <em>no es lo mismo</em> que Java.</p>
```
*   `<strong>`: Se ve en negrita (indica importancia).
*   `<em>`: Se ve en cursiva (indica énfasis).

### 3.3 Listas: Organizando Información
Dependiendo de si el orden importa o no, elegi numerada o no numeradamos la lista:

**Ejemplo: Receta de Cocina**
```html
<h3>Ingredientes (Lista desordenada)</h3>
<ul>
  <li>Harina</li>
  <li>Huevos</li>
  <li>Leche</li>
</ul>

<h3>Pasos a seguir (Lista ordenada)</h3>
<ol>
  <li>Batir los huevos.</li>
  <li>Agregar la harina lentamente.</li>
  <li>Hornear por 20 minutos.</li>
</ol>
```

### 3.4 Enlaces e Imágenes
La web es una red de conexiones. Así se crean:

**Ejemplo de Enlaces:**
```html
<!-- Enlace externo que abre en pestaña nueva -->
<a href="https://google.com" target="_blank" rel="noopener">Ir a Google</a>

<!-- Enlace interno a otra página de tu sitio -->
<a href="contacto.html">Ir a la página de contacto</a>
```

**Ejemplo de Imágenes:**
```html
<img src="perrito.jpg" alt="Un cachorro de Golden Retriever jugando" width="400">
```
*El atributo `alt` es vital: si la imagen falla o el usuario es ciego, el navegador leerá esa descr.

## 4. Interactividad Básica: Multimedia y Formularios

En este capítulo pasamos de mostrar información a permitir que el usuario consuma contenido multimedia y, lo más importante, que nos envíe sus propios datos.

### 4.1 Integrando Multimedia (Audio y Video)
Ya no necesitamos plugins externos. HTML5 permite insertar archivos multimedia de forma nativa.

**Ejemplo de Video:**
```html
<section>
  <h3>Tutorial de Instalación</h3>
  <video src="tutorial.mp4" controls poster="miniatura.jpg" width="600">
    Tu navegador no soporta la reproducción de videos.
  </video>
</section>
```

**Explicación del código:**
*   `controls`: Añade los botones de Play, Pausa y Volumen. Sin esto, el video sería solo una imagen estática.
*   `poster`: Define la imagen que se ve antes de darle a Play.
*   El texto dentro de la etiqueta es un "fallback": solo se muestra si el navegador es muy antiguo y no soporta el video.

**Ejemplo de Audio:**
```html
<audio src="podcast.mp3" controls></audio>
```

### 4.2 Formularios: Capturando Datos del Usuario
El formulario es la herramienta más potente para convertir un sitio web en una aplicación.

**Ejemplo de un Formular hacia un PHPio de Registro:**

```html
<form action="procesar_registro.php" method="POST">
  <!-- Campo de Texto con Label -->
  <div>
    <label for="nombre">Nombre completo:</label>
    <input type="text" id="nombre" name="usuario_nombre" required placeholder="Ej. Juan Pérez">
  </div>

  <!-- Campo de Email -->
  <div>
    <label for="correo">Correo electrónico:</label>
    <input type="email" id="correo" name="usuario_email" required>
  </div>

  <!-- Campo de Contraseña -->
  <div>
    <label for="pass">Contraseña:</label>
    <input type="password" id="pass" name="usuario_pass" required>
  </div>

  <!-- Botón de Envío -->
  <button type="submit">Crear Cuenta</button>
</form>
```

**Explicación del código:**
*   `action="procesar_registro.php"`: Indica a dónde se envían los datos una vez que el usuario hace clic en enviar.
*   `method="POST"`: Envía los datos de forma oculta. Es obligatorio para contraseñas.
*   `label for="nombre"` + `id="nombre"`: Esta conexión es clave. Si haces clic en el texto "Nombre completo", el cursor saltará automáticamente al cuadro de texto.
*   `required`: Es una validación nativa de HTML. El navegador no dejará enviar el formulario si el campo está vacío.


## 5. Introducción a CSS:

Ahora que tenemos una estructura funcional, vamos a aprender a diseñarla. El CSS nos permite separar el **qué** (HTML) del **cómo** (CSS).

### 5.1 Formas de aplicar CSS
Existen tres maneras de añadir estilos, pero no todas son recomendables.

**1. Estilo en línea (Inline):** Se escribe dentro de la etiqueta. Solo para pruebas rápidas.
```html
<p style="color: red; font-weight: bold;">Este texto es rojo y negrita.</p>
```

**2. Estilo interno:** Se escribe en el `<head>` del documento. Útil para páginas únicas.
```html
<head>
  <style>
    body { background-color: #f0f0f0; }
    h1 { color: navy; }
  </style>
</head>
```

**3. Estilo externo (La mejor práctica):** Se crea un archivo `.css` aparte y se enlaza en el HTML.
```html
<!-- En el HTML -->
<link rel="stylesheet" href="estilos.css">
```

### 5.2 La Anatomía de una Regla CSS
Para darle estilo a algo, necesitamos un **Selector**, una **Propiedad** y un **Valor**.

**Ejemplo práctico:**
```css
/* Selector: a todos los párrafos con la clase 'destacado' */
p.destacado {
  color: #2c3e50;        /* Propiedad: color / Valor: azul oscuro */
  font-size: 18px;       /* Propiedad: tamaño / Valor: 18 píxeles */
  line-height: 1.6;      /* Propiedad: interlineado / Valor: 1.6 */
  text-align: justify;    /* Propiedad: alineación / Valor: justificado */
}
```

### 5.3 La Cascada y la Prioridad
CSS significa *Cascading Style Sheets* (Hojas de Estilo en Cascada). Si hay dos reglas que chocan, el navegador decide cuál gana según la **especificidad**.

**Ejemplo de conflicto:**
```html
<p id="unico" class="azul">¿De qué color seré?</p>
```
```css
p { color: green; }       /* Prioridad Baja (Etiqueta) */
.azul { color: blue; }    /* Prioridad Media (Clase) */
#unico { color: red; }    /* Prioridad Alta (ID) */
```
**Resultado:** El texto será **rojo**, porque el ID siempre gana sobre la clase y la etiqueta.

## 6. Estilización de Texto y Fondos:

El diseño no es solo poner colores; es guiar la vista del usuario. En este capítulo aprenderemos a controlar la tipografía y el espacio visual.

### 6.1 Tipografía Avanzada
La legibilidad es la prioridad. No basta con elegir una fuente; hay que saber cómo presentarla.

**Ejemplo de tipografía profesional:**
```css
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 16px;
  color: #333333;
  line-height: 1.6; /* Espacio entre líneas para evitar el cansancio visual */
}

h1 {
  font-size: 2.5rem; /* rem es relativo al tamaño raíz, mejor para accesibilidad */
  text-align: center;
  color: #2c3e50;
  text-transform: uppercase; /* Convierte todo a mayúsculas */
}
```
**Explicación del código:**
*   **`font-family`**: Ponemos varias fuentes. Si el usuario no tiene la primera, el navegador prueba la segunda, y así sucesivamente.
*   **`rem`**: A diferencia del `px`, el `rem` permite que si el usuario cambia el tamaño de letra en su navegador por problemas de vista, toda la página se adapte proporcionalmente.

### 6.2 Fondos Dinámicos
El fondo puede ser un color sólido o una imagen que se adapte a cualquier pantalla.

**Ejemplo de fondo a pantalla completa:**
```css
.hero-section {
  background-image: url('paisaje.jpg');
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover; /* La imagen cubre todo el espacio sin deformarse */
  height: 400px;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
```
**Explicación del código:**
*   `background-size: cover`: Es la propiedad más importante. Evita que la imagen se vea estirada o repetida como un mosaico; la recorta inteligentemente para llenar el contenedor.


## 7. Layouts en CSS:



### 7.1 El Modelo de Caja (Box Model)
Cada elemento HTML es un rectángulo compuesto por cuatro capas.

**Ejemplo práctico de una "Tarjeta de Producto":**
```css
.card {
  width: 300px;
  padding: 20px;       /* Espacio interno: separa el texto del borde */
  border: 2px solid #ddd; /* La línea que rodea la caja */
  margin: 15px;       /* Espacio externo: separa esta tarjeta de otras */
  box-sizing: border-box; /* ¡CLAVE! Hace que el padding no sume al ancho total */
}
```
**Explicación del código:**
*   **Sin `border-box`**: Si el ancho es `300px` y el padding es `20px`, la caja mediría en realidad `340px`.
*   **Con `border-box`**: La caja mide exactamente `300px` y el padding "empuja" el contenido hacia adentro. **Usa siempre esta propiedad.**

### 7.2 Posicionamiento (`position`)
A veces necesitamos que un elemento ignore las reglas normales y se coloque donde nosotros queramos.

**Ejemplo: Un botón de "Chat" flotante:**
```css
.btn-chat {
  position: fixed;   /* Se queda pegado a la pantalla aunque hagas scroll */
  bottom: 20px;      /* A 20px del borde inferior */
  right: 20px;       /* A 20px del borde derecho */
  background-color: green;
  color: white;
  padding: 10px;
  border-radius: 50%;
}
```

### 7.3 Fleional
Flexbox sirve para alinear elementos en una sola dirección (ya sea una fila o una columna).

**Ejemplo: Una Barra de Navegación (Navbar):**
```html
<nav class="navbar">
  <div class="logo">MiLogo</div>
  <ul class="menu">
    <li>Inicio</li>
    <li>Productos</li>
    <li>Contacto</li>
  </ul>
</nav>
```
```css
.navbar {
  display: flex;
  justify-content: space-between; /* Logo a la izquierda, Menú a la derecha */
  align-items: center;            /* Centra verticalmente los elementos */
  background-color: #333;
  padding: 10px 20px;
  color: white;
}

.menu {
  display: flex;
  list-style: none;
  gap: 20px; /* Crea espacio exacto entre los elementos del menú */
}
```

### 7.4 CSS plejo
Grid se usa cuando necesitamos controlar filas y columnas al mismo tiempo.

**Ejemplo: Layout de Periódico Digital:**
```css
.grid-container {
  display: grid;
  grid-template-columns: 200px 1fr; /* Columna fija de 200px y el resto flexible */
  grid-template-rows: auto 1fr auto; /* Header, Contenido, Footer */
  gap: 10px;
}

.header { grid-column: 1 / 3; } /* Ocupa desde la col 1 hasta la 3 (todo el ancho) */
.sidebar { grid-column: 1; }    /* Ocupa solo la primera columna */
.main-content { grid-column: 2; } /* Ocupa la segunda columna */
.footer { grid-column: 1 / 3; } /* Ocupa todo el ancho inferior */
```
**Explicación del código:**
*   `1fr`: Significa "una fracción del espacio disponible". Es mucho más flexible que usar porcentajes.
*   `grid-column: 1 / 3`: Le dice al elemento que se estire a través de dos columnas.
tulo 8. Fundamentos de JavaSca Web

Si HTML es el cuerpo y CSS es la ropa, **JavaScript (JS)** es el sistema nervioso. Es el lenguaje que permite que la página "piense", tome decisiones y reaccione a lo que el usuario hace.

### 8.1 El Entorno y la Carga del Script
JavaScript se ejecuta en el navegador del usuario. Para que no ralentice la carga de la página, la mejor práctica es usar el atributo `defer`.

**Ejemplo de integración correcta:**
```html
<head>
  <!-- El script se descarga en paralelo y se ejecuta al final -->
  <script src="app.js" defer></script>
</head>
```

### 8.2 Variables y Constantes: Guardando Información
En programación, necesitamos "cajas" para guardar datos. En JS moderno, usamos `let` y `const`.

**Ejemplo práctico: Un carrito de compras simple**
```javascript
const IVA = 0.16;           // Constante: El impuesto no cambia durante la ejecución
let totalCompra = 0;       // Variable: El total cambiará según lo que el usuario añada
let nombreUsuario = "Ana";   // String: Texto

totalCompra = 100 + 50;     // Actualizamos el valor de la variable
console.log(`Hola ${nombreUsuario}, tu total es: ${totalCompra}`);
```

### 8.3 Tipos de Datos y Operadores
JS es flexible con los tipos de datos, pero debemos saber cuáles existen para evitar errores.

**Ejemplo de tipos y comparaciones:**
```javascript
let edad = 20;              // Number
let esMayorDeEdad = true;   // Boolean
let frutas = ["Manzana", "Pera"]; // Array (Lista)
let usuario = { nombre: "Luis", id: 1 }; // Object (Ficha)

// Operador de comparación estricta (===)
if (edad === 20) {
  console.log("Tienes exactamente 20 años");
}
```
*Nota: Siempre usa `===` en lugar de `==`. El primero comprueba que el valor Y el tipo sean iguales, evitando errores extraños.*

### 8.4 Estructuras de Control: Tomando Decisiones
La potencia de JS reside en su capacidad de ejecutar código basado en condiciones.

**Ejemplo: Validador de acceso**
```javascript
let passwordIngresada = "12345";
const passwordCorrecta = "admin123";

if (passwordIngresada === passwordCorrecta) {
  console.log("Acceso concedido. Bienvenido al panel.");
} else {
  console.log("Contraseña incorrecta. Intenta de nuevo.");
}
```

### 8.5 Funciables
Una función es un bloque de código que hace una tarea específica y que puedes llamar cuantas veces quieras.

**Ejemplo: Calculadora de descuentos**
```javascript
// Definimos la función
function calcularDescuento(precio, porcentaje) {
  let ahorro = precio * (porcentaje / 100);
  return precio - ahorro;
}

// Usamos la función para diferentes productos
let zapato = calcularDescuento(100, 20); // 20% de descuento
let camisa = calcularDescuento(50, 10);  // 10% de descuento

console.log(`El zapato cuesta ${zapato} y la camisa ${camisa}`);
```


### 9. El DOM

El **DOM (Document Object Model)** es la representación que hace el navegador del HTML. JavaScript no edita el archivo `.html`, sino que edita el DOM en la memoria del navegador.

### 9.1 Selección de Elementos: ¿A quién queremos cambiar?
Para modificar algo, primero debemos "atraparlo".

**Ejemplo de selección:**
```html
<h1 id="titulo">Hola Mundo</h1>
<p class="texto">Párrafo 1</p>
<p class="texto">Párrafo 2</p>
```
```javascript
const titulo = document.getElementById("titulo"); // Selecciona por ID (único)
const parrafos = document.querySelectorAll(".texto"); // Selecciona todos los que tengan la clase .texto
```

### 9.2 Modificación de Contenido y Estilos
Una vez seleccionado el elemento, podemos cambiarlo totalmente.

**Ejemplo: Cambiar el color y texto al hacer clic**
```html
<p id="mensaje">Texto original</p>
<button id="btnCambiar">¡Haz clic aquí!</button>
```
```javascript
const btn = document.getElementById("btnCambiar");
const msg = document.getElementById("mensaje");

btn.onclick = function() {
  msg.textContent = "¡El texto ha sido cambiado por JS!";
  msg.style.color = "blue";
  msg.style.fontWeight = "bold";
};
```

### 9.3 Creación Dinámica de Elementos
Podemos añadir contenido a la página sin que el usuario recargue el sitio.

**Ejemplo: Lista de tareas (To-Do List) básica**
```html
<input type="text" id="tareaInput" placeholder="Nueva tarea...">
<button id="btnAgregar">Agregar</button>
<ul id="listaTareas"></ul>
```
```javascript
const input = document.getElementById("tareaInput");
const btn = document.getElementById("btnAgregar");
const lista = document.getElementById("listaTareas");

btn.onclick = function() {
  if (input.value !== "") {
    const nuevoItem = document.createElement("li"); // Crea el elemento <li>
    nuevoItem.textContent = input.value;            // Le asigna el texto del input
    lista.appendChild(nuevoItem);                   // Lo mete dentro de la <ul>
    input.value = "";                                // Limpia el input
  }
};
```

### 9.4 Eventos y el `addEventListener`
La forma profesional de manejar eventos es mediante el "escuchador de eventos".

**Ejemplo: Alerta al pasar el ratón**
```javascript
const imagen = document.querySelector("img");

imagen.addEventListener("mouseenter", () => {
  console.log("El usuario está mirando la imagen");
});

imagen.addEventListener("mouseleave", () => {
  console.log("El usuario quitó el ratón de la imagen");
});
```


## 10. Bootstrap y Frameworks de Diseño: Desarrollo Profesional

Hasta ahora, hemos construido cada botón y cada columna escribiendo líneas de CSS. Sin embargo, en el mundo laboral, la velocidad es clave. **Bootstrap** es la biblioteca de estilos más popular del mundo y permite crear sitios web modernos y responsivos en una fracción del tiempo.

### 10.1 ¿Qué es Bootstrap y cómo se integra?
Bootstrap es un conjunto de clases CSS y componentes JavaScript ya creados. En lugar de escribir 20 líneas de CSS para un botón elegante, solo añades una clase al HTML.

**Ejemplo de integración rápida vía CDN:**
```html
<head>
  <!-- CSS de Bootstrap -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
  <!-- Contenido aquí -->
  
  <!-- JS de Bootstrap (para modales, menús desplegables, etc.) -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
```

### 10.2 El Sistema de Rejilla (Grid Sysstrap
El concepto más potente de Bootstrap es su división de la pantalla en **12 columnas invisibles**. Tú decides cuántas de esas columnas ocupa cada elemento según el dispositivo.

**Ejemplo: Una sección de "Servicios" responsiva**
```html
<div class="container">
  <div class="row">
    <!-- En móvil ocupa 12 (todo), en tablets 6 (mitad), en PC 4 (un tercio) -->
    <div class="col-12 col-sm-6 col-md-4">
      <div class="p-3 border bg-light">Servicio A</div>
    </div>
    <div class="col-12 col-sm-6 col-md-4">
      <div class="p-3 border bg-light">Servicio B</div>
    </div>
    <div class="col-12 col-sm-6 col-md-4">
      <div class="p-3 border bg-light">Servicio C</div>
    </div>
  </div>
</div>
```

**Explicación del código:**
*   `.container`: Centra el contenido y evita que toque los bordes de la pantalla.
*   `.row`: Crea una fila que contiene las columnas.
*   `.col-12 col-sm-6 col-md-4`: Aquí definimos la **responsividad**. El elemento cambia de tamaño automáticamente según el ancho de la pantalla del usuario.

### 10.3 Componentes Listos para Usar
Bootstrap ofrece "piezas de Lego" que ya tienen diseño y comportamiento profesional.

**Ejemplo: Una Tarjeta de Perfil con Botón**
```html
<div class="card" style="width: 18rem;">
  <img src="usuario.jpg" class="card-img-top" alt="Foto de perfil">
  <div class="card-body">
    <h5 class="card-title">Juan Pérez</h5>
    <p class="card-text">Desarrollador Fullstack apasionado por el código limpio.</p>
    <a href="#" class="btn btn-primary">Contactar</a>
    <a href="#" class="btn btn-outline-secondary">Ver Portfolio</a>
  </div>
</div>
```
**Explicación del código:**
*   `.card`: Crea el contenedor con borde y sombra suave.
*   `.btn .btn-primary`: Crea un botón azul profesional con efectos de hover (cambio de color al pasar el ratón).
*   `.btn-outline-secondary`: Crea un botón con borde gris, ideal para acciones secundarias.

### 10.4 Utilidades Rápidas (Helper Classes)
Bootstrap permite hacer ajustes pequeños sin escribir una sola línea de CSS propio.

**Ejemplo de utilidades de espacio y color:**
```html
<div class="mt-5 p-3 text-center bg-dark text-white">
  Este div tiene margen superior (mt-5), padding (p-3), texto centrado, fondo oscuro y letras blancas.
</div>
```
*   `mt-5`: Margin Top nivel 5.
*   `p-3`: Padding nivel 3.
*   `text-center`: Alineación centrada

