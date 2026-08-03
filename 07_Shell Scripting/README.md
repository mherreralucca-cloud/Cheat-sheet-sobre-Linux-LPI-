# 📂 Shell Scripting y Automatización en Bash

Escribir scripts en Bash no solo permite mandar comandos en fila; puede crear herramientas para no repetir tareas a mano en un servidor. Desde crear usuarios en lote hasta filtrar logs o correr mantenimientos de forma automática, usar scripts bien armados ahorra tiempo y reduce los errores típicos de tipear rápido en la consola.

---

## 1. Estructura y Ejecución de un Script

Para que el sistema interprete un archivo de texto como un programa ejecutables, hay que seguir tres pasos básicos:

### A. El Shebang (`#!/bin/bash`)
Es la primera línea obligatoria de cualquier script. Le indica al kernel de Linux qué programa debe procesar el código que viene abajo.
```bash
#!/bin/bash
echo "Iniciando tarea de mantenimiento..."
```

### B. Permisos de Ejecución
Por seguridad, los archivos nuevos se crean sin permiso de ejecución. Se habilita con `chmod`:
```bash
chmod +x mi_script.sh
./mi_script.sh
```

### C. Parámetros Posicionales (Pasar datos por consola)
Bash captura automáticamente los argumentos que le pasamos al script al momento de ejecutarlo:
* `$0`: Nombre del script que se está corriendo.
* `$1`, `$2`, `$3`: Primer, segundo y tercer argumento ingresado.
* `$#`: Cantidad total de argumentos pasados.
* `$*`: Lista completa de argumentos.

```bash
#!/bin/bash
# Uso: ./bloquear_ip.sh 192.168.1.50
IP_OBJETIVO=$1
echo "Aplicando regla para bloquear a la IP: $IP_OBJETIVO"
```

---

## 2. Control de Flujo: Encadenamiento de Comandos

En scripting necesitás controlar si un comando debe correr dependiendo de cómo terminó el anterior. Cada comando en Linux devuelve un estado de salida (*exit status*): `0` si salió bien, y un número mayor a `0` si dio error.

* **Punto y coma (`;`):** Corre un comando tras otro en secuencia, sin importar si el anterior falló o no.
  ```bash
  cd /tmp ; ls -l
  ```
* **Operador AND (`&&`):** Corre el segundo comando **únicamente si el primero terminó bien** (código 0).
  ```bash
  apt update && apt upgrade -y
  ```
* **Operador OR (`||`):** Corre el segundo comando **únicamente si el primero falló** (código distinto de 0). Se usa mucho para manejo de errores.
  ```bash
  ping -c 1 10.0.0.1 || echo "Servidor fuera de línea, notificando..."
  ```

---

## 3. Estructuras de Decisión y Bucles

### A. Menús con `case`
Cuando tenés múltiples opciones, usar un bloque `case` evita llenar el código de condicionales `if` juntos difíciles de leer.

```bash
read -p "Seleccione una opción [1-3]: " OPCION

case $OPCION in
  1)
    echo "Iniciando respaldo de la base de datos..."
    ;;
  2)
    echo "Reiniciando servicio web..."
    ;;
  3)
    echo "Saliendo del script."
    ;;
  *)
    echo "Opción no válida."
    ;;
esac
```

### B. Bucles `while` leyendo archivos de entrada
Una técnica muy usada es tomar un archivo de texto con datos (como una lista de IPs o de usuarios) y procesarlo línea por línea.

```bash
#!/bin/bash
LISTA_USUARIOS="usuarios.txt"

# Lee el archivo línea por línea usando redirección
while read USUARIO; do
    echo "Deshabilitando acceso a: $USUARIO"
    # usermod -s /bin/false $USUARIO
done < $LISTA_USUARIOS
```

---

## 4. Funciones y Reutilización de Código

Las funciones te permiten empaquetar bloques de código para llamarlos en distintas partes del script sin andar copiando y pegando lo mismo.

```bash
# Declaración de la función
ReportarError() {
    echo " [ERROR] Falló la operación en la fecha: $(date)"
}

# Invocación
if ! cd /var/www/html; then
    ReportarError
    exit 1
fi
```

---

## 5. Perfiles de Shell y Hardening de Accesos

### Archivos de Configuración de la Shell
Cuando abrís una consola, Bash lee archivos de configuración para cargar variables de entorno, rutas y alias. La precedencia en sesiones de inicio (*login shells*) es:
1. `~/.bash_profile` (Toma prioridad).
2. `~/.bash_login`
3. `~/.profile`
4. `~/.bashrc` (Se ejecuta en consolas interactivas sin login directo).

> **Tip (`source` o `. `):** Si querés ejecutar un script para que sus variables o alias queden cargados en tu terminal actual (en lugar de correr en una subshell y borrarse al terminar), tenés que ejecutarlo usando `source mi_script.sh` o `. mi_script.sh`.

### Seguridad en Cuentas y Bloqueos de Sistema
* **Shell `/bin/false`:** Cuando creás cuentas de sistema destinadas exclusivamente a correr servicios (como Apache, MySQL o un daemon), hay que asignarles `/bin/false` o `/usr/sbin/nologin` como shell por defecto. Esto evita que alguien pueda abrir una terminal interactiva si la cuenta se llega a comprometer.
* **Archivo `/etc/nologin`:** Si creás este archivo en el servidor (podés escribirle un mensaje adentro), el sistema frena automáticamente cualquier intento de inicio de sesión de usuarios comunes, dejando entrar únicamente a `root`. Es muy útil para tareas de mantenimiento urgente o aislamiento en caso de incidente.

---
Como es habitual dejo el .pdf con una guia mas extensa. Para una referencia rápida de comandos, consulte la Cheat Sheet gráfica en esta misma carpeta.
---

## Feedback y Contribuciones

Este repositorio funciona como una base de conocimiento abierta y en constante expansión. 

* 🤝 **¿Querés contribuir?** Si encontrás un error conceptual,los *Pull Requests* son más que bienvenidos.
* 🐛 **Reportar anomalías:** Si tenés sugerencias o mejoras, sentite libre de abrir un *Issue*.
* ⭐ **Apoyo al proyecto:** Si estos resúmenes y hojas de trucos te resultaron útiles para repasar, ¡dejame una estrella en el repositorio para apoyar el contenido Open Source!
  
