# 📂 05 - Administración de Usuarios, Permisos y Hardening (IAM)

En el ecosistema Linux, la Gestión de Identidades y Accesos (IAM) es el núcleo de la seguridad del servidor. Un atacante rara vez "hackea" el sistema de cero; en su lugar, busca escalar privilegios explotando usuarios mal configurados, permisos excesivos o archivos huérfanos.

Esta guía documenta los estándares de administración segura de cuentas, la anatomía de los archivos críticos y la aplicación de políticas de *Hardening* avanzadas.

---

## 1. Identidad y Archivos Críticos (Anatomía)

La base de datos de identidades locales no es mágica, es texto plano. El proceso de autenticación moderna se delega a **PAM** (*Pluggable Authentication Modules*), cuyos archivos residen en `/etc/pam.d/`. Sin embargo, la estructura de usuarios recae en dos archivos fundamentales:

### A. `/etc/passwd` (Público)
Es legible por cualquier usuario del sistema. Almacena las propiedades de las cuentas divididas en 7 campos separados por dos puntos (`:`).
Ejemplo: `User1:x:1001:1001:User1 Admin:/home/User1:/bin/bash`
1. **User1:** Nombre de login.
2. **x:** Indica que la contraseña no está aquí, sino ofuscada en `/etc/shadow`.
3. **1001 (UID):** User ID. El `0` es exclusivo de root; del 1 al 999 son para servicios (Daemons); a partir de 1000 son usuarios humanos.
4. **1001 (GID):** Group ID principal.
5. **GECOS:** Comentarios y datos del usuario.
6. **Directorio Home:** `/home/User1`.
7. **Shell:** `/bin/bash` (o `/usr/sbin/nologin` si es una cuenta de servicio para impedir acceso interactivo).

### B. `/etc/shadow` (Estrictamente Protegido)
Solo legible por root. Contiene las contraseñas encriptadas (generalmente bajo el algoritmo SHA-512) y las políticas de expiración (edad de la clave).

> **🛡️ Perspectiva de Ciberseguridad:** Una de las primeras revisiones en una auditoría es buscar la ausencia de la `x` en `/etc/passwd`. Si un hash de contraseña se vuelca allí, cualquier usuario sin privilegios podría copiarlo e intentar romperlo por fuerza bruta (*Cracking* offline con John The Ripper o Hashcat).

---

## 2. Gestión de Cuentas y Políticas

Crear y modificar usuarios requiere precaución para no otorgar permisos de más ni romper accesos existentes.

* **`usermod -a -G grupo usuario`:** Añade a un usuario a un grupo suplementario. El flag `-a` (Append) es **crítico**; si se omite, el usuario será eliminado de todos sus otros grupos, lo cual puede causar la pérdida de acceso `sudo` o la caída de servicios.
* **`userdel -r usuario`:** Elimina al usuario y, gracias a la `-r`, borra también su directorio `/home` y su buzón de correo.
* **`chage -d 0 usuario`:** Modifica la edad de la contraseña. Ponerla en `0` fuerza al usuario a cambiar su contraseña inmediatamente en el próximo inicio de sesión.

> **🛡️ Perspectiva de Ciberseguridad:** Si eliminas un usuario usando `userdel` sin el parámetro `-r`, sus archivos personales quedan en el sistema. El dueño de esos archivos pasará a ser un número de UID inexistente (Ej: 1005). Esto crea un **"Archivo Huérfano"**, representando un riesgo de seguridad severo si ese UID se reasigna a un nuevo usuario en el futuro.

---

## 3. Sistema de Permisos Base y Máscaras (`umask`)

Linux utiliza un modelo de permisos de lectura (`r=4`), escritura (`w=2`) y ejecución (`x=1`). 
*(Nota: En los directorios, el permiso `x` es obligatorio para poder hacer `cd` y entrar en ellos).*

* **`chmod`**: Cambia los permisos (Ej. `chmod 750 archivo` otorga lectura, escritura y ejecución al dueño, lectura/ejecución al grupo, y nada a los demás).
* **`chown`**: Cambia el propietario o grupo del archivo.

### El Filtro `umask`
Es el valor que "filtra" o resta permisos automáticamente al crear un archivo o carpeta nueva, dictando el nivel de seguridad por defecto. Por seguridad, los archivos nuevos nunca nacen con permisos de ejecución.

---

## 4. Permisos Especiales (SUID y SGID)

Cuando el esquema tradicional (rwx) no es suficiente, entran en juego los permisos especiales, los cuales son extremadamente potentes y peligrosos:

* **SUID (Set User ID):** Permite que un ejecutable corra con los privilegios del **dueño** del archivo, sin importar quién lo ejecute. (Ejemplo legítimo: El comando `passwd` necesita SUID para que un usuario normal pueda editar el archivo `/etc/shadow` y cambiar su propia clave temporalmente como root).
* **SGID (Set Group ID):** Ejecuta el archivo con los privilegios del **grupo**. Si se aplica a un directorio, fuerza a que todos los archivos nuevos creados adentro hereden el grupo del directorio, facilitando el trabajo colaborativo.

> **🛡️ Perspectiva de Ciberseguridad (Escalada de Privilegios):** Lo primero que hace un atacante al vulnerar a un usuario normal es buscar binarios mal configurados ejecutando `find / -perm -4000 2>/dev/null`. Si el atacante encuentra un binario como `nmap`, `vim` o `python` con el bit SUID activado y propiedad de root, puede utilizarlo para ejecutar una shell y convertirse en superusuario instantáneamente (Técnicas documentadas en *GTFOBins*).

---

## 5. Hardening Defensivo: Atributos Inmutables (`chattr`)

Incluso el usuario `root` tiene límites si se configuran atributos extendidos en el sistema de archivos. Esta es la última línea de defensa contra administradores negligentes o ataques avanzados (como Ransomware).

* **`chattr +i archivo` (Immutable):** Vuelve al archivo completamente inmutable. Ni siquiera el usuario `root` puede borrarlo, renombrarlo, escribir en él o crear un enlace duro. Ideal para blindar `/etc/passwd`.
* **`chattr +a archivo` (Append Only):** El archivo solo permite que se le agregue contenido al final, pero prohíbe terminantemente borrar o modificar el texto existente. 

> **🛡️ Perspectiva de Ciberseguridad (Blue Team):** Aplicar `chattr +a /var/log/auth.log` es un seguro de vida invaluable para la respuesta a incidentes. El log seguirá creciendo registrando accesos, pero si un intruso obtiene acceso root e intenta borrar su rastro (`rm -rf /var/log/*`), el sistema se lo impedirá, preservando la evidencia forense intacta.

---
Mas informacion en el .pdf adjunto, donde hay una guia practica a modo de ejemplo. Ademas una hoja de trucos (.png adjunto) para siempre tener a mano.
---

## Feedback y Contribuciones

Este repositorio funciona como una base de conocimiento abierta y en constante expansión. 

* 🤝 **¿Querés contribuir?** Si encontrás un error conceptual,los *Pull Requests* son más que bienvenidos.
* 🐛 **Reportar anomalías:** Si tenés sugerencias o mejoras, sentite libre de abrir un *Issue*.
* ⭐ **Apoyo al proyecto:** Si estos resúmenes y hojas de trucos te resultaron útiles para repasar, ¡dejame una estrella en el repositorio para apoyar el contenido Open Source!

