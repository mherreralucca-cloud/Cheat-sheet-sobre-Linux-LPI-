# 📂 01 - Comandos Básicos y Gestión del Sistema

Estos comandos son esenciales porque la terminal es el motor central de Linux. A diferencia de otros sistemas operativos, la línea de comandos en Linux no es un accesorio, sino la forma más directa y potente de controlar tu computadora.

---

##  1. Listado y Detalles (`ls`)

La salida del comando `ls` lista el contenido de los directorios y permite distinguir visualmente los objetos mediante colores. Los archivos que empiezan con un punto (.) están ocultos y suelen usarse para configuraciones.

**Comandos Clave:**
* `ls -a`: Muestra todos los archivos, incluyendo los ocultos.
* `ls -l`: Muestra información detallada (permisos, enlaces, dueño, grupo, tamaño en bytes, fecha de modificación).
* `ls -lh`: Al agregar la h (human-readable), los tamaños se muestran en unidades más amigables (KB, MB, etc.).
* `ls -R`: Lista el contenido del directorio y sus subdirectorios de forma recursiva.

> **🛡️ Perspectiva de Ciberseguridad:** Revisar archivos ocultos con `ls -a` es el paso cero en auditorías e investigaciones, Los atacantes frecuentemente ocultan sus herramientas maliciosas, scripts o carpetas de exfiltración de datos simplemente anteponiendo un punto a sus nombres.

---

## 2. Gestión de Espacio en Disco (`du` y `df`)

Ambas herramientas supervisan el almacenamiento, pero con enfoques distintos. `du` evalúa qué espacio físico ocupan uno o varios archivos/directorios puntuales. `df` permite ver el panorama general: cuánto espacio libre y usado hay por cada partición o volumen lógico montado.

**Comandos Clave:**
* `du -ch`: Muestra el uso de disco de manera legible y agrega una línea con el total.
* `df -h`: Despliega el estado de los sistemas de archivos de manera legible.

> **🛡️ Perspectiva de Ciberseguridad:** Un vector de ataque local común de denegación de servicio (DoS) consiste en llenar deliberadamente una partición (como `/var/log` inundando los registros). Monitorear periódicamente los porcentajes de uso con `df` previene la caída de servicios críticos.

---

##  3. El Historial de la Terminal (`history`)

Este comando expone y enumera cuáles fueron las instrucciones ingresadas por el usuario leyendo el archivo oculto `.bash_history` ubicado en su directorio home. 

**Ejemplos Prácticos:**
* `history`: Devuelve la lista de los últimos comandos ejecutados.
* `!382`: Vuelve a ejecutar inmediatamente el comando asociado a ese número sin tener que tipear de nuevo.

> **🛡️ Perspectiva de Ciberseguridad:** El archivo `.bash_history` es una mina de oro en el análisis forense. Si un atacante logra comprometer una cuenta, el historial revelará qué archivos modificó o qué privilegios intentó escalar.

---

##  4. Manipulación de Directorios y Archivos

Herramientas de gestión diaria para estructurar el file system, copiar datos, moverlos o eliminarlos.

* **`mkdir`**: `mkdir -p` crea una estructura completa de carpetas anidadas en un solo paso.
* **`rmdir`**: Elimina directorios, pero solo si están completamente vacíos.
* **`cp`**: `cp -a` hace una copia exacta. Esta opción preserva permisos, enlaces y copia subdirectorios tal cual.
* **`mv`**: Mueve o renombra archivos, y sobrescribe el destino si este ya existe.
* **`touch`**: Crea un archivo vacío si no existe, o actualiza su fecha de última modificación si ya existe.
* **`rm`**: Borra archivos sin posibilidad de recuperarlos. `rm -ri` borra de forma recursiva pidiendo confirmación interactiva.

> **🛡️ Perspectiva de Ciberseguridad:**
> * **Prevención:** La opción `-i` en `rm` previene accidentes catastróficos que puedan comprometer la disponibilidad del servidor.
> * **Respaldos:** Copiar configuraciones con `cp -a` es seguro ya que no altera la estructura de permisos y propietarios original.
> * **Timestomping:** La propiedad del comando `touch` para alterar la fecha de los archivos es una técnica que utilizan los atacantes para ocultar malware.

---

##  5. Enlaces (`ln`)

Permite crear referencias (links) a otros recursos.
* **Hard links (Duros):** Enlazan directamente al mismo inodo y bloques del archivo. No pueden apuntar a carpetas ni salir de la partición actual.
* **Soft links (Simbólicos):** Representan un enlace a otro inodo, como un acceso directo. Útiles para apuntar a versiones actualizadas de un programa. Se crean con `ln -s`.

> **🛡️ Perspectiva de Ciberseguridad:** Los enlaces simbólicos rotos pueden ser un indicador temprano de que dependencias críticas del sistema fueron eliminadas o comprometidas.

---

##  6. Ciclo de Energía

Estos comandos permiten manejar el ciclo de energía de manera abrupta o controlada.
* **Apagar:** `poweroff`, `halt`, `shutdown -h` o `systemctl poweroff`.
* **Reiniciar:** `reboot`, `shutdown -r` o `systemctl reboot`.

> **🛡️ Perspectiva de Ciberseguridad:** El comando `shutdown` es la herramienta preferida frente a comandos abruptos como `halt`. Impide nuevos logins mientras los servicios se detienen limpiamente, evitando la corrupción de bases de datos o la pérdida de registros de auditoría en memoria.

---

##  7. Exploración del Sistema y Rutas

* **Rutas:** Un punto solo (`.`) representa el Directorio Actual de Trabajo. Dos puntos (`..`) representa el Directorio Padre. Una ruta absoluta se define comenzando siempre desde la raíz (`/`).
* **`/home`**: Contiene los datos personales y las configuraciones específicas de cada usuario.
* **`/etc`**: Contiene todos los archivos de configuración globales del sistema y de los servicios.
* **`/usr/bin`**: Contiene la inmensa mayoría de los binarios (ejecutables) y comandos estándar preinstalados.
* **`/tmp`**: Directorio de archivos temporales donde cualquier programa o usuario puede escribir para guardar datos transitorios.
* **`/var/log`**: Contiene los registros (logs).

---
Mas informacion en el .pdf adjunto, donde hay una guia practica a modo de ejemplo y una hoja de trucos para siempre tener a mano.
Cualquier recomendacion es bienvenida!
