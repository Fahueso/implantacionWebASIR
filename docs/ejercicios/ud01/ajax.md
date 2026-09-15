# Caso de Estudio: Análisis de la API de Wikipedia


### Metodología
Wikipedia utiliza una API llamada **MediaWiki**. Para obtener resultados de búsqueda en formato JSON, utilizaremos el siguiente endpoint:

**URL de búsqueda:** `https://es.wikipedia.org/w/api.php`

### Desarrollo del Ejercicio

El alumno deberá configurar una petición `GET` en Postman con los siguientes **Query Parameters** (Parámetros de consulta). En lugar de escribir la URL larga, deberá añadirlos en la pestaña **Params** de Postman:

| Key | Value | Descripción |
| :--- | :--- | :--- |
| `action` | `query` | Indica que queremos realizar una consulta. |
| `list` | `search` | Especifica que la consulta es una búsqueda de texto. |
| `srsearch` | `Python` | El término que queremos buscar (puede cambiarse por cualquier otro). |
| `format` | `json` | **Crucial:** Indica al servidor que queremos la respuesta en JSON y no en HTML. |

### Análisis de la Respuesta

Una vez ejecutada la petición, el alumno deberá analizar la estructura del JSON devuelto y responder a lo siguiente:

1.  **Jerarquía de Datos:** El JSON de Wikipedia es más complejo que el de JSONPlaceholder. 
    *   ¿En qué "capa" o clave se encuentran los resultados de la búsqueda? (Ej: `query` $\rightarrow$ `search`).
2.  **Análisis de un Resultado:** Selecciona el primer resultado de la lista y localiza los siguientes campos:
    *   `title`: ¿Cuál es el título exacto del artículo?
    *   `pageid`: ¿Cuál es el identificador numérico único de esa página en la base de datos de Wikipedia?
3.  **El experimento del Formato:** 
    *   Cambia el parámetro `format=json` por `format=xml`. 
    *   **Análisis:** ¿Cómo cambia la estructura de la respuesta? ¿Sigue siendo legible para un humano? ¿Por qué el JSON es preferible para una aplicación móvil o web?
