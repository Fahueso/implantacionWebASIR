# Bloque 2: Entornos de Desarrollo y Servidores Web

## 2.1. Introducción a los Entornos de Desarrollo Web

Antes de comenzar a desarrollar aplicaciones web, es imprescindible disponer de un entorno de trabajo adecuado. Definimos un **entorno de desarrollo web** como el conjunto de herramientas, servicios y configuraciones que permiten desarrollar, probar y desplegar aplicaciones. En este módulo, nuestro trabajo se centrará en tecnologías fundamentales como los servidores Apache o Nginx, el lenguaje PHP y el sistema de gestión de bases de datos MariaDB. El objetivo final es que el alumno sea capaz de montar un entorno que reproduzca, en la medida de lo posible, las condiciones reales donde se ejecutará la aplicación. 

### La necesidad de diferentes entornos
En el desarrollo de software, es habitual realizar cambios constantes, corregir errores y experimentar con nuevas funcionalidades. Si estas modificaciones se hicieran directamente sobre la aplicación que utilizan los usuarios finales, cualquier error podría provocar interrupciones del servicio o, en el peor de los casos, la pérdida de información. Por esta razón, las organizaciones implementan un ciclo de vida dividido en tres entornos:

1.  **Entorno de Desarrollo:** Es el espacio utilizado por desarrolladores y administradores para crear y modificar la aplicación. Aquí se programan las funciones, se corrigen errores y se realizan las primeras pruebas. Un equipo de desarrollo típico suele contar con Apache, PHP, MariaDB, el sistema de control de versiones Git y un editor de código como Visual Studio Code. Sus ventajas son la configuración sencilla y la libertad de experimentación sin afectar a nadie, aunque su principal inconveniente es que puede diferir ligeramente del entorno final.
2.  **Entorno de Pruebas (Staging o Preproducción):** Su objetivo es validar que la aplicación funciona correctamente antes del lanzamiento. Aquí se realizan pruebas funcionales, comprobaciones de seguridad y verificaciones de despliegue. Es un entorno espejo de la producción, pero sin usuarios reales.
3.  **Entorno de Producción:** Es el entorno final donde residen los usuarios. Ejemplos de esto serían el portal web de una empresa o la plataforma Aules de la Generalitat Valenciana. En esta etapa, los pilares fundamentales son la disponibilidad, la estabilidad, la seguridad y el rendimiento. Cualquier cambio aquí debe estar estrictamente controlado.

### El concepto de "Stack" de desarrollo
Para que una aplicación web funcione, necesita varios componentes trabajando conjuntamente. A este conjunto de tecnologías se le llama **stack**. Los más habituales son:
*   **Stack LAMP:** Compuesto por Linux (Sistema Operativo), Apache (Servidor Web), MariaDB o MySQL (Base de Datos) y PHP (Lenguaje de programación). Es el stack sobre el que corren aplicaciones como WordPress, Moodle o Nextcloud.
 *   **Stack LEMP:** Es una alternativa donde se sustituye Apache por **Nginx** (pronunciado *Engine-X*). La principal diferencia radica en cómo el servidor gestiona las conexiones y el rendimiento.

![](../assets/images/ud01/Pasted%20image%2020260909173157.png)

### Aislamiento: Virtualización y Contenedores
Para evitar conflictos entre versiones de software y mantener el sistema limpio, utilizamos técnicas de aislamiento:
*   **Virtualización:** Permite ejecutar varios sistemas operativos independientes sobre un mismo equipo físico mediante máquinas virtuales. Ofrece un aislamiento total y permite crear "instantáneas" (snapshots) para restaurar el sistema a un estado anterior. Sin embargo, consume mucha memoria y almacenamiento, y el arranque es lento ya que requiere un SO completo por cada máquina.
*   **Contenedores:** Son una alternativa más ligera. En lugar de virtualizar el SO completo, comparten el núcleo del sistema anfitrión y ejecutan solo la aplicación y sus dependencias. Esto permite un arranque casi instantáneo y un consumo de recursos mínimo.
*   **Docker:** Es la tecnología de contenedores más utilizada hoy en día. Permite crear y distribuir aplicaciones mediante contenedores independientes (por ejemplo, un contenedor para Apache, otro para PHP y otro para MariaDB), facilitando que el entorno sea idéntico en cualquier equipo.


## 2.2. Servidores Web: Análisis y Comparativa

El servidor web es el software encargado de procesar las peticiones HTTP de los usuarios y entregarles el contenido solicitado. Aunque existen muchas opciones, destacan tres principales:

**Apache HTTP Server** es un proyecto de software libre desarrollado por la Apache Software Foundation. Es extremadamente popular en entornos educativos y empresariales debido a su estabilidad, su arquitectura modular y la inmensa cantidad de documentación disponible. Es la base del stack LAMP que utilizaremos en este módulo.

**Nginx** fue diseñado específicamente para ofrecer un alto rendimiento y un bajo consumo de memoria. Utiliza una arquitectura basada en eventos que le permite gestionar un número masivo de conexiones simultáneas, por lo que se usa frecuentemente como balanceador de carga o proxy inverso.

**Microsoft IIS (Internet Information Services)** es el servidor desarrollado por Microsoft para Windows Server. Su gran ventaja es la integración total con el ecosistema de Microsoft, como Active Directory y el soporte nativo para aplicaciones ASP.NET.


## 2.3. Apache HTTP Server y su Arquitectura Modular

Apache ha mantenido su popularidad desde los años 90 gracias a su flexibilidad. Sus funciones son muy diversas: desde publicar páginas estáticas y servir imágenes, hasta ejecutar aplicaciones PHP, gestionar conexiones seguras HTTPS, autenticar usuarios y redirigir peticiones.

### El sistema de módulos
La clave del éxito de Apache es su **arquitectura modular**. El servidor tiene un núcleo reducido y las funcionalidades adicionales se añaden mediante módulos que pueden activarse o desactivarse. Esto permite que el servidor sea ligero y seguro, ya que solo cargamos lo que necesitamos, reduciendo así la "superficie de ataque" para posibles hackers.

En los sistemas Debian y Ubuntu, los módulos se organizan de la siguiente manera:
*   Los módulos disponibles se almacenan en `/etc/apache2/mods-available`.
*   Cuando activamos un módulo, Apache crea un enlace simbólico en `/etc/apache2/mods-enabled`.

Para gestionar estos módulos, disponemos de herramientas específicas:
*   **Activar un módulo:** `sudo a2enmod nombre_modulo` (ej. `sudo a2enmod rewrite`).
*   **Desactivar un módulo:** `sudo a2dismod nombre_modulo`.
*   **Consultar módulos activos:** `sudo apache2ctl -M`.

### Módulos habituales y sus funciones
*   `mod_alias`: Permite crear alias para acceder a directorios mediante rutas diferentes.
*   `mod_dir`: Gestiona las páginas predeterminadas de un directorio (como `index.html` o `index.php`).
*   `mod_autoindex`: Genera listados de archivos cuando no existe una página de inicio.
*   `mod_rewrite`: Permite modificar y reescribir URLs, transformando una ruta compleja como `producto.php?id=25` en una amigable como `/productos/25`.
*   `mod_ssl`: Imprescindible para habilitar HTTPS, cifrar comunicaciones y usar certificados digitales.
*   `mod_userdir`: Permite que cada usuario del sistema tenga su propio espacio web personal.
*   **Módulos de autenticación:** `mod_auth_basic`, `mod_authn_*` y `mod_authz_*`, que sirven para solicitar contraseñas y restringir el acceso a recursos.


## 2.4. Instalación y Gestión del Servidor

En Debian, la instalación se realiza mediante el gestor de paquetes APT con el comando `sudo apt install apache2`. Durante este proceso, se instalan también componentes auxiliares: `apache2-data` (con iconos y páginas de error) y `apache2-utils` (que incluye herramientas como `ab` para pruebas de rendimiento y `htpasswd` para gestionar usuarios).

### Control del servicio
El servidor se gestiona mediante `systemd` con los siguientes comandos:
*   `sudo systemctl start apache2` (Iniciar)
*   `sudo systemctl stop apache2` (Detener)
*   `sudo systemctl restart apache2` (Reiniciar el servicio por completo)
*   `sudo systemctl reload apache2` (Recargar la configuración sin interrumpir el servicio)
*   `sudo systemctl status apache2` (Consultar el estado)

### Verificación y Diagnóstico Inicial
Para comprobar que Apache funciona, podemos acceder a `http://localhost` o a la dirección IP del equipo (obtenida con `hostname -I` o `ip addr`). También podemos usar la terminal con `curl http://localhost` o `wget -O - http://localhost`.


## 2.5. Configuración Detallada de Apache

La configuración de Apache en Debian es jerárquica. El archivo principal es `/etc/apache2/apache2.conf`, donde se definen los parámetros globales. Los puertos de escucha se configuran en `/etc/apache2/ports.conf` mediante la directiva `Listen 80`.

### Gestión de Sitios y Configuraciones
Apache separa los sitios definidos de los sitios activos:
*   **Sitios disponibles:** `/etc/apache2/sites-available`. Aquí se guarda la configuración de cada web. El archivo `000-default.conf` es el sitio por defecto.
*   **Sitios activos:** `/etc/apache2/sites-enabled`. Contiene enlaces a los sitios que Apache debe cargar al arrancar.
*   **Comandos:** `sudo a2ensite` para activar y `sudo a2dissite` para desactivar.

De igual forma, existen las configuraciones generales en `conf-available` y `conf-enabled`, gestionadas con `a2enconf` y `a2disconf`.

### El DocumentRoot
Cada sitio necesita un directorio donde guardar sus archivos públicos, llamado **DocumentRoot**. Por defecto es `/var/www/html`. Si un usuario pide `index.html`, Apache lo buscará exactamente en esa ruta.


## 2.6. Funcionalidades Avanzadas de Alojamiento

### Virtual Hosts (Hosts Virtuales)
Un único servidor puede alojar múltiples sitios web independientes. Esto se hace mediante Virtual Hosts. Existen dos tipos:
1.  **Basados en IP:** Cada sitio tiene su propia IP (poco común).
2.  **Basados en nombre:** Todos comparten la misma IP y puerto, pero Apache distingue el sitio mediante la cabecera `Host` de la petición.

Para configurar un Virtual Host, creamos un archivo en `sites-available` con las directivas `ServerName` (el dominio, ej. `miweb1.local`) y `DocumentRoot` (la carpeta del sitio). También es recomendable definir `ErrorLog` y `CustomLog` independientes para cada sitio. Para que el navegador reconozca estos dominios locales, debemos añadirlos al archivo `/etc/hosts`.

### Directorios Personales (`mod_userdir`)

Este módulo permite que cada usuario del sistema tenga su propio espacio web independiente en `/home/usuario/public_html`, accesible mediante la URL `http://localhost/~usuario`. Es muy útil en entornos educativos para que cada alumno despliegue sus pruebas sin interferir con los demás.

**Procedimiento de activación:**

1. **Habilitar el módulo:** `sudo a2enmod userdir` $\rightarrow$ `sudo systemctl reload apache2`.
2. **Crear el espacio web:** El usuario debe crear la carpeta específica en su home: `mkdir ~/public_html`.
3. **Asignar permisos:** Para que Apache pueda leer el contenido, la carpeta debe tener permisos de lectura y ejecución: `chmod 755 ~/public_html`.

**Importante: La ejecución de PHP en directorios personales** Por defecto, en Debian y Ubuntu, la ejecución de scripts PHP está **desactivada** dentro de los directorios personales por razones de seguridad (para evitar que un usuario ejecute código malicioso en el home de otro). Si al acceder a `~usuario/index.php` el navegador descarga el archivo o muestra el código fuente en lugar de ejecutarlo, es necesario habilitarlo manualmente:

1. Editar el archivo de configuración del módulo PHP: `sudo nano /etc/apache2/mods-enabled/php8.x.conf` (sustituir `8.x` por la versión instalada).
2. Localizar el bloque que contiene la directiva `<IfModule mod_userdir.c>` y **comentar** la línea `php_admin_value engine Off` añadiendo un `#` al principio: `# php_admin_value engine Off`
3. Guardar los cambios y reiniciar el servidor: `sudo systemctl restart apache2`.


## 2.7. Seguridad y Control de Accesos

### Autenticación y Autorización
La **autenticación** verifica la identidad (¿quién eres?), mientras que la **autorización** define los permisos (¿qué puedes hacer?). 

La **Autenticación Básica** se implementa con `mod_auth_basic`. Primero creamos el archivo de contraseñas con `sudo htpasswd -c /etc/apache2/.htpasswd usuario`. Luego, en la configuración del directorio, añadimos:
*   `AuthType Basic`
*   `AuthName "Mensaje de aviso"`
*   `AuthUserFile /etc/apache2/.htpasswd`
*   `Require valid-user` (o un usuario específico).

También podemos restringir el acceso por dirección IP usando la directiva `Require ip 192.168.1.0/24`.

### HTTPS y Cifrado
Para proteger la información, activamos `mod_ssl` y el sitio `default-ssl`. En desarrollo, generamos un certificado autofirmado con:
`sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt`.
En producción, se utilizan certificados de autoridades como **Let's Encrypt**.

### Cortafuegos con UFW
Para que el servidor sea accesible, usamos `ufw`. Los perfiles disponibles son:
*   `Apache`: Puerto 80.
*   `Apache Secure`: Puerto 443.
*   `Apache Full`: Ambos puertos.
**Regla de oro:** Siempre ejecutar `sudo ufw allow OpenSSH` antes de `sudo ufw enable` para evitar quedar bloqueados fuera del servidor.


## 2.8. Plataformas Integradas y Documentación Profesional

### Alternativas de Despliegue
Para agilizar el desarrollo, existen plataformas integradas:
*   **XAMPP:** Instala Apache, PHP y MariaDB en un solo paquete. Es rápido pero su configuración difiere de un servidor real.
*   **Docker Compose:** Permite definir todo el stack en un archivo `docker-compose.yml`. Es la opción profesional, ya que garantiza que el entorno sea idéntico en cualquier máquina.

### La Documentación Técnica
Documentar es un criterio de evaluación obligatorio. Una documentación profesional debe permitir que cualquier técnico reproduzca el sistema. Debe seguir una estructura: **Objetivo, Entorno, Procedimiento, Configuración, Comprobación e Incidencias**.


## 2.9. Diagnóstico y Resolución de Problemas (Troubleshooting)

Cuando un servidor web no responde o no muestra el contenido esperado, es fundamental seguir un proceso de descarte lógico para localizar el fallo.

### El flujo de diagnóstico básico
Si un sitio web no es accesible desde otro equipo de la red, debemos hacernos la siguiente pregunta: **¿Responde el servidor en localhost?**
*   **Si la respuesta es NO:** El problema está en el servidor Apache. Debemos revisar si el servicio está activo o si hay errores de configuración.
*   **Si la respuesta es SÍ:** El servidor funciona, pero algo impide que la petición llegue desde el exterior. El problema suele estar en el cortafuegos (`ufw`) o en la red.

### Problemas comunes y sus soluciones

**1. El servidor no arranca tras modificar la configuración**
Si Apache falla al iniciar después de editar un archivo `.conf`, el primer paso es validar la sintaxis:
`sudo apache2ctl configtest`
Si hay un error, el comando nos indicará la línea exacta y el archivo donde se encuentra el fallo. Solo cuando el resultado sea `Syntax OK` debemos intentar reiniciar el servicio.

**2. El puerto 80 está ocupado**
Apache no puede iniciar si otro programa ya está utilizando el puerto 80. Para identificar qué proceso está bloqueando el puerto, ejecutamos:
`sudo ss -tulpn | grep :80`
Esto nos mostrará el nombre del proceso y su PID, permitiéndonos detenerlo o cambiar el puerto de Apache en `ports.conf`.

**3. La página no es accesible desde la red (pero sí en localhost)**
Si `curl http://localhost` funciona pero el navegador de otro equipo no, debemos revisar el cortafuegos:
`sudo ufw status`
Si el estado es `active` pero no vemos el perfil de Apache, debemos ejecutar `sudo ufw allow "Apache Full"`.

**4. Errores de "Forbidden" (403) o "Not Found" (404)**
Cuando el servidor responde pero da un error de acceso:
*   **403 Forbidden:** Suele ser un problema de permisos de Linux. Debemos comprobar que el directorio `DocumentRoot` y sus archivos tengan permisos de lectura y ejecución para el usuario de Apache (`www-data`). En el caso de los directorios personales, recordar ejecutar `chmod 755 ~/public_html`.
*   **404 Not Found:** El archivo solicitado no existe en el `DocumentRoot` o el nombre del archivo está mal escrito (recordar que en Linux las mayúsculas y minúsculas importan).

**5. Análisis profundo mediante Logs**
Cuando el error no es evidente, la fuente de verdad son los registros:
*   **Log de errores (`/var/log/apache2/error.log`):** Aquí aparecen fallos de módulos, errores de permisos y caídas del servicio. Usar `sudo tail -f /var/log/apache2/error.log` para ver los errores en tiempo real mientras refrescamos la web.
*   **Log de accesos (`/var/log/apache2/access.log`):** Permite ver si la petición del cliente está llegando realmente al servidor y qué código de respuesta está devolviendo Apache.