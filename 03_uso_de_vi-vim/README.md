# 📂 03 - El Editor de Texto Vim: Arquitectura, Modos y Hardening

En el ecosistema Unix/Linux, **Vim (Vi Improved)** no es simplemente un editor de texto; es una herramienta estándar de administración y una competencia crítica para ingenieros de infraestructura, administradores de sistemas y analistas de seguridad. 

Esta guía aborda la filosofía de diseño modal de Vim, su arquitectura interna de gestión de buffers y su aplicación en entornos de alta criticidad y respuesta a incidentes.

---

##  1. Fundamentos Teóricos y Filosofía de Diseño

### El Estándar POSIX y Ubicuidad
De acuerdo con los estándares **POSIX**, cualquier sistema operativo certificado debe incluir el editor `vi` por defecto. Es por esto que en distribuciones minimalistas, contenedores Docker limpios, sistemas empotrados o entornos de recuperación (*Rescue Mode* y *Single-User Mode*), Vim (o Vi) suele ser el único editor disponible. Dominarlo garantiza que podrás operar en cualquier servidor del planeta, independientemente de si tiene interfaz gráfica o conectividad a internet.

### Eficiencia del Teclado e Historia (ADM-3A)
La navegación clásica de Vim (`h`, `j`, `k`, `l` en lugar de las flechas direccionales) no es un capricho de diseño. Históricamente, Bill Joy (creador de Vi) desarrolló el editor en una terminal **Lear Siegler ADM-3A**, cuyo teclado no tenía flechas físicas dedicadas; las flechas estaban impresas sobre las letras H, J, K y L. Hoy en día, esta disposición mantiene una ventaja teórica masiva: el administrador nunca necesita despegar las manos de la "fila de inicio" (*home row*) del teclado, reduciendo el movimiento físico y maximizando la velocidad de edición remota a través de conexiones de red con alta latencia.

---

##  2. Arquitectura Interna: Buffers, Memoria y Archivos Swap (`.swp`)

Entender cómo maneja Vim los datos es vital para no corromper archivos críticos del sistema operativo. 

Cuando abrís un archivo con Vim, el editor **no** modifica el archivo directamente en el disco duro. En su lugar, lee el archivo, asigna un espacio en la memoria RAM volátil y crea un **Buffer**. Todas las ediciones que realizás ocurren en este buffer intermedio. El archivo original en disco permanece intacto hasta que ejecutás explícitamente un comando de escritura (`:w`).

Para proteger la sesión ante un corte de energía, una desconexión de la sesión SSH o la caída del sistema, Vim genera dinámicamente un archivo oculto en el disco llamado `.nombre_del_archivo.swp`. Este archivo actúa como un log de transacciones en tiempo real.

> **🛡️ Perspectiva de Ciberseguridad e Incident Forensics:** Si un servidor se apaga abruptamente durante una respuesta a incidentes, la presencia de un archivo `.swp` te permitirá recuperar el estado exacto de la edición usando `vim -r archivo.conf`. Además, en análisis forense, auditar la existencia de archivos `.swp` residuales en directorios críticos (como `/etc` o `/root`) revela qué configuraciones estuvo alterando un intruso o un administrador antes del colapso del sistema.

---

##  3. El Motor Modal: Operadores, Movimientos y Modos Avanzados

Vim es un editor **gramatical**. Combina **Operadores** (acciones) con **Movimientos** (hacia dónde o qué extensión).

* **Operador `d` (Delete/Cut)** + **Movimiento `w` (Word)** = `dw` (Borra una palabra).
* **Operador `c` (Change)** + **Movimiento `$` (Hasta el final de la línea)** = `c$` (Borra hasta el final y entra en Modo Inserción).

### Los 4 Modos Fundamentales del Motor

Para alternar con precisión entre la administración del entorno y la inserción de datos, debés conocer el ciclo de vida de los modos:

* **1. Modo Normal (Comando):** El estado soberano del editor. Cada tecla es un comando o un atajo macro. Protege al administrador de introducir caracteres erróneos por accidente en archivos de configuración sensibles.
* **2. Modo Inserción:** Activado mediante `i` (insertar antes), `a` (añadir después) u `o` (abrir línea inferior). Transforma el teclado en una máquina de escribir convencional.
* **3. Modo Línea de Comandos (Ex):** Activado presionando `:` desde el Modo Normal. Invoca al intérprete de comandos interno del editor para interactuar con el sistema de archivos, realizar búsquedas complejas o modificar variables del entorno de edición (ej. `:set number` para ver las líneas).
* **4. Modo Visual (Bloque y Selección):** Activado mediante `v` (visual estándar) o `Ctrl + v` (Visual Block). Este último es una herramienta de ingeniería de infraestructura brutal: permite seleccionar columnas verticales de texto plano de forma simultánea.
* Siempre presionamos `ESC` para volver al Modo Normal.

---

## 🛠️ 4. Cheat Sheet Avanzada de Comandos

### Navegación Estructural (Modo Normal)
* `gg`: Salta inmediatamente a la primera línea del archivo.
* `G`: Salta inmediatamente a la última línea del archivo.
* `50G`: Salta directamente a la línea número 50 del buffer.
* `w`: Desplaza el cursor al inicio de la siguiente palabra.
* `b`: Retrocede el cursor al inicio de la palabra anterior.
* `0` (Cero): Mueve el cursor al inicio absoluto de la línea actual.
* `$` (Signo de dólar): Mueve el cursor al final absoluto de la línea actual.

### Edición y Destrucción de Datos (Modo Normal)
* `x`: Elimina el carácter que se encuentra exactamente debajo del cursor.
* `yy`: Copia (*Yank*) la línea actual completa en el registro del editor.
* `dd`: Elimina/Corta la línea actual por completo.
* `d$`: Corta desde la posición del cursor hasta el final de la línea.
* `p`: Pega el contenido del registro una línea abajo del cursor actual.
* `u`: Deshace la última acción (*Undo*), respaldado por el árbol de deshacer histórico de Vim.

### Comandos de Control del Sistema (Modo Última Línea `:`)
* `:w`: Escribe las modificaciones del buffer hacia el almacenamiento físico en disco.
* `:q`: Cierra la sesión del editor (fallará si el buffer difiere del archivo en disco).
* `:wq` o `:x`: Escribe en disco y cierra la sesión de forma limpia en un solo paso.
* `:q!`: Salida forzada. Destruye el buffer en memoria RAM y mantiene el archivo en disco intacto.
* `:%s/antiguo/nuevo/g`: Busca globalmente (`g`) en todo el archivo (`%`) la palabra "antiguo" y la reemplaza por "nuevo".

---

*Como es habitual dejo el .pdf con una guia mas extensa y una parte practica a modo de ejemplo. Para una referencia rápida de comandos, consulte la Cheat Sheet gráfica en esta misma carpeta.*

---

## Feedback y Contribuciones

Este repositorio funciona como una base de conocimiento abierta y en constante expansión. 

* 🤝 **¿Querés contribuir?** Si encontrás un error conceptual,los *Pull Requests* son más que bienvenidos.
* 🐛 **Reportar anomalías:** Si tenés sugerencias o mejoras, sentite libre de abrir un *Issue*.
* ⭐ **Apoyo al proyecto:** Si estos resúmenes y hojas de trucos te resultaron útiles para repasar, ¡dejame una estrella en el repositorio para apoyar el contenido Open Source!

