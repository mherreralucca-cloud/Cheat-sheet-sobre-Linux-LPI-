# 📂 06 - Administración de Almacenamiento, File Systems y Hardening de Montajes

En la arquitectura Linux, la máxima corporativa *"todo es un archivo"* se sostiene gracias al sistema de almacenamiento. Comprender cómo el Kernel prepara los dispositivos físicos, cómo los mapea en un árbol lógico global y cómo aplicar directivas de montaje restrictivas es importante para diseñar infraestructuras resilientes y defendibles ante intrusiones.

---

## 1. La Filosofía del Almacenamiento: El Árbol Único y VFS

A diferencia de sistemas operativos comerciales que asignan letras independientes a cada unidad física (como `C:`, `D:` o `E:`), Linux unifica absolutamente todo el almacenamiento bajo un único árbol lógico que nace en el directorio raíz (`/`).

Para interactuar con un disco, partición o recurso de red, el Administrador de Sistemas debe asociar de forma obligatoria ese bloque físico a un directorio existente en el sistema. Este proceso se denomina **Montaje**, y el directorio receptor actúa como **Punto de Montaje**.

### La Capa VFS (Virtual File System)
El usuario y las aplicaciones nunca hablan directamente con el hardware del disco rígido. El Kernel de Linux implementa una capa de abstracción intermedia llamada **VFS**. 

El VFS traduce las llamadas estándar del sistema (como abrir, leer o escribir un archivo) al idioma específico del sistema de archivos en el que está formateado el disco (ej. EXT4, XFS, BTRFS). Esto permite que navegues transparentemente por las carpetas sin que te importe si los datos residen en un disco mecánico local, un SSD NVMe de última generación o un recurso compartido en la nube.

---

## 2. Anatomía del Almacenamiento: Dispositivos, Particionado y Formateo

Para que un bloque de almacenamiento sea utilizable por el sistema operativo, debe atravesar tres fases estrictas de preparación:

```
[Hardware Crudo] -----------> [División Lógica] -----------> [Estructura de Índice]
/dev/sda o /dev/nvme0n1        Particiones (fdisk)             Sistemas de Archivos (mkfs)
```

### Capa A: Identificación en el Espacio `/dev`
Linux representa los dispositivos de almacenamiento físicos como archivos especiales de bloques dentro del directorio `/dev`:
* **Discos SATA / SAS tradicionales:** Se nombran secuencialmente como `/dev/sda`, `/dev/sdb`, `/dev/sdc`.
* **Discos Sólidos M.2 NVMe modernos:** Utilizan el controlador nativo del bus PCIe, mapeándose como `/dev/nvme0n1`, `/dev/nvme1n1`. El kernel añade una `p` para denotar sus particiones internas (ej: `/dev/nvme0n1p1`).

### Capa B: Particionado Lógico (`fdisk` y `parted`)
Las particiones permiten dividir un disco físico en zonas independientes aisladas.
* `fdisk`: Herramienta clásica interactiva por comandos para gestionar tablas de particiones MBR (discos de hasta 2TB).
* `parted`: Herramienta avanzada requerida para inicializar discos modernos con tablas GPT (GUID Partition Table), rompiendo la barrera de los 2TB y permitiendo redundancia en los datos de la tabla de particiones.

### Capa C: Creación del Sistema de Archivos / Formateo (`mkfs`)
Una partición cruda no puede almacenar datos de forma ordenada; necesita que se le "dibuje" un sistema de archivos que actúe como índice estructural.
* **EXT4 (`mkfs.ext4`):** El estándar histórico de Linux. Excelente balance de velocidad y compatibilidad. Utiliza *Journaling* (un registro de transacciones integrado) para evitar la corrupción de archivos si el servidor sufre un apagado abrupto o un corte de energía.
* **XFS (`mkfs.xfs`):** El sistema de archivos por defecto en distribuciones empresariales como Red Hat (RHEL) o Rocky Linux. Diseñado desde cero para manejar transferencias masivas de datos en paralelo, buffers gigantescos y archivos de escala de Terabytes (ideal para servidores de bases de datos de alta demanda).

---

## 3. Persistencia y Hardening de Montajes (`/etc/fstab`)

Cuando montás una partición manualmente con el comando `mount`, el acceso se pierde al reiniciar el servidor. Para lograr persistencia, la configuración debe declararse en el archivo crítico `/etc/fstab` (File System Table).

### La Trampa de los Nombres de Dispositivo y el uso de UUIDs
Históricamente, los administradores mapeaban los discos en el `fstab` usando su nombre de ruta directo (ej: `/dev/sdb1`). **Esto es un error grave de infraestructura.** Si desconectás un disco para mantenimiento, cambiás un cable de puerto en la placa madre o migrás la máquina virtual, el Kernel puede alterar el orden de detección al arrancar, asignándole `/dev/sdb1` a un disco completamente diferente. Esto causaría una falla de arranque o corrupción de datos.

La solución estándar de la industria es obtener el identificador único universal de la partición ejecutando el comando `blkid` y colocandolo en el archivo:
```text
UUID=a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6  /datos  ext4  defaults  0  2
```

### Hardening Avanzado de Particiones (Blindaje del Sistema)
El archivo `/etc/fstab` permite pasar flags de seguridad personalizadas en el cuarto campo (reemplazando u omitiendo la directiva por defecto `defaults`). Aplicar estas banderas es una de las estrategias de robustecimiento más efectivas de la ciberseguridad defensiva:

* **`noexec`:** Prohíbe terminantemente la ejecución de cualquier binario o script dentro de esa partición. Si un atacante logra subir un troyano o un script malicioso a una carpeta como `/tmp` o `/var/www/uploads`, el sistema operativo se negará a ejecutarlo, neutralizando la amenaza.
* **`nosuid`:** Desactiva los bits SUID y SGID en toda la partición. Impide que un usuario malicioso intente colar un binario modificado con permisos de root para lograr una escalada de privilegios local.
* **`nodev`:** Bloquea la interpretación de archivos especiales de dispositivos (carácter o bloque). Evita que un atacante con accesos de escritura cree un dispositivo falso apuntando directamente a la memoria RAM o al disco principal para saltarse el control del File System.

> **Configuración recomendada para `/tmp` en producción:**
> `UUID=...  /tmp  ext4  defaults,noexec,nosuid,nodev  0  2`

---

## 4. Monitoreo y Gestión de Capacidades

Un disco duro lleno al 100% degrada el rendimiento del servidor y puede causar una denegación de servicio (DoS) local al impedir que servicios críticos (como bases de datos o servidores web) escriban sus archivos temporales o logs de transacciones.

* **`df -h` (Disk Free):** Expone el espacio libre y ocupado de todos los sistemas de archivos montados. El flag `-h` (Human Readable) traduce los bloques a Gigabytes y Megabytes legibles.
* **`du -sh /ruta` (Disk Usage):** Analiza un directorio específico de forma recursiva y resume (`-s`) el tamaño total acumulado en disco de esa carpeta.

### El Misterio del Espacio Desaparecido
Un escenario clásico para un SysAdmin: Ejecutás `df -h` y te dice que el disco está al 100% de ocupación. Sin embargo, corrés `du -sh /*` en la raíz para buscar la carpeta culpable y la sumatoria te dice que solo tenés ocupado el 40%. **¿Dónde está el espacio restante?**

* **La Explicación Interna de Linux:** Cuando un proceso o aplicación (ej: un script mal configurado o un servidor web comprometido) está escribiendo activamente en un archivo de log masivo, y un administrador borra ese archivo usando el comando `rm`, el índice visible del archivo desaparece del File System, pero el espacio físico **no se libera**. 
* Linux mantiene el espacio reservado e invisible en disco porque el descriptor de archivo (*File Descriptor*) sigue abierto y el proceso sigue inyectando datos en memoria. La solución forense para identificar estos archivos fantasmas es ejecutar:
```bash
lsof +L1
```
Este comando lista los archivos abiertos con un conteo de enlaces igual a cero, permitiéndote matar el PID del proceso causante para recuperar el almacenamiento instantáneamente sin necesidad de reiniciar el servidor.

---

## 5. Empaquetado, Compresión y Resguardos Seguros (`tar`)

El resguardo de configuraciones y datos es la última línea de defensa ante incidentes. En Linux, la herramienta estándar es `tar` (Tape Archiver), combinada con algoritmos de compresión avanzados.

### Comandos de Operación
* **Crear un Tarball (`tar -cvf backup.tar /etc`):** Empaqueta la carpeta `/etc` completa en un único archivo, preservando los permisos de usuario y la estructura de directorios, pero sin aplicar compresión.
* **Desempaquetar (`tar -xvf backup.tar -C /restore`):** Extrae los datos. El flag `-C` es crítico, ya que redirige la descompresión hacia un directorio seguro de destino, evitando sobreescribir archivos vivos por error.

### Tabla Comparativa de Algoritmos de Compresión

| Operador Tar | Extensión | Algoritmo | Velocidad de Procesamiento | Ratio de Compresión (Ahorro de espacio) | Uso de CPU/RAM |
| :---: | :---: | :---: | :---: | :---: | :---: |
| `-z` | `.tar.gz` | **Gzip** | Ultra Rápido | Moderado / Estándar | Muy Bajo (Ideal para tareas repetitivas) |
| `-j` | `.tar.bz2`| **Bzip2** | Moderado | Alto | Medio |
| `-J` | `.tar.xz` | **XZ** | Lento | Máximo / Extremo | Muy Alto (Ideal para backups históricos) |

```bash
# Ejemplo de Resguardo Máximo (Uso de XZ):
tar -cJvf respaldo_seguro.tar.xz /var/www/html
```
*Se recomiendan verificar siempre el almacenamiento de destino usando `df -h` antes de lanzar un proceso pesado de compresión XZ para evitar colapsar la swap o el espacio de logs durante la transacción.*


