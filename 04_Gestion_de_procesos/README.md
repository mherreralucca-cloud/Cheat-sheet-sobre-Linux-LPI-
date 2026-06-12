# 📂 04 - Gestión de Procesos y Control de Trabajos en Linux

En Linux, todo es un archivo o un proceso. Comprender cómo el kernel asigna recursos, cómo se comunican los procesos entre sí y cómo asegurar su persistencia frente a fallos de red es una habilidad esencial tanto para la administración de infraestructura como para el análisis de seguridad (Blue Team / Red Team).

---

## 1. Conceptos Fundamentales de la Arquitectura

Un proceso es, en esencia, un programa en ejecución. El sistema operativo gestiona estos procesos basándose en dos reglas fundamentales:

* **PID (Process ID):** Todo proceso recibe un identificador numérico único. La información en tiempo real de cada proceso (memoria, estado, ejecutables vinculados) se almacena dinámicamente en el sistema de archivos virtual `/proc`.
* **Jerarquía Padre-Hijo:** La creación de procesos en Linux es estrictamente jerárquica. Todo proceso (hijo) es iniciado inexorablemente por otro proceso preexistente (padre). Si el padre muere, el sistema debe reasignar o destruir a los hijos.

---

## 2. Los 3 Tipos de Procesos

Entender esta clasificación es clave para saber qué buscar durante una auditoría en vivo del servidor:

1. **Normales (Interactivos):** Se ejecutan vinculados a una terminal (TTY/PTS) y bajo el contexto de un usuario específico. **Regla de oro:** Si cierras la terminal, el proceso muere.
2. **Daemons (Servicios):** Corren en segundo plano (*background*) sin salida directa a una terminal. Generalmente se inician junto con el sistema operativo (mediante `systemd`) y se quedan "escuchando" peticiones a través de un puerto de red. Ejemplos: Servidor web Apache (`httpd`), servicio SSH (`sshd`).
3. **Zombies (Defunct):** Son procesos que ya terminaron su ejecución lógica, pero siguen ocupando un lugar en la tabla de procesos. Esto ocurre porque su proceso "padre" no ha leído su señal de finalización (código de salida).

> **🛡️ Perspectiva de Ciberseguridad:** > * **Auditoría de Daemons:** Los atacantes suelen instalar troyanos o *backdoors* que se comportan como Daemons legítimos, ocultándose en segundo plano para escuchar comandos remotos.
> * **Riesgo de Zombies:** Aunque un proceso Zombie no consume CPU, sí consume una entrada en la tabla de procesos. Un ataque local de Denegación de Servicio (DoS), como un *Fork Bomb*, busca agotar deliberadamente esta tabla, impidiendo que el sistema pueda abrir nuevos procesos y congelando el servidor.

---

## 3. Control de Trabajos (Foreground vs. Background)

Cuando ejecutas un comando normal, la terminal queda bloqueada hasta que la tarea finalice (ejecución en primer plano o *foreground*). Para evitar esto y operar en entornos multitarea desde la misma consola, utilizamos el Control de Trabajos:

* **`comando &`**: El *ampersand* al final de un comando (ej. `script_respaldo.sh &`) envía el proceso inmediatamente a ejecutarse en segundo plano (*background*), liberando tu terminal.
* **`jobs`**: Lista todos los trabajos en segundo plano vinculados a tu sesión actual, asignándoles un ID de trabajo (`[1]`, `[2]`, etc.).
* **`fg %1`**: Trae el trabajo número 1 desde el fondo hacia el primer plano (*Foreground*).
* **`bg %1`**: Toma un trabajo pausado y lo reanuda forzadamente en segundo plano (*Background*).

### Señales de Teclado (Signals)
* **`Ctrl + C` (SIGINT):** Envía la señal de "Interrupción". Fuerza la detención y muerte inmediata del proceso en primer plano.
* **`Ctrl + Z` (SIGTSTP):** Envía la señal de "Pausa". Detiene temporalmente el proceso, manteniéndolo en memoria, permitiéndote mandarlo al fondo luego con el comando `bg`.

---

## 4. Escenario practico: Persistencia ante SIGHUP (`nohup`)

**El Escenario:** Estás conectado por SSH a un servidor crítico y lanzas una auditoría de red pesada que tardará horas:
```bash
nmap -p- -T2 localhost &
```
El operador `&` te devuelve el control de la consola, pero hay una trampa de arquitectura: **ese escaneo sigue siendo proceso "hijo" de tu sesión SSH**. Si tu Wi-Fi se corta, o cierras la ventana de la terminal, la sesión SSH muere y le envía una señal **SIGHUP** (Signal Hang Up) a todos sus procesos hijos, matando el escaneo a la mitad.

**La Solución de Infraestructura (`nohup`):**
Para aislar el proceso y volverlo inmune a la desconexión de la terminal, se utiliza el comando `nohup` (No Hang Up). 

```bash
nohup nmap -p- -T2 localhost > escaneo.txt 2>&1 &
```

**Análisis del comando:**
1. `nohup`: Inmuniza el proceso contra el cierre de la terminal.
2. `nmap -p- -T2 localhost`: El comando a ejecutar (auditoría de todos los puertos locales de manera sigilosa).
3. `> escaneo.txt`: Al no haber pantalla, redirigimos el resultado a un archivo de texto.
4. `2>&1`: Redirige cualquier error (canal 2) hacia la misma salida estándar (canal 1).
5. `&`: Lo envía al fondo para recuperar la terminal instantáneamente.

> **🛡️ Perspectiva de Ciberseguridad (Red Team / Blue Team):** `nohup` es el mejor amigo del auditor de seguridad. Permite dejar corriendo *scripts* de recolección de evidencia, escaneos de vulnerabilidades masivos o capturas de tráfico (`tcpdump`) en un servidor remoto, y desconectarse con seguridad sabiendo que el proceso sobrevivirá hasta completar su misión.
