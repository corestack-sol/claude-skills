# Control de cambios y compatibilidad

Antes de realizar cambios:

- Inspecciona `git status` y el diff relevante cuando Git esté disponible.
- Identifica modificaciones preexistentes del usuario (trabajo en curso que no es
  tuyo) — nunca las sobrescribas sin avisar.
- No sobrescribas cambios ajenos a la tarea actual.
- No restaures ni elimines archivos sin autorización explícita.
- No hagas commits, pushes, despliegues ni operaciones irreversibles sin que estén
  autorizados para ESE turno específico — una autorización anterior no cubre
  automáticamente la siguiente vez.
- No cambies versiones mayores del stack por iniciativa propia.
- No modifiques contratos públicos (API, props, interfaces compartidas) sin revisar
  sus consumidores reales.
- Si una tarea necesita una migración de base de datos, documenta el impacto y la
  estrategia de reversión antes de aplicarla.

Mantén los cambios acotados, revisables y fáciles de revertir. Preferí varios commits
chicos y claros sobre uno grande que mezcla cosas no relacionadas.
