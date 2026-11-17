# SOLID, DRY/KISS y estructura de archivos

## Principios SOLID (aplicar cuando aporten claridad, nunca por ceremonia)

- **S — Single Responsibility**: cada módulo, clase o función debe tener una
  responsabilidad principal claramente identificable.
- **O — Open/Closed**: diseña componentes que permitan extender comportamientos sin
  modificar repetidamente código estable.
- **L — Liskov Substitution**: las implementaciones sustitutas deben respetar el
  contrato y las expectativas de sus abstracciones.
- **I — Interface Segregation**: prefiere contratos pequeños y específicos en vez de
  interfaces excesivamente amplias.
- **D — Dependency Inversion**: las reglas de negocio no deben depender directamente
  de detalles de infraestructura cuando sea útil desacoplarlos mediante interfaces o
  contratos.

## Reglas complementarias

- Aplica DRY sin crear abstracciones prematuras.
- Sigue KISS: no añadas complejidad que el requerimiento no justifique.
- Separación de responsabilidades y cohesión alta.
- Evita archivos gigantes, componentes monolíticos y servicios con demasiadas
  funciones.
- Evita crear una clase, interfaz o capa adicional solo para aparentar una
  arquitectura limpia.
- Usa patrones de diseño únicamente cuando resuelven un problema real.
- Respeta las convenciones y el estilo de la base de código existente — no le impongas
  el estilo de otro proyecto.

**Antes de introducir una abstracción importante, explica brevemente qué problema
real resuelve.** Si no hay un problema real todavía (solo "podría hacer falta
después"), no la introduzcas.

## Estructura de archivos y módulos

Antes de definir o modificar la estructura del proyecto:

1. Analiza el stack y la estructura existentes — no impongas una estructura universal
   a todos los stacks; propón una adaptada al framework y justifica las decisiones
   importantes.
2. Identifica los límites entre presentación, lógica de negocio, acceso a datos e
   infraestructura.
3. Agrupa los archivos por funcionalidad cuando esto mejore la cohesión.
4. Mantén responsabilidades y dependencias claramente definidas.
5. Evita dependencias circulares.
6. Mantén configuraciones y secretos fuera del código fuente.
7. Centraliza validaciones, manejo de errores y contratos compartidos cuando sea
   apropiado.
8. Mantén las pruebas cerca del módulo o en una estructura de pruebas coherente con
   el resto del proyecto.

Cuando crees o muevas archivos, actualiza el árbol del proyecto en `PROJECT.md`
(ver `project-md.md`).
