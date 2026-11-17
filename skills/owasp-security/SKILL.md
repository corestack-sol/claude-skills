---
name: owasp-security
description: Application security guidance from the OWASP Cheat Sheet Series. Use when writing, reviewing or auditing code that touches authentication, sessions, passwords, JWT, OAuth, authorization, input validation, SQL/NoSQL/command injection, XSS, CSRF, CSP, security headers, file uploads, SSRF, REST/GraphQL/WebSocket/webhook APIs, cryptography, secrets, TLS, Node.js/npm/Next.js, Docker, CI/CD, GitHub Actions, supply chain, or LLM/agent/MCP security. Also for "security review", "is this secure", "harden this", "threat model", "revisión de seguridad", "ciberseguridad", "es seguro esto", "vulnerabilidades", "OWASP".
---

# OWASP security

Rules distilled from 50 cheat sheets of the [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/),
grouped by topic in `references/`. The files hold the specifics (exact parameters,
header values, per-context rules); this page tells you which one to open and how to
apply it.

## How to use it

1. Work out which areas the code touches and open only those reference files. A
   login endpoint needs `authentication.md` and probably `browser.md` for cookies
   and CSRF; it does not need `devsecops.md`.
2. Check the code against the rules in the file, not against memory. The value of
   this skill is the specifics: a work factor, a header value, an encoding context.
3. Report what you find by severity, each finding with the file and line, what an
   attacker can do with it, and the concrete fix. Skip theoretical issues that the
   surrounding code already rules out.
4. When writing new code, apply the rules as you go and say which ones shaped the
   design, so the user can see what was protected and why.

## Which file

| The code involves | Open |
| --- | --- |
| Login, passwords, sessions, cookies, MFA, password reset, JWT, OAuth2/OIDC | `references/authentication.md` |
| Roles, permissions, object ownership, IDOR, mass assignment, multi-tenancy, business rules | `references/authorization.md` |
| User input, SQL/NoSQL queries, shell commands, deserialization, prototype pollution, file uploads, outbound requests to user-supplied URLs, redirects | `references/injection-and-input.md` |
| Rendering untrusted data, innerHTML and friends, CSP, CSRF, security response headers | `references/browser.md` |
| REST, GraphQL, WebSocket, webhooks, error responses, logging | `references/api.md` |
| Encryption, hashing, random values, keys, secrets, environment variables, TLS | `references/crypto-and-secrets.md` |
| Node.js servers, npm dependencies, TypeScript at trust boundaries, Next.js | `references/javascript-and-node.md` |
| Dockerfiles, CI/CD pipelines, GitHub Actions workflows, dependencies and provenance | `references/devsecops.md` |
| LLM prompts, agents with tools, MCP servers or clients, RAG over untrusted content | `references/ai-security.md` |
| Reviewing a codebase or pull request for security, or threat modeling a design | `references/review-and-threat-modeling.md` |

For a full security review of an unfamiliar codebase, start with
`review-and-threat-modeling.md` to set the order of work, then pull in the topic
files as each area comes up.

## Limits

- The reference files are a digest produced with NotebookLM from the cheat sheets,
  not the cheat sheets themselves. Each file links its sources. Before a
  high-stakes decision, or when a value looks surprising, open the source and
  confirm it. Say so when you have not.
- They cover 50 of the series' cheat sheets, chosen for web, API, JavaScript and AI
  work. Topics outside that set (Java, .NET, PHP, Rails, Django, Kubernetes, mobile,
  SAML, XML/XXE, LDAP, among others) are not here: go to
  https://cheatsheetseries.owasp.org/ for those rather than extrapolating.
- The series is revised over time. The digest reflects it as of October 2026.
- These are defensive rules. They are for securing and reviewing systems the user
  owns or is authorized to assess.

Content derived from the OWASP Cheat Sheet Series, licensed CC BY-SA 4.0.
