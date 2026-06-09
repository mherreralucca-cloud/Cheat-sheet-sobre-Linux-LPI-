<p align="center">
  <a href="https://github.com/mherreralucca-cloud/Cheat-sheet-sobre-Linux-LPI-">
    <img src="images/may.png" width="160" height="160">
  </a>
</p>



# Repositorio de Apuntes: Administrador de Redes Linux con Orientación en Ciberseguridad 🐧

¡Hola! 👋🤠 Bienvenido a mi base de conocimiento técnico y repositorio de "Cheat Sheets" (Hojas de Trucos). 

Como **Técnico de Redes**, sé que la velocidad y precisión en la línea de comandos marcan la diferencia entre resolver un incidente en minutos o en horas. Este repositorio documenta mi proceso de aprendizaje y especialización continua en entornos Linux y Ciberseguridad.

##  Sobre este Proyecto

Este repositorio nace como complemento práctico de la **Certificacion de Administrador de Redes Linux con Orientación en Ciberseguridad**, dictado en conjunto por el **LPI (Linux Professional Institute)**, **Linux College** y **UTN**.

 **Base Teórica y Referencia (LPI):** Además del material de la diplomatura, el contenido y los resúmenes de este repositorio están fuertemente alineados y comparten información con el libro **"How Linux Works, 3rd Edition"**. Dado que las certificaciones del LPI se basan en gran medida en la estructura de este libro, estos apuntes garantizan el seguimiento de los estándares más rigurosos de la industria.

---
## 🚀 Cómo navegar por este repositorio
Dentro de cada carpeta encontrarás:
1. Un archivo **`README.md`** con la síntesis.
2. El archivo **`.pdf`** original adjunto como informe completo, que incluye el marco teórico, capturas de pantalla del entorno de pruebas (terminal) y una cheat sheet para tener a mano.
   
## 📂 Índice de Contenidos

Los apuntes están divididos por temáticas clave. En cada carpeta encontrarás un resumen  con enfoque en ciberseguridad y hardening, junto con una infografía visual (Cheat Sheet) de resumen rápido:

* 📁 **[01_Comandos_Basicos](./01_Comandos_Basicos/)**

  * Gestión del sistema de archivos, almacenamiento y manipulación de directorios.
  * *Herramientas:* `ls`, `du`, `df`, `cp`, `mv`, `rm`, `history`.
  * *Enfoque de Seguridad:* Monitoreo para prevención de DoS, técnicas de *Timestomping* y análisis del historial de la terminal.

* 📁 **[02_Manejo_de_Archivos](./02_Manejo_de_Archivos/)**
  * Visualización, filtrado y búsqueda.
  * *Herramientas:* `cat`, `less`, `tail`, `grep`, `find`, `wc`, `diff`.
  * *Enfoque de Seguridad:* Auditoría de logs en vivo, búsqueda de patrones de malware y revisión de cambios no autorizados.

* 📁 **[03_Editor_Vim](./03_Editor_Vim/)**
  * Guía de supervivencia en el editor modal por excelencia.
  * *Herramientas:* Modos Normal, Inserción y Última Línea.
  * *Enfoque de Seguridad:* Edición rápida y segura de configuraciones críticas (como `sshd_config` o `iptables`) en servidores sin interfaz gráfica.

* 📁 **[04_Gestion_Procesos](./04_Gestion_Procesos/)**
  * Clasificación de Daemons y Zombies, y control de trabajos.
  * *Herramientas:* Operador `&`, `jobs`, `nohup`, `kill`, `killall`.
  * *Enfoque de Seguridad:* Threat Hunting básico y persistencia de escaneos/auditorías frente a desconexiones (`SIGHUP`).

* 📁 **[05_Identidad_y_Accesos](./05_Identidad_y_Accesos/)**
  * El núcleo del Hardening en Linux (IAM local).
  * *Herramientas:* `/etc/passwd`, `/etc/shadow`, `usermod`, `chage`, `chmod`, `chown`, ACLs, `chattr`.
  * *Enfoque de Seguridad:* Auditoría de permisos SUID/SGID, ciclo de vida de contraseñas y atributos inmutables (`+i`, `+a`) para proteger evidencia forense de ataques Ransomware.

Esto es solo el principio, seguire actualizando este repositorio constantemente!!
---

## 🤝 Contacto

Siéntete libre de utilizar estos apuntes y Cheat Sheets como material de consulta rápida para tu día a día en la terminal.

🔗 **Conectemos en LinkedIn:** https://www.linkedin.com/in/lucca-martinenghi-it/
