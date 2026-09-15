# Interacción con APIs REST y Análisis del Protocolo HTTP

## Para el desarrollo de la práctica se utilizarán las siguientes herramientas:
*   **Cliente de peticiones:** Postman (o software equivalente).
*   **Servidor de pruebas:** JSONPlaceholder (`https://jsonplaceholder.typicode.com`).


## 1. Desarrollo de la Práctica

### Fase I: Recuperación de Información (Método GET)
El estudiante deberá realizar las siguientes peticiones para analizar la lectura de datos:

1.  **Consulta general:** Realizar una petición `GET` al endpoint `/posts` para obtener el listado completo de publicaciones.
2.  **Consulta específica:** Realizar una petición `GET` al endpoint `/posts/5` para recuperar únicamente el recurso con el identificador 5.
3.  **Consulta filtrada:** Realizar una petición `GET` al endpoint `/posts?userId=1` para filtrar los resultados por un usuario específico.

### Fase II: Creación y Modificación de Recursos (POST, PUT, PATCH)
El estudiante deberá simular la gestión de registros en el servidor:

1. **Creación:** Configurar una petición `POST` hacia `/posts`. En el cuerpo (**Body**), seleccionar formato **raw** y tipo **JSON**, enviando un objeto con los campos `title`, `body` y `userId`.
```json
{
  "title": "Análisis de Protocolos HTTP",
  "body": "Este contenido ha sido generado para la práctica de APIs REST utilizando Postman.",
  "userId": 1
}

```

4. **Actualización Total:** Configurar una petición `PUT` hacia `/posts/1`, enviando el objeto completo con datos modificados.

```json
{
  "id": 1,
  "title": "Título Actualizado por el Alumno",
  "body": "El cuerpo del mensaje ha sido modificado íntegramente mediante una petición PUT.",
  "userId": 1
}

```

5. **Actualización Parcial:** Configurar una petición `PATCH` hacia `/posts/1`, enviando únicamente el campo `title`.

```json
{
  "title": "Título Modificado mediante PATCH"
}

```

### Fase III: Eliminación de Recursos (Método DELETE)
El estudiante deberá solicitar la baja de un recurso:

1.  Configurar una petición `DELETE` hacia el endpoint `/posts/1`.
2.  Ejecutar la petición y analizar la respuesta del servidor.

### Fase IV: Análisis Avanzado de Parámetros y Cabeceras
En esta fase, el estudiante manipulará variables adicionales para observar la reacción del servidor:

1. **Manipulación de Cabeceras (Headers):**

    * Realizar una petición `POST` al endpoint `/posts`, pero cambiar manualmente en la pestaña **Headers** el `Content-Type` de `application/json` a `text/plain`.
    * Modificar la cabecera `User-Agent` por un valor personalizado (ej: `"Estudiante_Analisis_Redes/1.0"`). Investiga la utilidad de esta cabecera.
2. **Parámetros de Consulta Complejos:**

    * Intentar combinar múltiples parámetros en una sola petición `GET` (ej: `/posts?userId=1&id=2`).
    * Solicitar un recurso inexistente mediante un ID inválido (ej: `/posts/9999`).
3. **Pruebas de Robustez (Sintaxis):**

    * Intentar realizar una petición `POST` enviando un cuerpo JSON con errores de sintaxis (ej: omitir una coma o una llave de cierre).


## 5. Entregables y Evaluación
Para la validación de la actividad, el alumno deberá presentar un informe técnico que incluya:

**A. Matriz de Operaciones Básicas:**
Una tabla que detalle cada petición realizada en las Fases I, II y III, incluyendo:

*   Método HTTP empleado.
*   Endpoint utilizado.
*   Código de estado de la respuesta.

**B. Matriz de Pruebas de Configuración:**
Una tabla que detalle los experimentos de la Fase IV:

| Variable Modificada | Acción Realizada | Resultado Observado | Código de Estado |
| :--- | :--- | :--- | :--- |
| Content-Type | Cambiado a `text/plain` | | |
| User-Agent | Modificado a valor personalizado | | |
| Query Params | Filtrado múltiple | | |
| Sintaxis JSON | Envío de JSON mal formado | | |
| Método HTTP | `PATCH` con datos parciales | | |

**C. Análisis Técnico Final:**
Responder a los siguientes cuestionamientos:
1.  ¿Cuál es la diferencia técnica entre un parámetro de ruta (`/posts/1`) y un parámetro de consulta (`/posts?userId=1`)?
2.  ¿Qué sucede cuando el cliente y el servidor no acuerdan el mismo `Content-Type`?
3.  Explique la diferencia operativa entre los métodos `PUT` y `PATCH`.
4.  ¿Qué ocurre técnicamente cuando el servidor devuelve un código de error 404 y en qué se diferencia de un error 400?
5.  Justifique la importancia de utilizar el formato JSON sobre el texto plano en la comunicación entre sistemas heterogéneos.