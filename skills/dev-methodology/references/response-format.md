# Formato de cierre de tarea, y documentación para el usuario final

## Resumen obligatorio al terminar cada tarea

No afirmes que ejecutaste comandos, inspeccionaste archivos o validaste resultados si
no lo hiciste de verdad. El resumen cubre:

### Resultado
Qué se completó y qué comportamiento cambió.

### Archivos modificados
Lista de rutas exactas y propósito de cada cambio.

### Verificaciones
Comandos ejecutados y resultados reales — separa lo aprobado, lo fallido y lo que no
se pudo ejecutar (y por qué).

### Estado del módulo
Estado anterior, estado actual, y criterio pendiente si existe (ver
`project-md.md` para los nombres de estado).

### Documentación
Qué secciones de `PROJECT.md` se actualizaron.

### Riesgos y pendientes
Problemas conocidos, decisiones que requieren aprobación, próximos pasos
necesarios.

En un contexto conversacional informal (no un reporte formal pedido explícitamente),
esta estructura puede condensarse, pero el CONTENIDO —qué cambió, qué se verificó de
verdad, qué queda pendiente— sigue siendo obligatorio.

## Documentación para el usuario final (sección 14 de PROJECT.md)

Explica el producto, no la implementación interna. Incluye cuando corresponda:

- Qué permite hacer cada funcionalidad.
- Requisitos previos.
- Instrucciones paso a paso.
- Campos que debe completar el usuario.
- Resultado esperado.
- Mensajes de error y cómo resolverlos.
- Permisos necesarios.
- Limitaciones y situaciones especiales.

Lenguaje claro, consistente y no técnico cuando sea posible. **No expongas al usuario
final** credenciales, secretos, detalles de infraestructura, instrucciones internas ni
información de seguridad que facilite ataques. Si una funcionalidad cambia, actualiza
la guía correspondiente en el mismo momento.
