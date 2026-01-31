**Plataforma:** DockerLabs  
**Sistema:** Linux  
**Dificultad:** Media  
**Tipo:** Web exploitation  

---

## 🧭 Enumeración

Tras desplegar la máquina, se realiza una enumeración inicial de servicios con
el objetivo de identificar posibles vectores de entrada.

El análisis revela múltiples puertos abiertos, destacando un servicio web
expuesto en el puerto **9090**, que se convierte en el foco principal del
análisis.

Al acceder a dicho servicio, se identifica un panel de administración de
**Openfire**, desde el cual es posible obtener información relevante como la
versión exacta del software.

---

## ⚔️ Identificación de vulnerabilidad

La versión identificada corresponde a **Openfire 4.7.4**, una versión que
presenta vulnerabilidades públicas conocidas.

Tras la investigación, se localiza la vulnerabilidad **CVE-2023-32315**, que
permite un bypass de autenticación en la consola de administración, lo que
supone un vector claro de explotación.

---

## ⚙️ Explotación

En primer lugar, se prueba la explotación manual de la vulnerabilidad mediante
un exploit público, lo que permite la creación de un usuario con privilegios
administrativos dentro del panel de Openfire.

Sin embargo, este acceso inicial se limita al panel de administración y no
proporciona control directo sobre el sistema operativo.

Para obtener acceso al sistema, se recurre a **Metasploit Framework**,
aprovechando la misma vulnerabilidad previamente identificada.

La explotación es exitosa y se obtiene una sesión remota en la máquina objetivo.

---

## 🧱 Post-explotación

Una vez obtenida la sesión, se verifica el contexto y los privilegios
disponibles, confirmando el acceso con privilegios de administrador (**root**).

El objetivo de la máquina queda así completamente cumplido.

---

## 📌 Aprendizajes

- Importancia de una enumeración de servicios completa antes de explotar
- Relevancia de identificar versiones exactas de software expuesto
- Diferencia entre acceso a una aplicación y control del sistema
- Uso responsable de vulnerabilidades públicas para evaluar la seguridad

---

## 📎 Material adicional

Se incluye un PDF con el proceso detallado y capturas utilizadas durante la
resolución de la máquina, como apoyo al aprendizaje.

⚠️ Realizado con fines educativos y en un entorno controlado.
