# Quality gates medibles — checklist pre-entrega

Cada uno de estos es verificable objetivamente, no una opinión. Si no se puede verificar, no es un gate, es una intención — no cuenta.

## UI

- [ ] Contraste WCAG 2.2: ≥4.5:1 texto normal, ≥3:1 texto grande / componentes de UI (`accessibility`)
- [ ] 0% de colores/espaciados/tipografía hardcodeados fuera del set de tokens (`design-dna`, design-system del proyecto)
- [ ] Touch targets ≥44×44px en todo elemento interactivo
- [ ] Estado de foco visible en todo elemento con teclado (no solo mouse)

## UX

- [ ] LCP <2.5s en simulación 4G (`core-web-vitals`)
- [ ] Todo mensaje de error incluye una acción de salida o solución — prohibido un "Something went wrong" sin salida
- [ ] Feedback visual (hover/active/focus) en <100ms para toda acción del usuario
- [ ] Todo estado de carga tiene skeleton/spinner — nunca una pantalla en blanco indefinida
- [ ] Todo estado vacío (empty state) tiene un CTA claro de qué hacer a continuación

## Organización de información / navegación

- [ ] Cualquier función crítica del PRD alcanzable en ≤3 niveles de profundidad
- [ ] Breadcrumbs presentes desde nivel 3 en adelante
- [ ] Estructura de URL semántica que refleje la jerarquía real de la app
- [ ] Si hay >20 elementos navegables, el buscador interno devuelve resultados relevantes en <500ms

## Responsividad

- [ ] Cero scroll horizontal desde 320px hasta 3840px (4K)
- [ ] CLS (Cumulative Layout Shift) <0.1
- [ ] Tipo de input se adapta al dato (teclado numérico para campos numéricos, etc.)
- [ ] Probado en al menos: móvil pequeño (320-375px), tablet (768px), desktop (1440px), ultra-wide (≥2560px)

## Primary directives sin skill dedicada (igual de obligatorias)

- [ ] TLS en tránsito; encriptación en reposo para campos sensibles
- [ ] Privacy by design — ningún campo se captura sin un uso actual definido
- [ ] UTF-8 en todo; fechas en ISO 8601; sin strings hardcodeados en lógica/markup
- [ ] Eventos de analítica definidos para medir si la hipótesis del `epic-hypothesis` se cumplió
- [ ] Rate limiting / cuotas por usuario definidas antes de exponer la feature

## Gatekeeper

Si algún ítem falla: no se marca la tarea como completa. O se corrige, o se declara explícitamente al usuario como tradeoff conocido — nunca se omite en silencio. Ver `verification-before-completion` (superpowers) para el protocolo de verificación con evidencia antes de afirmar éxito.
