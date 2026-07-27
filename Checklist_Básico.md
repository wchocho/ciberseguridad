# Lista de Verificación (Checklist) de Ciberseguridad para Proyectos de Software



## 📑 1. Identificación de Activos y Arquitectura Básica

- [ ] **[CRÍTICO] Identificación de Activos:** ¿Tienen identificados los datos y recursos más valiosos del sistema (ej.: contraseñas, datos personales de usuarios, BD, archivos del servidor)?
- [ ] **[CRÍTICO] Principio de Menor Privilegio:** ¿Los usuarios del sistema (y el usuario de conexión a la base de datos) poseen únicamente los permisos estrictamente necesarios para su función?
- [ ] **Defensa en Profundidad:** ¿Existen múltiples capas de seguridad (ej.: validación en el cliente + validación en el servidor + restricciones en la BD)?
- [ ] **Seguridad por Defecto:** ¿El sistema viene configurado por defecto con la opción más segura (ej.: cuentas inactivas hasta verificación, permisos denegados salvo autorización explícita)?

---

## 🔑 2. Autenticación, Autorización y Gestión de Sesiones

- [ ] **[CRÍTICO] Hashing de Contraseñas:** ¿Las contraseñas de los usuarios se almacenan en la base de datos utilizando algoritmos de *hashing* seguros con *salt* automático (ej.: `password_hash()` en PHP con `PASSWORD_DEFAULT` o `BCRYPT`)?
- [ ] **[CRÍTICO] Prohibición de Texto Plano / Hash Obsoletos:** ¿Se evita totalmente guardar contraseñas en texto plano o usar funciones criptográficamente rotas como `MD5()` o `SHA1()`?
- [ ] **[CRÍTICO] Control de Acceso (RBAC):** ¿Se verifica en el **servidor** que el usuario autenticado tiene el rol necesario antes de cargar una página privada o ejecutar una acción sensible (ej. panel `/admin`)?
- [ ] **Manejo Seguro de Sesiones:** ¿Se regenera el ID de sesión tras el inicio de sesión exitoso (ej.: `session_regenerate_id(true)`) para prevenir *Session Fixation*?
- [ ] **Atributos de Cookies:** Si se utilizan cookies de sesión, ¿están configuradas con las banderas `HttpOnly` (previene lectura por JavaScript/XSS) y `SameSite=Strict` o `Lax` (previene CSRF)?
- [ ] **Cierre de Sesión Efectivo:** ¿La función de *Logout* destruye la sesión tanto en el servidor (`session_destroy()`) como en la cookie del navegador?

---

## 🛡️ 3. Desarrollo Seguro y Validación de Entradas (OWASP Top 10)

### A. Inyección SQL (SQLi)
- [ ] **[CRÍTICO] Sentencias Preparadas:** ¿Se utilizan **exclusivamente** consultas preparadas (*Prepared Statements*) con parámetros vinculados (PDO o MySQLi) para **TODAS** las interacciones con la base de datos?
- [ ] **Prohibición de Concatenación:** ¿Se evita de forma estricta concatenar o interpolar variables directamente en las sentencias SQL (ej.: `SELECT * FROM users WHERE user = '$user'`)?

### B. Cross-Site Scripting (XSS)
- [ ] **[CRÍTICO] Sanitización y Escape de Salida:** ¿Se escapan adecuadamente todos los datos provistos por el usuario antes de renderizarlos en el HTML (ej.: utilizando `htmlspecialchars($data, ENT_QUOTES, 'UTF-8')`)?
- [ ] **Validación de Entradas:** ¿Se filtran y validan los tipos de datos recibidos por `$_GET`, `$_POST` o APIs (ej. usando `filter_var()` para emails, números enteros, etc.)?

### C. Cross-Site Request Forgery (CSRF)
- [ ] **Tokens Anti-CSRF:** En formularios HTML que realicen cambios de estado (POST/PUT/DELETE), ¿se incluye un token secreto único por sesión que sea verificado en el servidor antes de procesar la solicitud?

### D. Control de Acceso Granular e IDOR (*Insecure Direct Object References*)
- [ ] **[CRÍTICO] Prevención de IDOR / BOLA:** ¿Se verifica formalmente en el servidor que el usuario autenticado es el propietario o tiene permiso explícito para acceder/modificar el recurso específico solicitado mediante su identificador (ej.: `GET /factura.php?id=105`)?

### E. Protección contra Fuerza Bruta y Autenticación (*Broken Authentication*)
- [ ] **Limitación de Intentos (*Rate Limiting*):** ¿Existen controles para limitar la frecuencia de intentos fallidos de inicio de sesión o peticiones automatizadas a APIs, evitando el minado de contraseñas?

### F. Subida y Manejo Inseguro de Archivos (*File Upload / Inclusion*)
- [ ] **Validación de Archivos Subidos:** Si la aplicación permite subir archivos, ¿se valida la extensión, el tipo MIME y el tamaño en el servidor (no solo en JavaScript)?
- [ ] **Almacenamiento Seguro:** ¿Los archivos subidos por usuarios se guardan fuera del directorio raíz web o se renombran de forma aleatoria sin permisos de ejecución (`.php`, `.sh`)?
- [ ] **Inclusión de Archivos Securizada:** ¿Se evita pasar parámetros dinámicos no controlados a funciones de inclusión (ej. `include($_GET['page'])`) para prevenir *Local/Remote File Inclusion (LFI/RFI)*?

### G. Redirecciones Inseguras (*Open Redirects*)
- [ ] **Prevención de Redirecciones Abiertas:** ¿Se validan o restringen a una lista blanca interna las URLs destino utilizadas en cabeceras de redirección (evitando escenarios como `header("Location: " . $_GET['redirect'])`)?

### H. Falsificación de Peticiones del Lado del Servidor (*SSRF*)
- [ ] **Control de Solicitudes Salientes (SSRF):** Si la aplicación descarga o procesa recursos desde URLs ingresadas por el usuario, ¿se bloquea la posibilidad de realizar peticiones a direcciones IP internas o de la red local (`127.0.0.1`, `10.x.x.x`, `192.168.x.x`)?

### I. Cabeceras de Seguridad HTTP (*Security Headers*)
- [ ] **Cabeceras defensivas HTTP:** ¿El servidor o la aplicación envía cabeceras defensivas como `X-Frame-Options` (prevención de *Clickjacking*), `X-Content-Type-Options: nosniff` y `Content-Security-Policy (CSP)`?

---

## 💻 4. Programación Segura y Buenas Prácticas de Código

- [ ] **[CRÍTICO] Prohibición de Ejecución Dinámica Peligrosa:** ¿Se evita totalmente el uso de funciones de ejecución de código o comandos del sistema operativo sin control (ej.: `eval()`, `exec()`, `shell_exec()`, `passthru()`) y la desmaterialización insegura de objetos (`unserialize()`)?
- [ ] **[CRÍTICO] Manejo Seguro de Excepciones y Errores:** ¿Se capturan adecuadamente las excepciones mediante bloques `try-catch`, evitando que la aplicación falle de forma no controlada o exponga *stack traces* e información de infraestructura al usuario final?
- [ ] **Tipado y Comparaciones Estrictas:** ¿Se utilizan comparaciones estrictas (ej.: `===` y `!==` en PHP/JavaScript) y comprobaciones de tipo explícitas para prevenir vulnerabilidades de manipulación de tipos (*Type Juggling*)?
- [ ] **Gestión de Dependencias y Terceros:** ¿Se han verificado y actualizado los paquetes o librerías de terceros (ej. ejecutando `npm audit` o `composer audit`) para garantizar que no contengan vulnerabilidades conocidas (*Vulnerable and Outdated Components*)?
- [ ] **Limpieza de Código y Depuración:** ¿Se eliminaron todos los archivos de prueba o diagnóstico (ej.: `phpinfo.php`, `test.php`), var_dumps/console.logs de depuración y comentarios que expongan lógica sensible o credenciales de desarrollo?

---

## 🗄️ 5. Seguridad en Base de Datos (MySQL / MariaDB)

- [ ] **[CRÍTICO] Usuario de Aplicación:** ¿La aplicación se conecta a la base de datos con un usuario dedicado que solo tiene permisos de `SELECT`, `INSERT`, `UPDATE`, `DELETE` en la BD del proyecto, y **NUNCA** con el usuario `root`?
- [ ] **Sin Credenciales Expuestas:** ¿Las credenciales de acceso a la BD están almacenadas en un archivo de configuración restringido o en variables de entorno (`.env`), y excluidas del control de versiones (`.gitignore`)?
- [ ] **Hardening de MySQL:** ¿La BD está configurada para escuchar únicamente en `localhost` (`127.0.0.1`) a menos que requiera acceso remoto justificado?

---

## ⚙️ 6. Configuración del Servidor y Sistemas Operativos (*Hardening*)

- [ ] **[CRÍTICO] Desactivación de Errores Verbosos:** En entorno de producción, ¿están ocultos los mensajes detallados de error de base de datos o intérprete (`display_errors = Off` en `php.ini`) para evitar la fuga de información sensible (*Information Disclosure*)?
- [ ] **Páginas de Error Personalizadas:** ¿El sistema muestra páginas genéricas personalizadas para errores 404 (No encontrado) y 500 (Error interno)?
- [ ] **Ocultación de Banners/Versiones:** ¿Se han deshabilitado las cabeceras que revelan tecnologías o versiones del servidor (ej.: `expose_php = Off` en PHP, `ServerTokens ProductOnly` en Apache/Nginx)?
- [ ] **Permisos de Archivos en el SO:** ¿Los archivos del proyecto en el servidor tienen permisos restrictivos (ej.: carpetas `755`, archivos `644`, archivos de configuración con credenciales `600`)?

---

## 🔒 7. Gestor de Secretos y Control de Versiones

- [ ] **[CRÍTICO] Git sin Secretos Hardcodear:** ¿Se ha verificado mediante inspección que en el repositorio de código (Git) **NO** existen claves API, tokens, contraseñas de BD ni claves privadas escritas en duro en el código fuente?
- [ ] **Uso de `.gitignore`:** ¿Está configurado un archivo `.gitignore` adecuado que excluya archivos `.env`, carpetas de dependencias (`node_modules`, `vendor`), logs y archivos temporales?

---

## 🌐 8. Criptografía y Comunicaciones

- [ ] **Uso de HTTPS / TLS:** En caso de despliegue en red o servidor local de prueba, ¿se fuerzan las conexiones cifradas mediante HTTPS en lugar de HTTP claro?
- [ ] **Algoritmos Criptográficos Seguros:** Si el proyecto realiza cifrado manual de datos, ¿utiliza algoritmos estándar y probados (ej.: AES-256-GCM) en lugar de crear algoritmos de cifrado propios?

---

## 📋 9. Monitoreo, Logs y Documentación del Proyecto

- [ ] **Registro de Auditoría (Logging):** ¿El sistema registra eventos de seguridad relevantes (intentos fallidos de login, cambios de contraseña, errores críticos) sin almacenar datos sensibles en los logs (como contraseñas en texto plano)?



---
*Documento generado para la materia Ciberseguridad. DGETP - UTU.*
