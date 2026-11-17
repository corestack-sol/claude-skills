# El archivo maestro PROJECT.md

Mantén un único archivo Markdown maestro en la raíz del repositorio: `PROJECT.md`. Es
la fuente principal de documentación del proyecto — debe permitir que otro
desarrollador continúe el proyecto sin depender del historial del chat.

## Secciones que debe contener (separadas claramente)

1. Resumen del producto.
2. Alcance y funcionalidades.
3. Stack tecnológico y versiones relevantes.
4. Arquitectura general.
5. Diagrama de arquitectura (Mermaid).
6. Árbol actualizado de archivos.
7. Módulos y estado de implementación.
8. Requisitos funcionales y no funcionales.
9. Decisiones arquitectónicas y sus justificaciones.
10. Contratos entre módulos, endpoints y modelos de datos relevantes.
11. Configuración y variables de entorno, **sin valores secretos**.
12. Medidas de seguridad implementadas y verificaciones realizadas.
13. Instrucciones de instalación y ejecución.
14. Guía de uso para el usuario final.
15. Flujos funcionales principales.
16. Errores habituales y resolución de problemas.
17. Pruebas ejecutadas y resultados.
18. Limitaciones conocidas y tareas pendientes.
19. Historial breve de cambios.
20. Procedimientos de despliegue, respaldo y recuperación cuando correspondan.

## Mapa visual y estado de los módulos (Mermaid)

Mantén un diagrama actualizado de la arquitectura y del progreso, compatible con
Markdown:

1. Un diagrama de arquitectura y dependencias.
2. Un diagrama de flujo de los procesos principales cuando sea útil.
3. Un diagrama de estado de módulos con identificadores legibles.

Estados a usar (consistentes en todo el documento):

- `PENDIENTE` — todavía no implementado.
- `EN_PROCESO` — implementación activa o incompleta.
- `IMPLEMENTADO` — código creado, pero validación incompleta.
- `EN_VALIDACION` — esperando pruebas o correcciones.
- `TERMINADO` — criterios de aceptación cumplidos y verificaciones aprobadas.
- `BLOQUEADO` — depende de una decisión, acceso o recurso pendiente.

En el diagrama, diferencia visualmente los estados mediante clases Mermaid y una
leyenda. Mantén además una tabla accesible con: módulo, responsable, estado,
dependencias y criterio pendiente.

**No inventes porcentajes de progreso.** Si los estimas, explica el método y marca el
dato como aproximado.

Actualiza el diagrama cada vez que cambien los módulos, sus dependencias o sus
estados.

## Reglas de mantenimiento

- Actualiza únicamente las secciones afectadas por cada tarea — **nunca regeneres el
  documento completo** si basta con modificar unas pocas secciones.
- Evita duplicar información y conservar instrucciones contradictorias entre
  secciones.
- Mantén el historial de cambios breve: fecha, tarea, archivos principales y
  resultado.
- No registres cada prompt conversacional. Registra los cambios que tengan efecto en
  el producto o en las decisiones del proyecto. Si un prompt no produjo cambios, no
  agregues una entrada de historial.
- Si una tarea modifica el comportamiento de una funcionalidad, actualiza también su
  documentación de usuario (sección 14).
- Si un requisito se modifica, actualiza su descripción y el estado correspondiente.
- Si una prueba falla, registra el fallo relevante — **no marques la funcionalidad
  como terminada.**
- No documentes como implementado algo que solo se ha propuesto.

## Cuándo actualizar

Al final de cada tarea significativa, después de implementar y ejecutar las
verificaciones disponibles. No lo actualices automáticamente después de cada mensaje
del usuario si todavía no se realizó ningún cambio real. Si una tarea es larga, podés
registrar un estado provisional durante la ejecución, pero el cierre debe reflejar el
resultado comprobado, no el plan original.
