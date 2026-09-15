
# Análisis de la Arquitectura Cliente-Servidor con Python


## 1. Marco Teórico
La comunicación Cliente-Servidor es un modelo de diseño de software en el que las tareas se reparten entre los proveedores de recursos o servicios (servidores) y los demandantes de dichos servicios (clientes).

*   **Socket:** Es el punto final de un canal de comunicación bidireccional entre dos programas que se ejecutan en la red.
*   **Binding (Vinculación):** Proceso mediante el cual el servidor asigna una dirección IP y un puerto específicos a su socket.
*   **Listen & Accept:** El servidor entra en estado de escucha y espera a que un cliente inicie la conexión.
*   **Encoding/Decoding:** Dado que la red transporta bytes, el texto debe ser codificado (`encode`) antes de enviarse y decodificado (`decode`) al recibirse.

## 2. Metodología y Herramientas
Para el desarrollo de la práctica se utilizarán los siguientes recursos:
*   **Entorno:** Sistema operativo Linux con intérprete de Python 3.x.
*   **Archivos:** Se entregan los scripts `servidor.py` y `cliente.py`.
*   **Configuración:** Ejecución en entorno local utilizando la dirección de bucle invertido (`localhost` / `127.0.0.1`).

**Instrucciones de ejecución en Linux:**
1.  Abrir una terminal y ejecutar el servidor: `python3 servidor.py`
2.  Abrir una **segunda terminal** independiente y ejecutar el cliente: `python3 cliente.py`


## 3. Desarrollo de la Práctica

### Fase I: Análisis de Código y Ejecución
El estudiante deberá ejecutar el sistema y analizar la relación entre ambos scripts:
1.  **Flujo de Mensajes:** Enviar un mensaje desde el cliente y observar la respuesta en la consola del servidor y viceversa.
2.  **Localización de Parámetros:** Identificar en el código dónde se define la dirección IP y el puerto.
3.  **Mapeo de Funciones:** Identificar la secuencia de llamadas al sistema necesarias para establecer la conexión.

### Fase II: Laboratorio de Experimentación (Prueba y Error)
El alumno deberá provocar fallos controlados para comprender las restricciones del protocolo y registrar los resultados:

| Experimento | Acción | Resultado Observado | Explicación Técnica |
| :--- | :--- | :--- | :--- |
| **A** | Ejecutar el `cliente.py` **antes** que el `servidor.py`. | | |
| **B** | Ejecutar el `servidor.py` y luego intentar abrir **dos** clientes simultáneamente. | | |
| **C** | Cambiar la IP en el cliente a una dirección inexistente (ej. `1.1.1.1`). | | |
| **D** | Cerrar el servidor abruptamente mientras el cliente espera respuesta. | | |

### Fase III: Retos de Modificación (Análisis Funcional)
Sobre la base del código entregado, el estudiante deberá implementar las siguientes mejoras funcionales:

1.  **El Servidor Eco:** Modificar el servidor para que cualquier mensaje recibido sea devuelto íntegramente al cliente.
2.  **Filtro de Contenido:** Implementar una validación en el servidor para que, si el mensaje contiene una palabra prohibida (ej: "error"), el servidor responda con un mensaje de rechazo.
3.  **Contador de Sesiones:** Hacer que el servidor lleve un registro del número de mensajes procesados y lo incluya en cada respuesta (ej: *"Mensaje recibido. Total de mensajes: 5"*).

---

## 4. Entregables y Evaluación
El alumno deberá presentar un informe técnico que incluya:

**A. Matriz de Experimentos:**
La tabla de la Fase II debidamente cumplimentada con los resultados observados y la justificación técnica de cada error.

**B. Código Modificado:**
Los archivos `servidor.py` y `cliente.py` con las implementaciones de la Fase III.

**C. Cuestionario de Análisis:**
1.  ¿Por qué es necesario utilizar `.encode()` y `.decode()` al enviar mensajes? ¿Qué sucedería si se omitieran?
2.  En el experimento B, ¿por qué el segundo cliente no puede comunicarse con el servidor? Explique el concepto de "bloqueo" (*blocking*) en los sockets.
3.  ¿Cuál es la diferencia fundamental entre la IP `127.0.0.1` y una IP pública de internet?
4.  Si quisiéramos que el servidor atendiera a múltiples clientes a la vez, ¿qué concepto de programación debería investigarse? (Sugerencia: *Threading* o *Asincronismo*).

##5. Anexos

Codigo servidor:

```python
import socket

# Configuración de red
HOST = '127.0.0.1'  # Dirección local (localhost)
PORT = 65432        # Puerto alto para evitar conflictos con el sistema

def iniciar_servidor():
    # AF_INET indica que usamos IPv4, SOCK_STREAM indica que usamos TCP
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # Vinculamos el socket a la IP y el puerto definidos
        s.bind((HOST, PORT))
        
        # Ponemos el servidor en modo escucha
        s.listen()
        print(f"Servidor iniciado. Esperando conexión en {HOST}:{PORT}...")
        
        # accept() bloquea la ejecución hasta que un cliente se conecta
        conn, addr = s.accept() 
        
        with conn:
            print(f"Conexión establecida con: {addr}")
            while True:
                # Recibimos hasta 1024 bytes de datos
                data = conn.recv(1024)
                if not data:
                    break # Si no hay datos, el cliente ha cerrado la conexión
                
                # Decodificamos los bytes recibidos a texto
                mensaje = data.decode('utf-8')
                print(f"Cliente dice: {mensaje}")
                
                # Enviamos una respuesta codificada en bytes
                respuesta = "Mensaje recibido correctamente"
                conn.sendall(respuesta.encode('utf-8'))

if __name__ == "__main__":
    iniciar_servidor()

```

Codigo cliente:

```python
import socket

# Configuración de red (deben coincidir con el servidor)
HOST = '127.0.0.1' 
PORT = 65432

def iniciar_cliente():
    # Creamos el socket (IPv4, TCP)
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        try:
            # Intentamos establecer la conexión con el servidor
            s.connect((HOST, PORT))
            print("Conectado al servidor con éxito.")
            
            while True:
                # Solicitamos al usuario que escriba un mensaje
                mensaje = input("Escribe un mensaje para el servidor (o 'salir' para terminar): ")
                
                if mensaje.lower() == 'salir':
                    break
                
                # Enviamos el mensaje codificado en bytes
                s.sendall(mensaje.encode('utf-8'))
                
                # Esperamos la respuesta del servidor
                data = s.recv(1024)
                print(f"Servidor responde: {data.decode('utf-8')}")
                
        except ConnectionRefusedError:
            print("Error: No se pudo conectar. ¿Está el servidor encendido?")

if __name__ == "__main__":
    iniciar_cliente()

```