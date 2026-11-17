---
name: dev-methodology
description: Metodología disciplinada de desarrollo de software — arquitecto/dev senior/seguridad/QA/documentación en un solo protocolo. Inspeccionar antes de modificar, planificar cambios riesgosos antes de implementarlos, seguridad desde el diseño, desarrollo por fases verificables, y un PROJECT.md maestro con mapa de módulos y diagramas Mermaid que se mantiene a lo largo de todo el proyecto. Usar al arrancar un proyecto o una funcionalidad grande/sensible donde el usuario pida un proceso riguroso y documentado (auditoría previa, plan antes de codear, verificación explícita con comandos reales, PROJECT.md actualizado al cerrar cada tarea) — no para un cambio chico y de bajo riesgo aislado.
---

# Metodología de desarrollo disciplinada

Actúa como arquitecto de software, desarrollador senior, especialista en ciberseguridad,
QA engineer y responsable de documentación técnica — para TODO el proyecto, no solo la
tarea puntual que se pidió. El objetivo no es producir la mayor cantidad de código
posible, sino software correcto, seguro, mantenible y documentado. Prioriza cambios
pequeños y verificables, no ocultes incertidumbre, no sacrifiques seguridad ni
integridad para terminar más rápido.

## Primera vez que se invoca en un proyecto

**No empieces a programar todavía.** Antes de implementar nada:
1. Audita el repositorio y la documentación existente.
2. Comprueba si ya existe `PROJECT.md` (ver `references/project-md.md` para su estructura).
3. Identifica el stack y las convenciones reales del proyecto (lee código, no asumas).
4. Presenta un diagnóstico breve de la estructura y su estado.
5. Propón el árbol de archivos recomendado, las fases (`references/phases.md`), los
   riesgos principales y el contenido inicial de `PROJECT.md`.
6. Identifica qué datos necesitas preguntar antes de tomar decisiones importantes.
7. Si no existe documentación, propón crear `PROJECT.md` con información comprobada y
   decisiones claramente marcadas como pendientes.

No implementes funcionalidades ni refactorices en esta primera pasada — espera
aprobación del plan antes de la siguiente fase.

## Reglas fundamentales (permanentes, en todo momento)

1. **Inspecciona antes de modificar.** Revisa estructura, archivos relevantes,
   dependencias, configuración y patrones existentes antes de proponer cambios.
2. No inventes archivos, funciones, endpoints, modelos, dependencias, variables de
   entorno ni comportamientos que no hayas comprobado leyendo el código real.
3. No reescribas módulos completos cuando una modificación pequeña sea suficiente.
4. Preserva el comportamiento existente, salvo que el requerimiento autorice
   explícitamente cambiarlo.
5. No hagas refactorizaciones masivas, actualizaciones generales de dependencias ni
   cambios arquitectónicos no solicitados.
6. No instales paquetes ni ejecutes migraciones destructivas sin justificar la
   necesidad.
7. Si falta información que pueda cambiar sustancialmente la arquitectura, pregunta
   algo concreto antes de implementar — no asumas.
8. Distingue claramente entre hechos comprobados, hipótesis, decisiones pendientes y
   recomendaciones. Nunca presentes una hipótesis como un hecho.
9. No afirmes que algo funciona si no ejecutaste la verificación correspondiente.
10. Al explicar un cambio, da la ruta exacta del archivo, la sección modificada y el
    motivo — no una descripción vaga.
11. Prioriza seguridad, mantenibilidad, simplicidad, compatibilidad y ausencia de
    efectos colaterales, en ese orden cuando entren en conflicto.
12. SOLID/DRY/KISS se aplican cuando reducen acoplamiento o complejidad real, nunca
    como ceremonia — ver `references/solid-and-structure.md` antes de introducir una
    abstracción o capa nueva, y explica qué problema real resuelve.

## Protocolo obligatorio ante cada solicitud

### Paso A — Comprender el requerimiento
Objetivo funcional, qué queda incluido/excluido, dependencias y riesgos, si contradice
una decisión arquitectónica existente. Si hay ambigüedad crítica, pregunta antes de
escribir código (usa tu herramienta de preguntas si está disponible; si no, pregunta en
texto).

### Paso B — Inspeccionar el proyecto
Revisa `PROJECT.md` si existe, los archivos directamente relacionados, las
implementaciones actuales antes de sustituirlas, `package.json`/config/rutas/
modelos/pruebas/variables de entorno según corresponda. No asumas que la
documentación está actualizada: verifica los puntos relevantes contra el código real.

### Paso C — Planificar
Para tareas grandes, ambiguas o sensibles: presenta un plan breve (archivos a
crear/modificar/eliminar, dependencias necesarias, riesgos y mitigaciones, pruebas a
ejecutar, criterios objetivos de aceptación) y espera aprobación antes de implementar.
Para tareas pequeñas y de bajo riesgo, puedes ejecutar directo después de inspeccionar.

### Paso D — Implementar
Cambios incrementales y coherentes, sin duplicar lógica, siguiendo las convenciones
del proyecto. Añade o actualiza pruebas cuando corresponda. Nunca ocultes errores con
`catch` vacíos, conversiones inseguras o valores por defecto engañosos. Nunca
desactives un control de seguridad para hacer pasar una prueba.

### Paso E — Verificar
Ejecuta lo que aplique: formato/lint, type checking, pruebas unitarias/integración,
pruebas de autorización y validación, build/compilación, chequeos de seguridad
relevantes (ver `references/security-by-design.md`). Si una verificación no se puede
ejecutar, dilo explícitamente: cuál, por qué, y qué riesgo queda sin cubrir.

### Paso F — Documentar y cerrar
Actualiza `PROJECT.md` (`references/project-md.md`) si la tarea cambió código,
arquitectura, comportamiento, configuración, seguridad, interfaces o el uso del
producto — actualiza solo las secciones afectadas, nunca regeneres el documento
completo. Luego cierra con el resumen de `references/response-format.md`. No marques
una funcionalidad como terminada si no cumple sus criterios de aceptación — ver
`references/testing-and-definition-of-done.md`.

## Referencias (cargar según la tarea)

- `references/solid-and-structure.md` — SOLID aplicado con criterio, estructura de
  archivos/módulos, cuándo SÍ vale una abstracción nueva.
- `references/security-by-design.md` — análisis de superficie de ataque, checklist de
  controles a evaluar, reglas estrictas no negociables.
- `references/phases.md` — desarrollo por fases verificables (0 a 6), de descubrimiento
  a entrega/operación.
- `references/project-md.md` — estructura completa del archivo maestro `PROJECT.md`
  (20 secciones), el mapa visual con diagramas Mermaid y estados de módulo, y las
  reglas de cuándo/cómo actualizarlo.
- `references/testing-and-definition-of-done.md` — qué pruebas aplican y cuándo una
  funcionalidad puede pasar a `TERMINADO`.
- `references/change-control.md` — control de cambios, compatibilidad, qué nunca hacer
  sin autorización explícita (commits, pushes, despliegues, migraciones, versiones
  mayores).
- `references/response-format.md` — formato obligatorio del resumen al cerrar cada
  tarea, y la guía de documentación para el usuario final (no la implementación
  interna).

## Idioma

Respuestas en español, salvo que el proyecto requiera otro idioma para nombres
técnicos, código o documentación.
