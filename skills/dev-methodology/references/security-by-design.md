# Seguridad desde el diseño

Actúa con mentalidad de defensa en profundidad. **Nunca prometas que una aplicación es
invulnerable.**

Para controles de seguridad concretos por tema (XSS, CSRF, SSRF, inyección, headers,
auth, secretos, Next.js/Node, dependencias) usa la skill `owasp-security` si está
disponible — tiene los cheat sheets de OWASP desglosados por tema con valores exactos
de headers/parámetros. Esta referencia es el análisis de alto nivel que precede a eso.

## Análisis inicial (antes de tocar código)

Identifica:
- Superficie de ataque.
- Activos y datos sensibles.
- Límites de confianza (dónde cruza datos entre zonas de confianza distintas).
- Roles y permisos.
- Entradas controladas por usuarios.
- Dependencias y servicios externos.
- Riesgos de exposición de secretos.
- Amenazas específicas del stack en uso.

## Controles a evaluar (implementa los que correspondan, explica cuáles no aplican y por qué)

- Validación de entradas mediante esquemas y listas permitidas (allow-list, no
  deny-list).
- Autenticación y autorización verificadas en el servidor, nunca solo en el cliente.
- Principio de mínimo privilegio.
- Comprobación de permisos en cada operación sensible (no solo al entrar a la
  pantalla).
- Protección contra SQL injection mediante consultas parametrizadas u ORM usado
  correctamente.
- Prevención de XSS, CSRF, SSRF, path traversal, command injection y deserialización
  insegura, según la superficie de ataque real del proyecto.
- CORS restrictivo cuando aplique.
- Gestión segura de sesiones, tokens y cookies.
- Limitación de solicitudes (rate limiting) en endpoints expuestos.
- Contraseñas almacenadas con algoritmos de hashing apropiados (ej. Argon2id).
- Gestión de secretos mediante variables de entorno o un gestor de secretos — nunca
  en el repositorio.
- Cifrado en tránsito (TLS).
- Manejo de errores sin filtrar secretos, stack traces ni datos internos al usuario.
- Logs de auditoría sin credenciales, tokens ni datos sensibles innecesarios.
- Políticas de dependencias: actualización y revisión de vulnerabilidades conocidas.
- Protección de archivos, cargas de contenido (uploads) y procesos administrativos.
- Configuración segura de producción, cabeceras de seguridad y permisos.
- Backups, restauración y migraciones seguras cuando corresponda.

Consulta la documentación oficial del stack y referencias como OWASP Top 10 y OWASP
ASVS cuando estén disponibles. **No apliques controles irrelevantes solo por
completar la lista** — di explícitamente cuáles son aplicables a este proyecto y
cuáles no, y por qué.

## Reglas estrictas (no negociables)

- Nunca introduzcas credenciales reales en el repositorio.
- Nunca imprimas secretos en logs.
- Nunca desactives TLS, autenticación, autorización o validaciones como solución
  permanente (un bypass temporal documentado y con fecha de reversión es distinto a
  desactivar algo "para que pase la prueba").
- No ejecutes pruebas destructivas contra sistemas reales sin autorización explícita.
- No hagas pruebas de penetración fuera del entorno autorizado.
- No afirmes que el proyecto "está protegido" sin indicar qué comprobaciones
  concretas se hicieron.
- Prioriza vulnerabilidades según probabilidad, impacto y exposición real — no todas
  pesan igual.
