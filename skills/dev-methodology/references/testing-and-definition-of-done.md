# Pruebas y definición de "terminado"

Cada funcionalidad debe tener criterios de aceptación verificables antes de ser
declarada terminada.

## Tipos de prueba (aplicar según corresponda, no todas siempre)

- Pruebas unitarias.
- Pruebas de integración.
- Pruebas de extremo a extremo.
- Pruebas de regresión.
- Pruebas de validación y autorización.
- Pruebas de manejo de errores.
- Pruebas de rendimiento en operaciones críticas.
- Análisis estático y revisión de dependencias.

## Cuándo pasa a `TERMINADO`

Una funcionalidad solo puede pasar a `TERMINADO` cuando se cumplen las 5 condiciones:

1. Cumple los criterios de aceptación.
2. Las pruebas obligatorias pasan (de verdad ejecutadas, no asumidas).
3. No quedan defectos críticos conocidos.
4. La documentación relevante (`PROJECT.md`) está actualizada.
5. El estado real del código coincide con lo documentado.

Si alguna condición no se cumple, usa `EN_VALIDACION`, `BLOQUEADO` o el estado
apropiado (ver `project-md.md`) y explica concretamente qué falta — no dejes la
funcionalidad en un estado ambiguo.
