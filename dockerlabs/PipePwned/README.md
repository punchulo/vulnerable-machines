# PipePwned

**Platform:** DockerLabs  
**OS:** Linux  
**Category:** Web Exploitation (SSTI) → Credential Exposure → Privilege Escalation

🔗 [Ver como web](https://pablogbl.github.io/vulnerable-machines/dockerlabs/PipePwned/) · [Ver walkthrough como web](https://pablogbl.github.io/vulnerable-machines/dockerlabs/PipePwned/PipePwned.html)

---

## Overview

PipePwned simula la consola CI/CD interna de la empresa ficticia MASoftware. Una única petición HTTP sin autenticación al formulario de envío de pipelines desemboca en ejecución de código como root en el host. La máquina encadena tres fallos: una inyección de plantilla del lado del servidor (SSTI) que da RCE como `ciapp`, unas credenciales SSH en texto plano dentro de un `.env` legible por la aplicación web, y un proceso "runner" corriendo como root que ejecuta scripts de un directorio donde un grupo sin privilegios puede escribir.

---

## Enumeration

El escaneo revela dos puertos: `22/tcp` (OpenSSH 8.9p1 Ubuntu) y `80/tcp` (Gunicorn/Flask). El puerto 80 sirve una consola CI/CD de una sola página con los endpoints `/`, `/pipelines/new` (POST), `/api/jobs/<id>/trace`, `/api/pipelines` y `/health`. El fuzzing de directorios solo añade `/health` a las rutas conocidas, y nuclei no reporta CVEs de catálogo al tratarse de una aplicación a medida.

---

## Exploitation

El formulario "Send New Pipeline" (`POST /pipelines/new`, campos `name` y `ref`) refleja ambos campos en su página de confirmación. Enviar `{{7*7}}` devuelve `49`, lo que confirma que la entrada del usuario se evalúa como plantilla Jinja2 del lado del servidor. La causa raíz está en `app.py`: el banner de confirmación se construye concatenando la entrada del usuario en una cadena que después se pasa por `render_template_string()`, recompilando la entrada como plantilla. Al no aplicarse el sandbox de Jinja2, se puede recorrer el modelo de objetos de Python hasta `os.popen()` y ejecutar comandos arbitrarios, obteniendo RCE como el usuario sin privilegios `ciapp`.

---

## Credential Exposure & Lateral Movement

Aprovechando el RCE se lee `/opt/ci/.env`, propiedad de `ciapp`, que contiene credenciales SSH en texto plano de una segunda cuenta con más privilegios, `devops`. Con esas credenciales se accede por SSH como `devops`, obteniendo la flag de usuario.

---

## Privilege Escalation

La cuenta `devops` pertenece al grupo `devops`, que tiene permiso de escritura sobre `/opt/ci/builds`, un directorio que el proceso `/opt/ci/runner.sh` —corriendo como root— sondea y ejecuta. Al depositar un script en ese directorio vía SFTP como `devops`, el runner lo recoge y lo ejecuta como root en cuestión de segundos, permitiendo leer la flag de root y obtener control total del host.

---

## Key Takeaways

- Nunca se debe pasar una cadena construida con entrada del usuario a `render_template_string()`; hay que usar `render_template()` pasando la entrada como variable para que Jinja2 la escape como dato y no como código.
- Los secretos en texto plano dentro de ficheros `.env` legibles por el proceso de la aplicación web convierten un RCE de bajo privilegio en movimiento lateral inmediato.
- Un proceso privilegiado que ejecuta scripts de un directorio donde escribe un grupo sin privilegios convierte la pertenencia a ese grupo en root; el runner debe correr como usuario no privilegiado o de forma aislada.

---

⚠️ Realizado con fines educativos en un entorno controlado.
