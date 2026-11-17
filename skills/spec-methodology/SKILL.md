---
name: spec-methodology
description: Trabajo guiado por especificaciones (spec-driven) — cada funcionalidad o cambio pasa por Especificar, Planear, Tareas, Revisión de consistencia y aprobación, Implementar y Verificar con evidencia. No se escribe código antes de que la especificación esté aprobada. Solo el método: se conserva el stack del proyecto. Usar al iniciar sesión junto con dev-methodology y en toda funcionalidad o cambio no trivial; los arreglos de una línea no necesitan el ciclo completo.
---

# Metodología guiada por especificaciones

El agente no recibe una petición enorme ni improvisa. Trabaja con entregables pequeños,
aprobados y verificables. El usuario sigue siendo responsable de validar decisiones,
seguridad y resultado. Esta skill define el método; no cambia tecnologías ni instala herramientas
(por ejemplo Spec Kit) salvo que el usuario lo pida.

Complementa a `dev-methodology` (inspeccionar antes de modificar, seguridad desde el diseño,
PROJECT.md): esa define la disciplina general; esta define el ciclo por funcionalidad.

## El ciclo, para cada funcionalidad

### 1. Especificar — qué debe hacer y cómo se comprobará
Describir el problema y el comportamiento esperado, **sin** decidir todavía cómo escribir el código.
- Objetivo.
- Requisitos funcionales numerados (FR-001, FR-002…).
- Reglas de negocio.
- Fuera de alcance (explícito).
- Criterios de aceptación verificables (AC-01, AC-02…): escenario y resultado esperado.
  Un criterio debe poder comprobarse; nunca algo subjetivo como «la pantalla funciona bien».
- Criterio de finalización.

### 2. Planear — cómo se construye
Arquitectura, datos, componentes, contratos e integraciones, **dentro del stack existente**.
Documentar las decisiones relevantes antes de implementar. Cada capa con su responsabilidad.

### 3. Tareas — trabajo pequeño con dependencias claras
Lista ordenada (T001, T002…). Cada tarea es concreta y verificable y está ligada a uno o más
criterios. «Implementar toda la seguridad» es demasiado amplio; «rechazar X cuando Y» es comprobable.

### 4. Analizar consistencia y pedir aprobación
Antes de escribir código, revisar:
- ¿Hay criterios sin tarea asociada?
- ¿Hay algo en el plan que no esté en los requisitos?
- ¿Hay reglas sin forma de verificarse?
- ¿Alguna tarea construye algo fuera del alcance?
- ¿Alguna decisión contradice los principios del proyecto (DESIGN.md, memoria, CLAUDE.md)?
Si hay contradicciones, corregir primero la especificación, el plan o las tareas.
**No resolver en silencio una ambigüedad importante: preguntar.** Presentar especificación,
plan y tareas al usuario y esperar su aprobación.

### 5. Implementar — solo lo aprobado
Antes de modificar archivos: leer la especificación, el plan y las tareas; inspeccionar el
código y las convenciones existentes; identificar los archivos a tocar y por qué.
Durante: mantenerse dentro del alcance; no cambiar dependencias ni arquitectura sin justificarlo;
trabajar por fases si la funcionalidad es grande.

### 6. Verificar — con evidencia, no con palabras
- Ejecutar lo que el proyecto tenga: typecheck, lint, build, pruebas; revisión en navegador
  contra cada criterio cuando es interfaz.
- Reportar: archivos modificados, criterios cubiertos, pruebas fallidas, riesgos pendientes.
- **No declarar una tarea terminada si no se pudo verificar.**

## Skills especializadas por etapa

No se cargan al inicio de la sesión: cada una ocupa contexto mientras dure la sesión. Se invocan
con la herramienta Skill **al llegar a la etapa que las requiere**, según la especialización del
repositorio (frontend, backend, móvil) y de la tarea. Si una skill no está instalada, se dice en
una línea y se sigue.

Antes de aplicar cualquier consejo de una skill, gana lo que el proyecto ya fijó: su sistema de
diseño, su constitución, su `CLAUDE.md` y su stack. Una skill que proponga otra estética, otra
librería u otra arquitectura no se sigue en eso sin aprobación del usuario.

### Frontend web (React, Vite, CSS)
| Etapa | Skill | Cuándo |
|---|---|---|
| 1. Especificar | `mobile-principles`, `desktop-principles` | Al escribir los criterios de una pantalla para teléfono y escritorio: áreas táctiles, sin depender del hover, foco, teclado |
| 5. Implementar | `react-best-practices` | Al escribir o revisar componentes React; las reglas propias de Next.js no aplican si el proyecto no usa Next |
| 5. Implementar | `motion-principles` | Si la pantalla lleva transiciones o animaciones |
| 6. Verificar | `accessibility` | En cada pantalla nueva o cambiada: revisión WCAG 2.2; los hallazgos se vuelven pruebas automáticas |
| 6. Verificar | `web-quality-audit` | Antes de publicar una entrega: rendimiento, accesibilidad y buenas prácticas de la página |

Solo si el proyecto **no** tiene un sistema de diseño fijado, o el usuario pide una propuesta
visual: `frontend-design`, `ui-ux-pro-max`, `design-is`.

### Backend y datos
| Etapa | Skill | Cuándo |
|---|---|---|
| 2. Planear | `database-schema-designer` | Al diseñar tablas, relaciones e índices |
| 5. Implementar | `sql-queries` | Al escribir consultas no triviales |
| 2 y 5 | `owasp-security` | Al diseñar o tocar autenticación, sesiones, autorización, validación de entradas, secretos o una API |

### Móvil
| Etapa | Skill | Cuándo |
|---|---|---|
| 1. Especificar | `mobile-principles` | Criterios de cada pantalla de la app |
| 5. Implementar | `skills:react-native-best-practices`, `expo:*`, `compose-*`, `swiftui-*` | Solo la que corresponda al framework real del proyecto |

### Cualquier especialización
| Etapa | Skill | Cuándo |
|---|---|---|
| Si algo falla | `systematic-debugging` | Ante una prueba en rojo, un error o un comportamiento inesperado, antes de proponer un arreglo |
| 5 y 6 | `owasp-security` | Al tocar inicio de sesión, formularios, el servidor o la configuración de seguridad |
| Publicar | `cloudflare` | Si el proyecto publica en Cloudflare y la tarea toca Workers, `wrangler` o el despliegue |

## Si algo falla
1. Identificar el criterio incumplido.
2. Determinar si el problema está en la especificación, la implementación o la prueba.
3. Corregir la causa (si el requisito estaba mal, corregir primero la especificación).
4. Volver a verificar, incluidas regresiones.
5. Registrar el resultado.
No cambiar una prueba solo para que el código pase.

## Control de cambios
Cada cambio debe poder ligarse a una tarea y a un criterio («¿qué requisito justifica esto?»).
Commits solo cuando el usuario lo pida, revisando los archivos incluidos; nunca secretos ni `.env`.

## Incrementos
No construir todo de una vez. Cada fase o funcionalidad pasa por su propio ciclo completo.
Empezar por una funcionalidad vertical, demostrar que especificación, tareas, código y
verificación están alineados, y luego replicar.

## Dónde guardar los artefactos
Si el proyecto no define otra ubicación, proponer:
```
specs/
  constitution.md          principios y restricciones globales
  NNN-nombre/
    spec.md                comportamiento, requisitos, criterios
    plan.md                arquitectura y decisiones
    tasks.md               tareas y dependencias
```
Confirmar con el usuario antes de crear la estructura. La especificación no sustituye al código
ni a las pruebas: los tres evolucionan coordinados.

## Regla de oro
No «hazme X». Sí: «implementa esta funcionalidad, con estos requisitos, estos límites, estos
criterios de aceptación y esta verificación; no avances si hay una contradicción sin resolver».
