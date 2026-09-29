!!! info "Criterios de Evaluación (RA5)"
    En este apartado se trabajan los siguientes puntos del Resultado de Aprendizaje:

    - **a)** Se han identificado los lenguajes de guiones de servidor más relevantes.
    - **b)** Se ha reconocido la relación entre los lenguajes de guiones de servidor y los lenguajes de marcas utilizados en los clientes.

# 1. Identificación de lenguajes de servidor
Realiza un programa que imprima la fecha actual del servidor.
Se pide una versión diferente del programa por cada uno de los lenguajes de programación de scripts en el lado de servidor vistos en clase.

Todos los programas deben de poder accederse simultáneamente, con los puertos indicados.

## Snippets de ayuda:

### PHP-> Puerto 8080
```php
$hora = date('H:i:s');
```

### Python con Flask -> Puerto 8081
```python
hora = datetime.datetime.now().strftime("%H:%M:%S")
```

### Node js -> Puerto 8082

```javascript
const hora = new Date().toLocaleTimeString("es-ES")
```
### Ruby con Sinatra -> Puerto 8083

```ruby
hora = Time.now.strftime('%H:%M:%S')
```

# 2. Entregables y Evaluación
Sube a aules una carpeta comprimida con el código fuente, captura de consola y captura de  navegador
