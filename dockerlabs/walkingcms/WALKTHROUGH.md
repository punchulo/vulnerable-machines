# WalkingCMS

En este ejercicio vamos a vulnerar la máquina **WalkingCMS** de la plataforma DockerLabs, siguiendo un proceso estructurado que abarca enumeración inicial, identificación de vulnerabilidades, explotación del servicio web y escalada de privilegios hasta obtener control total del sistema.

El objetivo principal no es únicamente comprometer la máquina, sino comprender cada paso del proceso y analizar por qué cada vulnerabilidad permite avanzar en la intrusión.

---

## Despliegue de la máquina

Comenzamos levantando la máquina utilizando el comando proporcionado por la plataforma:

![Deploy](/dockerlabs/walkingcms/imagenes/Deploy.png)

Una vez desplegada correctamente, se nos proporciona una dirección IP interna que será el objetivo del análisis. A partir de este momento iniciamos la fase de reconocimiento, que resulta clave para entender la superficie de ataque disponible.

---

## Enumeración inicial

El primer paso consiste en realizar un escaneo de puertos sobre la IP proporcionada. Guardamos el resultado en un archivo llamado `escaneo` para poder analizarlo con mayor detalle posteriormente.

![Scan](/dockerlabs/walkingcms/imagenes/Scan.png)

Una vez finalizado el escaneo obtenemos el siguiente resultado:

![Scan Result](/dockerlabs/walkingcms/imagenes/Scan_Result.png)

En el análisis observamos que el puerto **80** está abierto, lo que indica la presencia de un servicio HTTP activo. Esto nos sugiere que el vector de ataque principal será una aplicación web.

El hecho de que el servicio esté expuesto únicamente por HTTP y no HTTPS implica que la comunicación no está cifrada, lo que en entornos reales podría facilitar la interceptación de tráfico.

---

## Identificación de la aplicación web

Accedemos desde el navegador a la dirección IP objetivo:

![Web](/dockerlabs/walkingcms/imagenes/Web.png)

Observamos que se trata de una aplicación web, por lo que comenzamos a identificar la tecnología utilizada. Dado que WordPress es uno de los CMS más utilizados, probamos acceder a la ruta habitual añadiendo `/wordpress` a la URL.

![WordPress Path](/dockerlabs/walkingcms/imagenes/WordPress_Path.png)

Con esta comprobación confirmamos que la aplicación está basada en **WordPress**, lo que nos permite enfocar la fase de explotación hacia vulnerabilidades típicas de este CMS.

---

## Identificación de versión

Para determinar la versión exacta instalada utilizamos la extensión **Wappalyzer**, que permite identificar tecnologías y versiones empleadas en aplicaciones web.

![Version](/dockerlabs/walkingcms/imagenes/Version.png)

La versión instalada es la **6.4.3**. Aunque no se trata de una versión extremadamente antigua, es importante revisar posibles configuraciones inseguras y servicios habilitados que puedan facilitar ataques.

---

## Análisis de XML-RPC

Revisamos el código fuente de la página en busca de referencias a XML-RPC:

![XMLRPC](/dockerlabs/walkingcms/imagenes/XMLRPC.png)

La presencia de XML-RPC habilitado es relevante, ya que permite realizar ataques de fuerza bruta más eficientes. Este servicio acepta múltiples intentos de autenticación en una sola petición, lo que reduce el número de solicitudes necesarias para probar credenciales.

---

## Enumeración de usuarios

Accedemos al panel de login de WordPress:

![Login](/dockerlabs/walkingcms/imagenes/Login.png)

Observamos que WordPress devuelve mensajes distintos dependiendo de si el usuario existe o no:

![Login Error](/dockerlabs/walkingcms/imagenes/Login_Error.png)

Este comportamiento permite realizar enumeración de usuarios válidos. Para automatizar el proceso utilizamos la herramienta `wpscan`.

![WPScan](/dockerlabs/walkingcms/imagenes/WPScan.png)

Tras ejecutar la herramienta, identificamos el siguiente usuario válido:

`mario`

![User Mario](/dockerlabs/walkingcms/imagenes/User_Mario.png)

Al probar este usuario en el formulario de login, el mensaje cambia indicando que la contraseña es incorrecta, confirmando que el usuario existe en el sistema.

![User Confirm](/dockerlabs/walkingcms/imagenes/User_Confirm.png)

---

## Ataque de fuerza bruta

Con un usuario válido identificado, procedemos a realizar un ataque de fuerza bruta utilizando la wordlist **Rockyou**, aprovechando la funcionalidad de XML-RPC.

![Bruteforce](/dockerlabs/walkingcms/imagenes/Bruteforce.png)

Tras ejecutar el ataque obtenemos una contraseña válida:

![Password Found](/dockerlabs/walkingcms/imagenes/Password_Found.png)

Con estas credenciales accedemos al panel de administración de WordPress:

![Admin Panel](/dockerlabs/walkingcms/imagenes/Admin_Panel.png)

Esto nos permite modificar el contenido y la configuración del sitio.

---

## Explotación – Reverse Shell

Una vez dentro del panel de administración, instalamos el plugin **File Manager**, que permite acceder y modificar archivos directamente desde la interfaz web.

![File Manager Plugin](/dockerlabs/walkingcms/imagenes/File_Manager_Plugin.png)

A través de este plugin navegamos hasta un archivo `.php` susceptible de modificación e insertamos un reverse shell en PHP.

![Reverse Shell](/dockerlabs/walkingcms/imagenes/Reverse_Shell.png)

Antes de ejecutar el archivo modificado, configuramos nuestro equipo atacante en modo escucha:

![Listener](/dockerlabs/walkingcms/imagenes/Listener.png)

Al recargar el archivo, se establece la conexión reversa y obtenemos acceso al sistema como usuario del servicio web.

![Shell](/dockerlabs/walkingcms/imagenes/Shell.png)

---

## Escalada de privilegios

Una vez dentro del sistema, el siguiente paso es escalar privilegios.

Realizamos una búsqueda de binarios con el bit SUID activado:

![SUID Search](/dockerlabs/walkingcms/imagenes/SUID_Search.png)

Entre los resultados encontramos el binario:

`/usr/bin/env`

![GTFOBins env](/dockerlabs/walkingcms/imagenes/GTFOBins_env.png)

Consultando **GTFOBins**, identificamos una técnica que permite abusar de este binario para ejecutar comandos con privilegios elevados.

Tras aplicar la técnica correspondiente, verificamos nuestra identidad:

![Root](/dockerlabs/walkingcms/imagenes/Root.png)

Confirmamos así el acceso como **root**, logrando el compromiso completo del sistema.

---

## Conclusión

La máquina WalkingCMS demuestra cómo una combinación de configuraciones inseguras en WordPress, la presencia de XML-RPC habilitado y permisos mal configurados en el sistema pueden derivar en una intrusión completa.

Este ejercicio refuerza la importancia de:

- Deshabilitar servicios innecesarios como XML-RPC.
- Evitar mensajes diferenciados en autenticación.
- Limitar la instalación de plugins administrativos.
- Revisar binarios con bit SUID activado.

⚠️ Realizado con fines educativos y en un entorno controlado.
