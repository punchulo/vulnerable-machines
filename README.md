# Vulnerable Machines

Repositorio donde documento máquinas vulnerables y laboratorios que voy resolviendo como parte de mi preparación práctica para la eJPT.

La idea es ir dejando writeups claros, con el proceso real que sigo en cada máquina: enumeración, análisis de servicios, explotación, errores cometidos, aprendizajes y siguientes pasos.

No busco hacer guías perfectas ni copiar soluciones. El objetivo es practicar, entender el proceso y mejorar la metodología.

---

## Índice de máquinas

| Máquina | Plataforma | Categoría | Técnicas clave |
|---|---|---|---|
| [WalkingCMS](dockerlabs/walkingcms) | DockerLabs | Web Exploitation (WordPress) | wpscan, enumeración de usuarios, fuerza bruta vía XML-RPC, plugin malicioso |
| [Road to Olympus](<dockerlabs/Road To Olimpus>) | DockerLabs | Pivoting y explotación encadenada | chisel, socat, proxychains, SQLi sobre SQLite, sudoers permisivo |
| [injection](dockerlabs/Injection) | DockerLabs | Web Exploitation (SQL Injection) | Bypass de autenticación por SQLi, reutilización de credenciales, escalada con `env` (GTFOBins) |
| [ChocolateFire](dockerlabs/chocolatefire) | DockerLabs | Web Exploitation | Openfire 4.7.4, CVE-2023-32315, bypass de autenticación |
| [BreakMySSH](dockerlabs/Breakmyssh) | DockerLabs | Enumeración y fuerza bruta SSH | Enumeración de usuarios SSH con Metasploit, fuerza bruta con Hydra |
| [TheDog](dockerlabs/TheDog) | DockerLabs | Web Exploitation (Apache 2.4.49) & Privilege Escalation | Apache 2.4.49 RCE, esteganografía (stegsnow), inyección de comandos, credenciales hardcodeadas |
| [PipePwned](dockerlabs/PipePwned) | DockerLabs | Web Exploitation (SSTI) → Escalada de privilegios | SSTI en Jinja2 (`render_template_string`), RCE, credenciales SSH en `.env`, escalada vía runner root con directorio escribible por grupo |

---

## Estructura de cada máquina

Cada carpeta contiene dos documentos:

- **`README.md`** — resumen por fases: `Overview`, `Enumeration`, fases intermedias adaptadas al ataque, `Exploitation`, `Privilege Escalation` y `Key Takeaways`.
- **`WALKTHROUGH.md`** — recorrido cronológico paso a paso, con capturas intercaladas y el razonamiento detrás de cada decisión.

Las capturas viven en la subcarpeta `imagenes/` de cada máquina.

---

## Qué documento en cada máquina

- Objetivo de la máquina.
- Reconocimiento inicial.
- Enumeración de puertos y servicios.
- Hallazgos relevantes.
- Explotación o camino seguido.
- Escalada de privilegios si aplica.
- Aprendizajes.
- Cosas a revisar o mejorar.

---

## Enfoque eJPT

Este repositorio está orientado a reforzar habilidades útiles para la eJPT:

- Enumeración y escaneo de servicios.
- Web application testing.
- Explotación y escalada de privilegios.
- Pivoting entre redes segmentadas.
- Uso de herramientas como Nmap, Gobuster, Burp Suite, Metasploit, Hydra y wpscan.
- Documentación clara del proceso.

---

## Aviso

Todo el contenido se realiza en entornos controlados, máquinas vulnerables públicas o laboratorios propios.

No se incluyen objetivos reales, sistemas de terceros ni información sensible.
