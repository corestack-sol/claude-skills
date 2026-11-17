# Índice completo de skills por dominio

Catálogo de referencia — consultar cuando el dispatch matrix de `SKILL.md` no cubra el caso exacto.

## Ingeniería / desarrollo

- **superpowers** (14): `brainstorming` (gate obligatorio pre-implementación), `writing-plans`, `executing-plans`, `test-driven-development`, `systematic-debugging`, `requesting-code-review`, `receiving-code-review`, `finishing-a-development-branch`, `dispatching-parallel-agents`, `subagent-driven-development`, `using-git-worktrees`, `verification-before-completion`, `using-superpowers`, `writing-skills`
- `mcp-builder` — integraciones externas vía MCP
- `webapp-testing` — Playwright, probar la app real en navegador
- `sql-queries` — todos los dialectos de warehouse
- `database-schema-designer`, `react-best-practices`, `best-practices` (seguridad/compat), `cloudflare` (infra)
- claude-mem: `mem-search`, `smart-explore`, `learn-codebase`, `make-plan`, `do`, `pathfinder`, `oh-my-issues`, `standup`, `version-bump` — memoria cruzada de sesiones, exploración/ejecución de planes
- `mcp` reference (`claude-api`) — pricing, límites, modelos al integrar la API de Claude

## Diseño / Frontend / UX

- `frontend-design` — dirección estética, evitar defaults genéricos de IA
- `ui-ux-pro-max` (design, design-system, ui-styling, brand, banner-design, slides)
- `design-dna` — extrae tokens/estilo de referencias reales a JSON reusable
- `design-audit`, `design-is` (10 principios de Dieter Rams)
- `accessibility` (WCAG 2.2), `core-web-vitals`, `performance`, `web-quality-audit`, `web-perf`
- `mobile-principles`, `desktop-principles` — touch targets, hover states, thumb zones, multi-window
- Animación: `gsap` (core, timeline, scrolltrigger, plugins, react, frameworks, performance, utils), `framer-motion`, `motion-design`, `motion-principles`, `css-native`
- 3D/generativo: `threejs-*` (fundamentals, geometry, materials, lighting, animation, shaders, textures, loaders, postprocessing, interaction — riesgo Critical revisado y aprobado por el usuario, r3f), `canvas-generative`
- Apps nativas: `swiftui-graphics`, `swiftui-motion`, `compose-graphics`, `compose-motion`, `compose-multiplatform`
- `cast`, `paint` — orquestadores de creatividad (genjutsu)
- `dataviz`, `artifact-design`, `artifact-diagramming` — gráficas y documentos visuales
- `seo` — SEO técnico de la implementación (meta tags, sitemap, structured data)

## Producto / Estrategia

- `prd-development`, `roadmap-planning`, `epic-hypothesis`, `epic-breakdown-advisor`, `user-story`, `user-story-mapping`, `user-story-mapping-workshop`, `user-story-splitting`, `prioritization-advisor`
- `discovery-process`, `discovery-interview-prep`, `jobs-to-be-done`, `problem-framing-canvas`, `problem-statement`, `opportunity-solution-tree`, `lean-ux-canvas`
- `customer-journey-map`, `customer-journey-mapping-workshop`, `proto-persona`, `stakeholder-identification`, `stakeholder-mapping`, `stakeholder-engagement-advisor`
- `feature-investment-advisor`, `derisk-measurement-advisor`, `recommendation-canvas`, `pol-probe`, `pol-probe-advisor`
- `market-landscape-scan`, `competitor-teardown`, `competitive-analysis-process`, `competitive-research-snapshot`, `competitive-intel-watch`, `battle-card-builder`, `intelligence-collection-disciplines`, `intel-discipline-advisor`, `company-intel`, `company-research`
- `tam-sam-som-calculator`, `ansoff-matrix`, `porters-five-forces`, `pestel-analysis`, `pestel-delta-monitor`, `swot-analysis`
- `positioning-statement`, `positioning-workshop`
- `product-lifecycle-plays`, `lifecycle-play-advisor`, `eol-checklist`, `eol-internal-enablement`, `eol-message`, `eol-process`, `eol-readiness-advisor`, `eol-stakeholder-sequence`
- `ai-shaped-readiness-advisor`, `context-engineering-advisor`, `agent-orchestration-advisor`

## Marketing / Growth

- `programmatic-seo`, `ai-seo`, `cro`, `ad-creative`, `marketing-psychology`, `pricing`
- `acquisition-channel-advisor`, `organic-growth-advisor`, `saas-revenue-growth-metrics`, `saas-economics-efficiency-metrics`, `finance-metrics-quickref`, `finance-based-pricing-advisor`, `pricing-packaging-tracker`
- `business-health-diagnostic`

## Social y Contenido

- `social`, `copywriting`, `content-strategy`, `video`
- `pillar-content-architecture`, `email-sequences`
- `press-release`, `voice-of-customer-miner`

## Finanzas

- `dcf-model`, `3-statement-model`, `lbo-model`, `comps-analysis`, `pitch-deck` — uso institucional/M&A. Sobredimensionado para una feature normal; reservar para fundraising o valuación real.

## Operaciones

- `process-doc` (SOP/RACI/flowcharts), `incident-response` (triage + postmortem), `business-case-development`
- `internal-comms`

## Legal / Compliance

- `review-contract`, `triage-nda`, `legal-risk-assessment`, `compliance-check`

## Liderazgo ejecutivo

- `executive-onboarding-playbook`, `director-readiness-advisor`, `vp-cpo-readiness-advisor`, `altitude-horizon-framework`, `incoming-request-advisor`

## Documentos / oficina

- `docx`, `xlsx`, `pptx`, `pdf` — generación/edición de documentos reales cuando el entregable no es código

## Nota sobre huecos sin skill instalada

Seguridad/privacidad, i18n, analítica/instrumentación, onboarding/empty states, deuda técnica, y escalabilidad de infraestructura no tienen skill dedicada — se gobiernan por las "Primary directives" de `SKILL.md`, no por invocación de skill. No inventar una skill que no existe; aplicar la regla directamente.
