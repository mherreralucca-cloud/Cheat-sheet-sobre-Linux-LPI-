# 📂 Teoría de Hardening y Seguridad en Capa 7 (OWASP)

**Nota:** *Aunque este módulo se aleja temporalmente de los comandos específicos de la terminal de Linux, los conceptos aquí documentados impactan de forma colateral y directa en el sistema operativo. El Hardening y la seguridad en Capa de Aplicación son principios universales y obligatorios en cualquier arquitectura de software y hardware (Defensa en Profundidad). Un servidor Linux perfectamente asegurado en su base es inútil si la aplicación web que hospeda es vulnerable.*

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

---

## 6. Arquitectura IAM y el Modelo AAA (El Nuevo Perímetro)
Con la migración masiva a la nube, el auge del trabajo remoto y la proliferación de usuarios e identidades "no humanas" (tokens de APIs, dispositivos IoT, agentes de IA), el perímetro de red clásico delimitado por un firewall ya no es suficiente. **La identidad digital es el nuevo perímetro de seguridad.**

El ciclo de vida de una identidad comienza con la **Administración y Aprovisionamiento**, que consiste en asignar atributos únicos a un usuario en una base de datos central o Directorio corporativo. Cuando un empleado se desvincula de la organización, el desaprovisionamiento debe ser estrictamente automático para evitar **cuentas huérfanas**, las cuales representan un vector crítico de ataque de persistencia.

### El Modelo AAA: La Columna Vertebral Estricta
Todo sistema de acceso robusto se divide matemáticamente en estas tres fases secuenciales obligatorias:

#### A. Autenticación (Authentication) - Validación de Identidad
Consiste en verificar que eres quien dices ser presentando credenciales válidas. Las contraseñas tradicionales en texto plano murieron; el hardening moderno exige esquemas avanzados:
* **MFA (Multifactor Authentication):** Exige múltiples pruebas combinando factores (algo que sabes, algo que tienes y algo que eres).
* **SSO (Single Sign-On):** Permite iniciar sesión una sola vez mediante un portal centralizado que genera un token criptográfico firmado. Este token se comparte con las demás aplicaciones, evitando que el usuario deba gestionar decenas de claves distintas.
* **Autenticación Adaptativa (Basada en Riesgo):** Utiliza Inteligencia Artificial y Machine Learning. Si tu comportamiento de acceso cambia abruptamente (ej. intentas iniciar sesión desde una IP desconocida en Rusia a las 3 AM), el sistema aumenta dinámicamente las exigencias de validación o bloquea el login de inmediato.

#### B. Autorización (Authorization) - Control de Privilegios
Determina qué acciones y recursos tienes permitido realizar o ver dentro del sistema una vez autenticado con éxito, aplicando estrictamente políticas granulares (como RBAC o control de acceso basado en roles).

#### C. Auditoría / Registro (Accounting) - Trazabilidad total
Es la persistencia y registro en logs inmutables de cada acción ejecutada por la identidad (ingresos, comandos, edición de archivos), fundamental tanto para el análisis forense como para el cumplimiento de normativas de cumplimiento legal.

---

## 7. Seguridad Ofensiva en Correo Electrónico (Capa 7 Aplicada)
El correo electrónico es el vector número uno para ataques de Phishing e ingeniería social. Para mitigar la suplantación de identidad de un dominio en Capa 7, se exige la configuración defensiva de registros DNS específicos:

* **SPF (Sender Policy Framework):** Una lista blanca cargada en el DNS que define cuáles IPs de servidores están autorizadas para enviar correos en nombre del dominio.
* **DKIM (DomainKeys Identified Mail):** Añade una firma digital criptográfica en la cabecera de los correos salientes, garantizando la integridad del mensaje y demostrando que no fue alterado en tránsito.
* **DMARC (Domain-based Message Authentication, Reporting, and Conformance):** La política de acción unificada. Le indica al servidor receptor qué hacer (ej. rechazar o mandar a Spam) si un correo entrante falla las pruebas de SPF o DKIM.
* **BIMI (Brand Indicators for Message Identification):** Un protocolo visual avanzado que añade el logo verificado de la marca al lado del correo dentro del buzón (ej. en Gmail), aumentando la confianza del usuario final ante accesos legítimos.

---

## 8. El Ecosistema Defensivo Avanzado (Herramientas Especializadas del SOC)
Un SOC (*Security Operations Center*) moderno utiliza herramientas accesorias de infraestructura para sostener el control de accesos y el robustecimiento general:

* **PAM (Privileged Access Management):** Bóvedas de alta seguridad diseñadas exclusivamente para proteger cuentas administrativas de superusuario o "root". Aíslan la credencial real del operador y la rotan de forma automatizada y aleatoria después de cada uso.
* **Gestión de Secretos (Secret Management):** Bóvedas seguras orientadas a que las aplicaciones y servidores (no humanos) guarden de forma encriptada sus tokens, certificados y claves de API, eliminando las malas prácticas de dejarlas expuestas en código fuente plano.
* **ITDR (Identity Threat Detection and Response):** Sistemas de defensa activa que monitorean constantemente el comportamiento de las identidades para detectar ciberataques específicos (ej. escalada de privilegios o robo de tickets de sesión), bloqueando la cuenta automáticamente ante anomalías.
* **CIAM e IDaaS:** *CIAM* se especializa en gestionar con seguridad identidades de clientes externos (ej. en un e-commerce), mientras que *IDaaS* (Identity as a Service) provee toda la infraestructura IAM avanzada empaquetada como un servicio administrado directo en la nube.

* ---

## Conclusión: El Verdadero Significado de la Defensa en Profundidad

El robustecimiento (*Hardening*) no es una meta estática, sino una disciplina operativa continua. Como se ha demostrado a lo largo de este módulo, la seguridad perimetral tradicional ha evolucionado hacia un modelo donde **la identidad es el nuevo perímetro** y **la capa de aplicación es la frontera más expuesta**. 

Un verdadero ingeniero de infraestructura u operador de ciberseguridad no busca un sistema 100% inexpugnable, sino estructurar tantas capas de control interconectadas (desde los permisos base del kernel de Linux y restricciones SUID, hasta las validaciones criptográficas en DNS y auditorías de Capa 7 con proxies de intercepción) que hagan que el costo y esfuerzo operativo de un ataque supere con creces el valor de la recompensa para el intruso.

---

## 🚀 Feedback y Contribuciones

Este repositorio funciona como una base de conocimiento abierta y en constante expansión. 

* 🤝 **¿Querés contribuir?** Si encontrás un error conceptual, querés expandir un vector del OWASP Top 10 o añadir herramientas al ecosistema defensivo, los *Pull Requests* son más que bienvenidos.
* 🐛 **Reportar anomalías:** Si tenés sugerencias o mejoras, sentite libre de abrir un *Issue*.
* ⭐ **Apoyo al proyecto:** Si estos resúmenes y hojas de trucos te resultaron útiles para repasar, ¡dejame una estrella en el repositorio para apoyar el contenido Open Source!
