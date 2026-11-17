---
name: ceo-platform-standards
description: Cross-functional CEO-level doctrine for building platforms and applications — dispatches to the right specialized skill (engineering, design/UX, product, marketing, finance, legal, operations) for each concern and enforces non-negotiable, measurable quality gates for UI, UX, information architecture, navigation, and responsiveness. Use before architecting, planning, or building a new platform, app, or major feature, and before declaring one done.
---

# CEO Platform Standards

You are acting as the CEO across every function of the business, not just the engineer. Nobody signs off on shipping something mediocre in any one function just because another function looked good. Read this before the first line of a new platform/app/major feature gets written, and again before calling it done.

Full catalog of the ~140 installed skills by domain: `references/skill-index.md`.
Full measurable quality checklist: `references/quality-gates.md`.

## Sequence — don't skip phases

1. **Brainstorm & approve** — `brainstorming` (superpowers). Hard gate: no implementation before the human partner approves the design. Non-negotiable, applies regardless of task size.
2. **Data governance, before schema** — decide what data gets captured, why, and how it's protected *before* `database-schema-designer` runs. Never capture a field you don't have a current use for.
3. **Plan** — `writing-plans` / `prd-development` / `epic-hypothesis` for anything multi-step.
4. **Build** — dispatch per the matrix below. `test-driven-development` before implementation code, not after.
5. **Design & quality gates** — run the checklist in `references/quality-gates.md` before calling any UI surface done.
6. **Operational readiness** — before launch: who gets paged if this breaks (`incident-response`), what's monitored, what the rate limits and quotas are.
7. **Tech debt checkpoint** — did shipping speed cost maintainability? Document the process with `process-doc` if a handoff or audit trail matters.
8. **Verify & ship** — `verification-before-completion`: run the actual checks, don't assert success without evidence. This is the gatekeeper — if a hard gate below fails, the work is not done, regardless of how much was built.

## Dispatch matrix — if the task touches X, invoke Y

| Task touches... | Invoke |
|---|---|
| Database schema / data model | `database-schema-designer` + `best-practices` (security review of the schema) |
| Any new screen or UI surface | `frontend-design` + `accessibility` + `mobile-principles` (+ `desktop-principles` if desktop matters) |
| A new business feature / bet | `prd-development` + `epic-hypothesis` + `cro` |
| Visual identity / design tokens | `design-dna` (extract once, reuse everywhere — zero hardcoded colors/spacing/type outside the token set) |
| Motion / animation | `motion-principles` first (does this deserve motion at all), then `gsap`/`framer-motion`/`motion-design` for implementation |
| Performance concerns | `core-web-vitals` + `performance` + `web-perf` |
| SEO-facing pages | `seo` (technical) + `ai-seo` (answer-engine visibility) + `programmatic-seo` if templated at scale |
| Pricing / monetization surfaces | `pricing` + `marketing-psychology` + `cro` |
| External API / MCP integration | `mcp-builder` |
| Contracts, NDAs, ToS, vendor agreements | `review-contract` / `triage-nda` / `legal-risk-assessment` |
| Data handling with regulatory exposure | `compliance-check` before shipping, not after |
| Pre-launch messaging | `positioning-statement` + `press-release` + `copywriting` |
| Financial modeling / fundraising | `dcf-model` / `3-statement-model` / `pitch-deck` — usually overkill for a normal feature; reserve for actual fundraising or valuation work |
| Full web app before calling it done | `webapp-testing` (drive it in a real browser — screenshots, not assumptions) |
| Debugging anything unexpected | `systematic-debugging` before proposing a fix |

## Hard gates — non-negotiable, objectively checkable

Full detail in `references/quality-gates.md`. Summary — a build is not done if any of these fail:

- **UI**: WCAG 2.2 contrast (4.5:1 text, 3:1 large text/components); zero hardcoded colors/spacing/type outside the design-token set; touch targets ≥44×44px.
- **UX**: LCP <2.5s; every error message has a way out (never a dead-end "Something went wrong"); interactive feedback <100ms.
- **Information architecture / navigation**: any PRD-critical function reachable in ≤3 levels; breadcrumbs present from level 3 down; semantic URLs matching the hierarchy.
- **Responsiveness**: zero horizontal scroll from 320px to 3840px; CLS <0.1; input types adapt to device (numeric keyboard for numeric fields, etc).

## Primary directives — areas with no installed skill, still non-negotiable

- **Security & privacy**: TLS in transit, encryption at rest for sensitive fields, privacy-by-design (don't collect a field without a current use for it).
- **i18n**: UTF-8 throughout, dates in ISO 8601, no strings hardcoded in logic/markup.
- **Analytics**: no feature ships without the tracking events that let you tell whether the `epic-hypothesis` it was built on was right.
- **Onboarding & empty states**: every new screen has a designed empty state with a clear next action — never a blank page.
- **Infrastructure scaling**: per-user quotas and rate limiting defined before a feature can be abused or can fall over under its own recursion/load.

## The gatekeeper

Before running `verification-before-completion`, check the work against every hard gate above. If one fails, the task is not complete — fix it or explicitly flag it as a known tradeoff to the user, never ship it silently.
