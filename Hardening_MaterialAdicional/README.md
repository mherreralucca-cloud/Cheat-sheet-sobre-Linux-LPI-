# 📂 Teoría de Hardening y Seguridad en Capa 7 (OWASP)

**Nota Arquitectónica:** *Aunque este módulo se aleja temporalmente de los comandos específicos de la terminal de Linux, los conceptos aquí documentados impactan de forma colateral y directa en el sistema operativo. El Hardening y la seguridad en Capa de Aplicación son principios universales y obligatorios en cualquier arquitectura de software y hardware (Defensa en Profundidad). Un servidor Linux perfectamente asegurado en su base es inútil si la aplicación web que hospeda es vulnerable.*

---

## 1. El Concepto Integral de Hardening

El **Hardening** (Robustecimiento) es el proceso técnico de blindar una infraestructura reduciendo sistemáticamente su **superficie de ataque**. La superficie de ataque abarca el conjunto total de vulnerabilidades, puertos abiertos, protocolos obsoletos o credenciales por defecto que un atacante puede explotar. 

En ciberseguridad, la regla de oro es: **"Menos es más"**. A menor cantidad de puertas lógicas habilitadas, menor es el riesgo de intrusión. 

### Los 5 Pilares Operativos
Para asegurar un servidor desde su núcleo, la defensa se estructura en cinco frentes:
1. **Gestión de Usuarios:** Implementar el Principio de Mínimo Privilegio (PoLP). Ningún servicio o usuario estándar debe correr bajo la cuenta de `Administrador` o `root` para sus tareas diarias.
2. **Actualizaciones y Parches:** Mitigar la explotación de vulnerabilidades conocidas (CVEs) manteniendo el sistema operativo y las aplicaciones rigurosamente actualizadas.
3. **Cierre de Puertos y Servicios:** Apagar todo servicio no crítico (ej. servidores de impresión locales, FTP en texto plano). Cada servicio ejecutándose es un vector de ataque potencial.
4. **Configuración de Red (Firewall):** Implementar una postura *Deny by Default* (Denegar por defecto). El cortafuegos debe bloquear absolutamente todo el tráfico entrante y solo abrir los puertos estrictamente necesarios para la operación (ej. 80 o 443 para un web server).
5. **Auditoría y Logs:** Revisión y monitorización centralizada de registros para detectar comportamientos anómalos, como ráfagas de inicios de sesión fallidos desde IPs desconocidas.

---

## 2. Dominios y Prácticas de Hardening 

En ciberseguridad no se inventan estas prácticas de cero, sino que se basan en estándares internacionales de la industria. El más famoso de todos son los CIS Benchmarks (del Center for Internet Security).

La fortificación debe ser holística y aplicarse por capas en toda la infraestructura:

### A. Hardening del Sistema Operativo
* **Parcheo Automático:** Usar herramientas para la aplicación programada de actualizaciones de seguridad del SO.
* **Limpieza de Componentes:** Eliminar software, compiladores (gcc), controladores, bibliotecas o servicios innecesarios en entornos de producción.
* **Refuerzo del File System:** Robustecer los permisos de registros (`/var/log`) y cifrar el almacenamiento local (Data at Rest).

### B. Hardening de Red
* **Filtrado:** Configuración estricta de firewalls e implementación de Listas de Control de Acceso (ACLs).
* **Cifrado de Tránsito:** Cifrar todo el tráfico de red (VPNs, TLS) para mitigar el ingreso de malware o la interceptación por parte de piratas informáticos (*Man-in-the-Middle*).
* **Bloqueo Físico/Lógico:** Bloquear cualquier punto de red, puerto físico o interfaz virtual que no esté siendo utilizado.

### C. Hardening de Base de Datos
* **Saneamiento de Cuentas:** Eliminar inmediatamente las cuentas predeterminadas o no utilizadas.
* **Control de Privilegios:** Crear restricciones granulares de administración sobre qué tablas pueden leer o escribir ciertos usuarios (separación de roles).
* **Criptografía:** Crear sistemas de encriptación tanto para el almacenamiento en disco como para la transferencia de datos hacia la aplicación.

### D. Hardening de Aplicaciones
* **Credenciales:** Reemplazar todas las contraseñas predeterminadas de software de terceros por contraseñas complejas y únicas.
* **Minimización:** Eliminar funciones de depuración (*debug*) o componentes de prueba antes de pasar a producción.
* **Auditoría de APIs:** Inspeccionar rigurosamente las integraciones con otros sistemas para cerrar aquellas que sean innecesarias.

(estos son solo los ejemplos más representativos para entender el concepto general dentro de cada área. En la vida real y en entornos corporativos, el hardening no es una lista de 10 o 15 pasos, sino un proceso exhaustivo y continuo)

---

## 3. El Paradigma de la Capa 7 (Capa de Aplicación)

La Capa 7 del Modelo OSI es la que interactúa directamente con el usuario mediante protocolos como HTTP/HTTPS, FTP o SMTP. 

**La Ceguera del Firewall:** Los firewalls de red tradicionales operan en las Capas 3 y 4 (IP y Puertos). No pueden detener ataques a aplicaciones web porque, para el cortafuegos, una inyección de código que viaja a través del puerto 443 luce exactamente igual que "tráfico web normal y legítimo".
**La Solución Estructural:** Se requiere la implementación de un **WAF** (*Web Application Firewall*) capaz de inspeccionar las cargas útiles (payloads) de HTTP, combinado con prácticas de desarrollo de software seguro.

### El Estándar OWASP
El *Open Web Application Security Project* (OWASP) es la fundación global que dicta el estándar de facto en seguridad de software. Su **OWASP Top 10** enumera las vulnerabilidades más críticas y explotadas a nivel mundial.

---

## 4. Vectores de Ataque Críticos (OWASP Top 10)

### 1. Inyección SQL (SQLi)
* **Mecánica:** El programador confía ciegamente en lo que el usuario escribe en un formulario de login. El atacante inserta sentencias de base de datos directamente en el campo de texto. El payload clásico `OR 1=1` engaña a la lógica del backend, haciendo que la validación resulte siempre en "verdadero" y permitiendo el acceso sin conocer la contraseña real.
* **Mitigación:** Jamás concatenar consultas SQL de forma directa. Se deben utilizar exclusivamente consultas parametrizadas (*Prepared Statements*) y realizar una sanitización estricta de las entradas.

### 2. Cross-Site Scripting (XSS)
* **Mecánica:** El atacante inyecta scripts maliciosos (generalmente JavaScript) en una página web confiable y dinámica. Cuando un usuario legítimo ingresa a esa página, su propio navegador ejecuta el código malicioso. Esto permite el "secuestro de sesión" (*Session Hijacking*) robando las cookies de autenticación del usuario.
* **Mitigación:** Escapar y codificar todas las variables de salida antes de renderizarlas en pantalla, convirtiendo los caracteres especiales (como `<` o `>`) a formatos HTML inofensivos.

### 3. Broken Access Control (IDOR)
* **Mecánica:** *Insecure Direct Object Reference*. El atacante accede a datos que no le corresponden saltándose la autorización. Consiste en iniciar sesión como usuario normal, notar que la URL dice `perfil?id=10`, y alterarla manualmente a `id=1` para forzar al servidor a mostrar el perfil o los documentos confidenciales del administrador.
* **Mitigación:** Validar siempre el token y los permisos de sesión del lado del servidor en cada petición HTTP, sin confiar jamás en el identificador (ID) que envía el navegador del cliente.

---

## 5. Ecosistema Práctico (Herramientas del Oficio)

Para replicar y auditar estas fallas de Capa 7, el profesional de seguridad debe pasar del simple escaneo de red a la manipulación directa del tráfico web:

1. **DevTools del Navegador (Nivel Básico):** Uso de la consola de desarrollador (tecla F12) para inyectar JavaScript en vivo, probar vectores XSS e inspeccionar el tráfico y almacenamiento local del navegador.
2. **VS Code / Sublime Text (Nivel Código Fuente):** Entornos de desarrollo ligeros utilizados para revisar repositorios de código (ej. PHP, Python o Node.js) en busca de vulnerabilidades de "caja blanca", como consultas a base de datos no parametrizadas.
3. **Burp Suite (El Estándar Profesional):** Funciona como un Proxy de Intercepción. Detiene la petición HTTP(S) justo en el momento en que sale del navegador web, manteniéndola en pausa. Esto le permite al analista (*Pentester*) modificar variables (inyectar SQL, manipular IDs o alterar Cookies) "en el aire", para luego soltar la petición hacia el servidor y analizar cómo responde el backend ante el ataque.
