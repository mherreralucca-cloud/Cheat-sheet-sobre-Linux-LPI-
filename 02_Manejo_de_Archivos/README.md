# 📂 02 - Manejo y Filtrado de Archivos en Linux

El dominio fluido de estos comandos en la terminal marca la diferencia entre tardar horas buscando un problema y resolverlo en minutos durante una auditoría o respuesta a incidentes. 

A continuación, se detalla la guía de herramientas fundamentales para la visualización, búsqueda y análisis forense de texto plano.

---

##  1. Visualización Rápida y Concatenación (`cat`, `zcat`)

La línea de comandos permite examinar y procesar archivos de texto plano sin necesidad de abrirlos en un editor interactivo (como nano o vim).

**Comandos Clave y Ejemplos:**
* `cat /etc/passwd`: Muestra todo el contenido del archivo en la salida estándar.
* `cat /etc/passwd /etc/group`: Lee ambos archivos secuencialmente y los une por pantalla.
* `zcat`: Permite leer el contenido de archivos comprimidos sin tener que descomprimirlos previamente en el disco.

> **🛡️ Perspectiva de Ciberseguridad:** Utilizar `cat` es la forma más segura de revisar un archivo de configuración crítico. Al no abrir un editor interactivo, se elimina el riesgo de modificar accidentalmente el archivo o alterar sus metadatos (como la fecha de modificación), lo cual es vital en auditorías forenses.

---

##  2. Paginadores (`more`, `less`)

Cuando los archivos son muy extensos para usar `cat`, se utilizan paginadores que dividen el contenido en pantallas o páginas. `less` es la herramienta más avanzada (y es el paginador por defecto del comando `man`).

**Comandos Clave de Navegación en `less`:**
* `/criterio`: Busca el patrón indicado dentro del texto y salta hacia él.
* `f` y `b`: Avanza (forward) o retrocede (backward) una página entera.
* `g` y `G`: Salta directo al principio (`g`) o al final (`G`) del archivo.
* `q`: Interrumpe el proceso y sale del paginador.

---

##  3. Extremos de Archivos (`head`, `tail`)

Estas herramientas están diseñadas para mostrar exclusivamente las primeras (`head`) o las últimas (`tail`) líneas de un archivo. (Si no se especifica, devuelven 10 líneas por defecto).

**Comandos Clave y Ejemplos:**
* `head -n 20 /var/log/syslog`: Muestra las primeras 20 líneas del archivo.
* `tail -n 50 /var/log/messages`: Imprime las últimas 50 líneas desde el final.
* `tail -f /var/log/auth.log`: Mantiene el archivo abierto y muestra en tiempo real los datos que se van agregando al final.

> **🛡️ Perspectiva de Ciberseguridad:** El parámetro `-f` (follow) es fundamental para el monitoreo en vivo. Permite visualizar ataques de fuerza bruta por SSH en tiempo real o revisar bloqueos de un firewall mientras ocurren.

---

##  4. Estadísticas y Diferencias (`wc`, `diff`)

* **`wc` (Word Count):** Cuenta la cantidad de líneas, palabras y caracteres (bytes) de un texto.
* **`diff`:** Es una herramienta de comparación que examina dos archivos y expone sus diferencias exactas, señalando con `<` y `>` dónde ocurren los cambios.

**Comandos Clave y Ejemplos:**
* `wc -l /etc/passwd`: Devuelve únicamente la cantidad total de líneas.
* `cat /etc/passwd | wc -l`: Usando tuberías (pipes), suministramos la salida de `cat` a `wc`. Nos devuelve solo el número omitiendo el nombre del archivo.
* `diff sshd_config.old sshd_config.new`: Compara ambas configuraciones línea por línea.

> **🛡️ Perspectiva de Ciberseguridad:** `diff` es tu mejor amigo para la auditoría de configuraciones. Te permite verificar exactamente qué cambió en un archivo crítico tras una actualización o sospecha de intrusión.

---

##  5. Búsqueda de Archivos (`locate`, `find`, `whereis`, `which`)

Existen distintas herramientas dependiendo de la precisión y velocidad requerida:
* **`locate`:** Búsquedas rápidas. Lee desde una base de datos interna (`updatedb`). Ej: `locate -i config`.
* **`whereis`:** Busca binarios, páginas del manual y código fuente. Ej: `whereis -b top`.
* **`which`:** Busca únicamente ejecutables dentro de la variable de entorno `$PATH`. Ej: `which ls`.
* **`find`:** Recorre la estructura real del sistema de archivos en vivo para búsquedas granulares y precisas.

**Ejemplos Prácticos con `find`:**
* `find /etc -name '*.conf'`: Busca archivos terminados en `.conf` (las comillas evitan que la shell expanda el asterisco).
* `find /etc -type f -user root -mtime 60`: Busca archivos regulares (`-type f`) de root modificados hace exactamente 60 días.
* `find / -type f -perm 777 -exec chmod 644 {} +`: Ejecuta una acción: busca archivos con permisos totales (777) y les aplica `chmod 644`.

> **🛡️ Perspectiva de Ciberseguridad:** `find` es clave en el Hardening. Buscar archivos con permisos inseguros (777) o ejecutables modificados recientemente por usuarios no autorizados es parte de la rutina diaria de un analista de seguridad.

---

##  6. Filtrado y Búsqueda de Texto (`grep`)

`grep` (Globally Regular Expressions Pattern) busca patrones y devuelve la línea completa donde encuentra la coincidencia. Es la herramienta de filtrado más potente de la terminal.

**Flags más utilizados:**
* `-i`: Ignora la diferencia entre mayúsculas y minúsculas.
* `-r`: Búsqueda recursiva dentro de todos los archivos de un directorio.
* `-v`: Invierte la búsqueda (devuelve las líneas que NO contienen el patrón).
* `-c`: Devuelve únicamente la cantidad (número) de coincidencias.
* `-n`: Imprime el número de la línea al lado del resultado encontrado.

**Ejemplos Prácticos:**
* `grep -i admin /etc/passwd`: Busca la palabra ignorando mayúsculas y minúsculas.
* `grep -v error /var/log/syslog`: Muestra un log limpio, ocultando las líneas de error.

> **🛡️ Perspectiva de Ciberseguridad:** Es la base del análisis forense de logs. Con `grep` podés extraer direcciones IP de atacantes, buscar códigos HTTP específicos (403 o 500) o identificar firmas de malware dentro de archivos sospechosos.

---
Mas informacion en el .pdf adjunto, donde hay una guia practica a modo de ejemplo y una hoja de trucos para siempre tener a mano.
Cualquier recomendacion es bienvenida!

