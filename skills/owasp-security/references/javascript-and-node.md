# JavaScript, Node.js, npm and Next.js

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Nodejs Security](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html)
- [NPM Security](https://cheatsheetseries.owasp.org/cheatsheets/NPM_Security_Cheat_Sheet.html)
- [JavaScript and TypeScript Security](https://cheatsheetseries.owasp.org/cheatsheets/JavaScript_and_TypeScript_Security_Cheat_Sheet.html)
- [Nextjs Security](https://cheatsheetseries.owasp.org/cheatsheets/Nextjs_Security_Cheat_Sheet.html)

## Rules

### Node.js Security Cheat Sheet
- **Command Execution & Process Invocation**:
  - **Avoid **`child_process.exec()`: Do not use `child_process.exec()` with untrusted inputs, as it passes strings directly to `/bin/sh` or the system shell, making the application vulnerable to OS command injection.
  - **Use Process APIs with Argument Arrays**: Use `child_process.execFile()` or `child_process.spawn()` and pass command arguments as an array of discrete strings, preventing shell metacharacter expansion (`&`, `|`, `;`, `$`, etc.).
  - **Runtime Permission Controls**: Enable the Node.js Permission Model (`node --permission`) and restrict process execution capabilities via `--allow-child-process`.
- **Brute Force & Rate Limiting**:
  - **Delay Failed Requests**: Implement packages like `express-brute` or `express-bouncer` on authentication routes to introduce progressive delays after failed login attempts.
  - **IP-Based Throttling**: Use `rate-limiter` middleware to cap the maximum number of requests an IP can make to sensitive endpoints (e.g., limiting `/login` to 5 attempts per 500 seconds) and return `429 Too Many Requests` when limits are exceeded.
  - **CAPTCHA Challenge**: Integrate CAPTCHA solutions like `svg-captcha` on sensitive forms to block automated credential stuffing scripts.
- **Node.js Permission Model**:
  - Enable the stable permission model using `node --permission`.
  - Restrict file system reads using `--allow-fs-read` and writes using `--allow-fs-write` (e.g., `node --permission --allow-fs-read=/uploads/ index.js`) to mitigate Local File Inclusion (LFI) and path traversal.
  - Selectively grant access to worker threads (`--allow-worker`), native addons (`--allow-addons`), and WASI modules (`--allow-wasi`).
  - **Symlink Bypass Caution**: Ensure allowed directory paths do not contain relative symbolic links, as symlinks pointing outside allowed directories can bypass file system restrictions.
- **Event Loop & Memory Hardening**:
  - **Prevent Loop Blocking**: Keep CPU-intensive operations asynchronous or offload them to worker threads to prevent blocking the single-threaded event loop.
  - **Event Loop Monitoring**: Use `toobusy-js` to track event loop latency; when response times cross safety thresholds, return an HTTP `503 Server Too Busy` error to remain responsive under heavy traffic.
  - **Request Size Limits**: Set strict request body limits across all POST/PUT routes using `raw-body` or Express body parsers (e.g., `limit: '1kb'`) to prevent memory exhaustion DoS.
- **Most Common Mistakes**:
  - Concatenating user inputs into `child_process.exec()` strings instead of using `execFile`/`spawn` with parameter arrays.
  - Using in-memory rate-limiter stores (`MemoryStore`) in production cluster environments.
  - Assuming path restrictions in the Permission Model protect against relative symbolic link traversal.

### NPM Security & Software Supply Chain Security Cheat Sheets
- **Lockfile Enforcement**:
  - Always commit `package-lock.json` or `yarn.lock` to source control to ensure deterministic builds across all environments.
  - In CI/CD and deployment pipelines, strictly enforce lockfile compliance by running `npm ci` or `yarn install --frozen-lockfile`. These commands abort installation if `package.json` mismatches the lockfile, preventing unintended package upgrades.
- **Run-Script Surface Minimization**:
  - Disable lifecycle scripts during dependency installation by appending `--ignore-scripts` to `npm install` or configuring `ignore-scripts=true` in `.npmrc` to neutralize malicious `postinstall` hooks.
  - Use an allowlist tool like `@lavamoat/allow-scripts` in `package.json` to explicitly authorize lifecycle scripts only for vetted packages that require build steps (e.g., `sharp`).
- **Publishing Hygiene & Secret Protection**:
  - Prevent leaking API keys or `.env` files when publishing packages to public or private registries.
  - **Ignore File Pitfall**: If both `.gitignore` and `.npmignore` exist in a project directory, the npm CLI ignores `.gitignore` and packs everything not listed in `.npmignore`. Use the `files` array in `package.json` or test with `npm publish --dry-run` to inspect tarball contents prior to publishing.
- **Typosquatting & Slopsquatting Defenses**:
  - Protect against typosquatting (malicious packages using names similar to legitimate packages like `cross-env`).
  - Protect against "slopsquatting" (attackers publishing malicious packages under non-existent names hallucinated by AI coding assistants like Copilot or ChatGPT).
  - **Verification Controls**: Run `npm view <package-name>` before installing any new or AI-suggested package, check package download counts, verify active GitHub repository links, and automate `npm view` checks in CI.
- **Dependency Confusion Mitigation**:
  - Use **scoped package names** (e.g., `@yourorg/package-name`) for all internal private modules.
  - Configure `.npmrc` to explicitly bind the organization scope to your private artifact registry (`@yourorg:registry=https://your-private-registry.example.com`).
  - Reserve internal package names on the public npm registry by publishing empty placeholder packages.
- **Authentication, 2FA & Token Management**:
  - Enable 2FA on npm accounts using `npm profile enable-2fa auth-and-writes`.
  - Use **Trusted Publishing with OIDC** in CI/CD pipelines to authenticate package publishing via short-lived, workflow-bound tokens instead of storing static, long-lived access tokens.
  - When static tokens are unavoidable, issue granular access tokens restricted to specific packages, set short expiration windows, restrict source IP ranges, and store them securely in CI secret stores.
- **Most Common Mistakes**:
  - Running `npm install` instead of `npm ci` in CI/CD pipelines, allowing unpinned transitive package updates.
  - Maintaining both `.gitignore` and `.npmignore`, inadvertently publishing `.env` or internal configuration files.
  - Blindly installing packages suggested by AI assistants without running `npm view` or checking download/repository metadata.
  - Copy-pasting insecure documentation README examples (e.g., `Math.random()`, unanchored regexes, weak key derivation) directly into production applications.

### JavaScript and TypeScript Security Cheat Sheet
- **TypeScript Trust Boundary Caveats**:
  - TypeScript types are completely erased at compile time; TypeScript alone provides **zero runtime security or type safety** against untrusted external data (API payloads, query params, `postMessage` events).
- **Avoiding False Confidence in Types**:
  - Avoid using `any`, as it disables type checking across the application; use `unknown` for unvalidated data to force explicit type narrowing before use.
  - Do not use type assertions (`as UserData`) or non-null assertions (`!`) on untrusted data, as they silence compiler warnings without validating runtime values.
- **Runtime Schema Validation**:
  - Validate all untrusted input at every trust boundary using a runtime schema validator (e.g., Zod, Valibot, or Ajv).
  - Infer TypeScript types directly from runtime schemas to ensure static types and runtime validation rules remain perfectly synchronized.
- **Dynamic Code Execution Surface**:
  - Never build or execute code from dynamic strings; avoid `eval()`, `new Function()`, `with`, and string arguments in `setTimeout`/`setInterval`.
  - Block string compilation in browsers using Content Security Policy directives that omit `'unsafe-eval'`.
  - Enable strict mode (`"use strict"` or ES modules) and enforce linter rules (`no-eval`, `no-implied-eval`, `no-new-func`).
- **ReDoS (Evil Regex) Prevention**:
  - Avoid regular expression patterns with nested quantifiers (e.g., `(a+)+`) or ambiguous alternations (`^(a|aa)*$`) that trigger exponential backtracking.
  - Keep regular expressions linear, cap the maximum length of untrusted input strings before matching, and use linear-time regex engines.
- **Cross-Origin Messaging (**`postMessage`):
  - In message receivers, validate `event.origin` against an explicit allowlist and validate `event.data` schemas before executing logic.
  - When sending messages, explicitly specify `targetOrigin`—never pass `"*"` when transmitting sensitive data.
- **Most Common Mistakes**:
  - Using TypeScript type assertions (`as UserDTO`) on raw HTTP responses, assuming TypeScript validates data at runtime.
  - Trusting `postMessage` handlers without verifying `event.origin` or validating payload schemas.
  - Evaluating complex regular expressions against un-capped input strings, allowing ReDoS attacks to block the main event loop.

### Next.js Security Cheat Sheet
- **Server Entry Point Classification**:
  - Classify every server entry point (Proxy/Middleware, Server Actions, Route Handlers, Pages API Routes, Server Components, Draft Mode) according to its intended audience and access permissions.
  - Enforce authorization at a boundary that every applicable request path traverses (e.g., a server-only Data Access Layer / DAL).
- **Middleware / Proxy Authorization Limits**:
  - Use Proxy (`proxy.ts` / `middleware.ts`) strictly for optimistic redirects and request filtering; **never rely on Proxy/Middleware as the sole authorization boundary**.
  - Matcher configurations can omit routes, and framework-level proxy bypasses (such as CVE-2025-29927) can leave endpoints exposed if authorization is not enforced at the data source or handler level.
- **Server Actions Security**:
  - Treat every Server Action as an independently callable HTTP POST endpoint.
  - Validate all action arguments as untrusted input, derive authentication context on the server (do not accept identity claims in arguments), and perform object-level authorization on target resources.
  - Keep `serverActions.allowedOrigins` tightly restricted to trusted origins.
- **Route Handlers & Pages API Routes**:
  - Treat Route Handlers (`route.ts`) and API Routes (`pages/api`) as public HTTP endpoints requiring explicit authentication and authorization.
  - Do not assume a Route Handler is private because it is called by a Server Component; if no external client requires the HTTP API, invoke the DAL directly from the Server Component instead.
- **Server vs. Client Components Data Exposure**:
  - The App Router serializes props passed to Client Components, and the Pages Router exposes `getServerSideProps`/`getStaticProps` props in initial HTML payloads.
  - Never pass database ORM records, session objects, or secrets into client props; construct minimal Data Transfer Objects (DTOs) returning only authorized fields.
  - Import `server-only` in sensitive data modules as a build-time guard to prevent accidental client-side imports.
- **Environment Variable Exposure (**`NEXT_PUBLIC_`):
  - Environment variables prefixed with `NEXT_PUBLIC_` are bundled into client JavaScript files and exposed to the public.
  - Reserve `NEXT_PUBLIC_` strictly for non-sensitive public configuration values.
- **Image & Redirect Configuration**:
  - Restrict `images.remotePatterns` in `next.config.js` tightly, as omitted fields act as broad wildcards.
  - Keep `images.dangerouslyAllowLocalIP` disabled unless explicitly reviewed, as it exposes the internal network to SSRF attacks.
  - Resolve external rewrites and redirects from fixed server-side allowlists.
- **CSP with Nonces**:
  - Generate unpredictable, per-request nonces in Proxy/Middleware and pass them to rendering components.
  - Note that nonce-based CSP forces dynamic rendering, disables static optimization / ISR for those pages, and prevents Partial Prerendering (PPR) due to nonce caching conflicts.
- **Most Common Mistakes**:
  - Relying exclusively on `middleware.ts` for route protection without enforcing authorization in Server Actions or Route Handlers.
  - Passing raw database ORM objects as props to Client Components, exposing sensitive background fields in serialized React Server Component streams.
  - Prefixing sensitive server API keys or database secrets with `NEXT_PUBLIC_`, leaking them in client JavaScript bundles.
  - Leaving `images.remotePatterns` unconstrained or enabling `dangerouslyAllowLocalIP`, opening internal microservices to SSRF.
