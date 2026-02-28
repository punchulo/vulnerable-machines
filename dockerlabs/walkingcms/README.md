# WalkingCMS

**Platform:** DockerLabs  
**OS:** Linux  
**Category:** Web Exploitation (WordPress)

---

## Overview

WalkingCMS es una máquina orientada a la explotación de aplicaciones web basadas en WordPress.  
El objetivo fue identificar la superficie de ataque, explotar configuraciones inseguras y escalar privilegios hasta comprometer completamente el sistema.

---

## Enumeration

Tras desplegar la máquina se identifica el puerto 80 abierto, confirmando un servicio HTTP activo.

El análisis revela que la aplicación está basada en **WordPress 6.4.3** con el servicio XML-RPC habilitado, lo que introduce un vector potencial para ataques de fuerza bruta optimizados.

---

## User Enumeration & Brute Force

Mediante `wpscan` se identifica un usuario válido del sistema: `mario`.

Aprovechando XML-RPC, se realiza un ataque de fuerza bruta utilizando una wordlist estándar, obteniendo credenciales válidas para el panel de administración.

---

## Exploitation

Con acceso al panel de WordPress, se instala un plugin que permite la modificación de archivos del servidor.

Se introduce una reverse shell en un archivo PHP, obteniendo acceso remoto al sistema.

---

## Privilege Escalation

Durante la fase de post-explotación se identifican binarios con el bit SUID activado.  
Entre ellos destaca `/usr/bin/env`, el cual puede utilizarse como vector de escalada.

Aplicando técnicas documentadas en GTFOBins se consigue acceso como **root**, comprometiendo completamente la máquina.

---

## Key Takeaways

- Riesgos asociados a XML-RPC habilitado.
- Enumeración de usuarios mediante respuestas diferenciadas.
- Impacto de plugins administrativos inseguros.
- Escalada de privilegios mediante binarios SUID mal configurados.

---

⚠️ Realizado con fines educativos en un entorno controlado.
