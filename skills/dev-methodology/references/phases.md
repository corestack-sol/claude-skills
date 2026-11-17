# Desarrollo por fases verificables

Organiza el proyecto en fases verificables. Adapta las fases al alcance real — no
fuerces todas si la tarea no las necesita. Puedes desarrollar fases en paralelo solo
cuando las dependencias lo permitan y no se comprometa la coherencia del proyecto.

## Fase 0 — Descubrimiento y auditoría
- Identificar stack, versiones y estructura.
- Revisar el estado actual del repositorio.
- Identificar módulos existentes, deuda técnica y riesgos.
- Determinar qué funciona y qué no se ha comprobado todavía.
- Establecer el alcance del proyecto.

## Fase 1 — Arquitectura y planificación
- Definir requisitos funcionales y no funcionales.
- Establecer módulos y responsabilidades.
- Definir contratos entre módulos.
- Crear el árbol inicial de archivos.
- Identificar decisiones arquitectónicas importantes.
- Definir criterios de aceptación.

## Fase 2 — Base técnica
- Configurar estructura, scripts, validaciones y herramientas.
- Establecer configuración por entorno.
- Preparar manejo de errores, logging y seguridad básica.
- Preparar pruebas y pipeline de verificación cuando corresponda.

## Fase 3 — Implementación incremental
- Construir módulos pequeños.
- Completar una funcionalidad de extremo a extremo cuando sea viable.
- Validar cada módulo antes de depender de él.
- Actualizar el mapa y los estados del proyecto (`PROJECT.md`).

## Fase 4 — Integración
- Verificar contratos y comunicación entre módulos.
- Probar flujos completos.
- Validar permisos, errores y escenarios límite.
- Comprobar consistencia de datos.

## Fase 5 — QA y seguridad
- Ejecutar pruebas funcionales, de regresión y de seguridad.
- Corregir defectos.
- Documentar limitaciones conocidas.
- No declarar aprobada una fase que tenga criterios críticos incumplidos.

## Fase 6 — Entrega y operación
- Preparar build o despliegue.
- Documentar instalación y configuración.
- Preparar instrucciones de uso.
- Documentar respaldo, recuperación y mantenimiento cuando aplique.
- Registrar las tareas que siguen pendientes.
