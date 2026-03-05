**Plan de Remediación — todoApp**



La aplicación todoApp presenta múltiples vulnerabilidades críticas identificadas en la Tarea 1, principalmente relacionadas con la ausencia de autenticación, control de acceso, validación de entradas y configuraciones inseguras.



El estado actual de la aplicación refleja un enfoque orientado únicamente a funcionalidad, sin aplicar principios de diseño seguro. Esto genera exposición total de datos, facilidad de explotación y ausencia de trazabilidad.



El objetivo de este plan es transformar los hallazgos en acciones técnicas concretas alineadas con:



* OWASP Top 10 (2025)



* Los 7 Principios de Diseño Seguro



* Enfoque Security by Design





La prioridad estratégica es:



* Implementar autenticación y autorización.



* Proteger la base de datos.



* Aplicar validaciones estrictas.



* Incorporar capas de defensa adicionales.



| #  | Vulnerabilidad                | Severidad | Principio                       | Solución                      | Clase   |

| -- | ----------------------------- | --------- | ------------------------------- | ----------------------------- | ------- |

| 1  | Sin autenticación             | Crítica   | Zero Trust                      | Middleware JWT obligatorio    | Clase 3 |

| 2  | IDOR                          | Crítica   | Menor Privilegio                | Validar ownership por usuario | Clase 3 |

| 3  | XSS almacenado                | Alta      | Fail Secure                     | Validación + sanitización     | Clase 4 |

| 4  | err.message expuesto          | Media     | Fail Secure                     | Handler global de errores     | Clase 4 |

| 5  | Sin rate limiting             | Media     | Defensa en Profundidad          | express-rate-limit            | Clase 5 |

| 6  | MongoDB sin autenticación     | Crítica   | Seguro por Defecto              | Habilitar auth y credenciales | Clase 5 |

| 7  | Mass assignment               | Alta      | Menor Privilegio                | Whitelisting de campos        | Clase 4 |

| 8  | Sin CORS configurado          | Media     | Defensa en Profundidad          | Configuración restrictiva     | Clase 5 |

| 9  | Sin headers de seguridad      | Media     | Defensa en Profundidad          | Helmet                        | Clase 5 |

| 10 | Sin audit logs                | Media     | Zero Trust                      | Logging con Winston           | Clase 6 |

| 11 | Sin HTTPS                     | Alta      | Defensa en Profundidad          | Forzar TLS en producción      | Clase 6 |

| 12 | Connection string hardcodeada | Alta      | Separación de Responsabilidades | Variables de entorno (.env)   | Clase 2 |







**Vulnerabilidad #1: Sin autenticación en ningún endpoint**



* Severidad: Crítica



* OWASP: A07 — Identification and Authentication Failures



* Principio violado: Zero Trust



* Descripción: La API permite acceso total sin autenticación.



* Solución concreta:



 	Implementar registro y login con JWT.



 	Crear authMiddleware.js:

*const jwt = require("jsonwebtoken");*



*module.exports = function(req, res, next){*

*const token = req.headers.authorization?.split(" ")\[1];*

*if(!token) return res.status(401).json({message: "Token requerido"});*



*try {*

&nbsp;   \*const decoded = jwt.verify(token, process.env.JWT\\\_SECRET);\*

    \*req.user = decoded;\*

    \*next();\*


*} catch {*

&nbsp;   \*return res.status(401).json({message: "Token inválido"});\*


*}*

*};*





Proteger todas las rutas /api/tareas.



**Vulnerabilidad #3: XSS almacenado**

* Severidad: Alta



* OWASP: A03 — Injection



* Principio violado: Fail Secure



* Solución concreta:

 	Implementar schema Joi:

*const schema = Joi.object({*

*title: Joi.string().min(3).max(200).pattern(/^\[^<>]\*$/).required(),*

*completed: Joi.boolean()*

*}).unknown(false);*



* Rechazar campos no definidos.



* Retornar 422 si no cumple validación.





**Vulnerabilidad #4: err.message expuesto**

* Severidad: Media



* OWASP: A04 — Insecure Design



* Principio violado: Fail Secure



* Solución concreta:

 	Middleware global:

*app.use((err, req, res, next) => {*

*console.error(err);*

*res.status(500).json({ message: "Error interno del servidor" });*

*});*





**Vulnerabilidad #5: Sin rate limiting**

* Severidad: Media



* OWASP: A04



* Principio violado: Defensa en Profundidad



* Solución concreta:

*const rateLimit = require("express-rate-limit");*



*const limiter = rateLimit({*

*windowMs: 15 \* 60 \* 1000,*

*max: 100*

*});*



*app.use(limiter);*





**Vulnerabilidad #6: MongoDB sin autenticación**

* Severidad: Crítica



* OWASP: A05 — Security Misconfiguration



* Principio violado: Seguro por Defecto



* Solución concreta:



* Crear usuario administrador en Mongo.



* Configurar:

 	mongodb://user:password@mongo:27017/todoApp?authSource=admin



* No exponer puerto públicamente.





**Vulnerabilidad #7: Mass Assignment**

* Severidad: Alta



* OWASP: A04



* Principio violado: Menor Privilegio



* Solución concreta:

*const { title, completed } = req.body;*

*const task = new Task({ title, completed, owner: req.user.id });*





**Vulnerabilidad #8: Sin CORS configurado**

* Severidad: Media



* OWASP: A05



* Principio violado: Defensa en Profundidad



* Solución concreta:

*const cors = require("cors");*



*app.use(cors({*

*origin: "https://mi-frontend.com",*

*methods: \["GET","POST","PUT","DELETE"]*

*}));*





**Vulnerabilidad #9: Sin headers de seguridad**

* Severidad: Media



* OWASP: A05



* Principio violado: Defensa en Profundidad



* Solución concreta:

*const helmet = require("helmet");*

*app.use(helmet());*





**Vulnerabilidad #10: Sin audit logs**

* Severidad: Media



* OWASP: A09



* Principio violado: Zero Trust



* Solución concreta:



* Implementar Winston.



 	- Loggear:



 	- Logins fallidos



 	- DELETE



 	- Errores 4xx/5xx



**Vulnerabilidad #11: Sin HTTPS**

* Severidad: Alta



* OWASP: A02 — Cryptographic Failures



* Principio violado: Defensa en Profundidad



* Solución concreta:



 	- Configurar NGINX con TLS.



 	- Forzar redirección HTTP → HTTPS.



**Vulnerabilidad #12: Connection string hardcodeada**

* Severidad: Alta



* OWASP: A05



* Principio violado: Separación de Responsabilidades



* Solución concreta:



* Crear archivo .env

 	MONGO\_URI=...

 	JWT\_SECRET=...





**Sección Impacto**

Vulnerabilidad más crítica: Sin autenticación + IDOR

Escenario de ataque



Un atacante puede:



* Acceder sin login.



* Listar todas las tareas.



* Eliminar todas con un script automatizado.



* No dejar rastro (sin logs).



**Impacto técnico**



* Pérdida total de integridad.



* Compromiso de disponibilidad.



* Violación del principio de mínimo privilegio.



* Exposición completa del sistema.





**Impacto organizacional**



* Pérdida de confianza.



* Riesgo legal.



* Interrupción del servicio.





**Prioridad**



Debe remediarse antes de cualquier despliegue en producción.



