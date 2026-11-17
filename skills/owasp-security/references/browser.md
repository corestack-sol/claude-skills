# Browser security: XSS, CSP, CSRF and headers

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Cross Site Scripting Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [DOM based XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- [Content Security Policy](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Cross Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [HTTP Headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)

## Rules

### Cross Site Scripting (XSS) Prevention Cheat Sheet
- **Contextual Output Encoding Rules**:
  - **HTML Body Context** (e.g., `<div>$var</div>`): Use HTML entity encoding to convert special characters into safe entity representations: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`, and `'` → `&#x27;`.
  - **HTML Attribute Context** (e.g., `<input value="ENCODED_DATA">`): Always enclose attribute values in single (`'`) or double (`"`) quotes. Use a library or framework encoder for quoted text attributes. (Note: HTML attribute encoding alone does **not** make event-handler attributes safe).
  - **JavaScript Data Values Context** (e.g., `<script>const msg = "ENCODED_DATA";</script>`): Insert untrusted data **only inside quoted string variables** using an encoder documented for both JavaScript string syntax and the enclosing HTML `<script>` tag.
  - **CSS Context** (e.g., `<div style="color: $var">`): Place variables **only** within CSS property values—never in property names or selectors. Use CSS hex encoding (`\XX` or `\XXXXXX` zero-padded).
  - **URL Parameters Context** (e.g., `<a href="/search?q=ENCODED_DATA">`): Use percent-encoding (`%HH`) for untrusted query parameter values. When constructing URLs dynamically in client JavaScript, use `encodeURIComponent()` for parameter values.
  - **Double Encoding for URLs in Attributes**: When placing a URL parameter inside an HTML attribute (e.g., `<a href="...">`), first apply URL parameter encoding to the parameter value, and then apply HTML attribute encoding to the complete attribute URL string.
- **Contexts Never Safe for Untrusted Data (Dangerous Contexts)**:
  - Never insert untrusted data directly into raw `<script>` blocks (as executable code, identifiers, or unquoted expressions), HTML comments (`<!--... -->`), `<style>` tags, attribute names (`<div $var=test />`), or HTML tag names (`<$var href="..." />`).
  - Avoid inserting untrusted data into callback function names, `javascript:` URLs in CSS/attributes, event-handler attribute bodies (`onclick="$var"`), or string-evaluation sinks (`eval()`, `setInterval()`, `setTimeout()`).
- **HTML Sanitization & Recommended Libraries**:
  - When users must author rich HTML (e.g., in WYSIWYG editors), output encoding breaks intended rendering. Use HTML sanitization to strip dangerous elements while preserving safe markup.
  - **Recommended Library**: Use **DOMPurify** (`DOMPurify.sanitize(dirty)`) (or `sanitize-html` for server-side Node.js applications).
  - **Sanitization Guidelines**: Never modify or mutate an HTML string *after* sanitizing it. Keep sanitization libraries regularly patched to protect against new browser parsing bypasses.
- **Most Common Mistakes & Anti-Patterns**:
  - **Sole Reliance on Content Security Policy (CSP)**: CSP is a defense-in-depth layer, not a replacement for output encoding. Relying solely on CSP fails if older user browsers lack full CSP Level 2/3 directive support or if policy exceptions exist.
  - **Reliance on HTTP Interceptors / Filters**: Applying generic output encoding or sanitization in global servlet filters or Spring interceptors fails because interceptors lack context awareness (they cannot distinguish between HTML, JavaScript, or CSS contexts). This leads to double-encoding bugs, broken UI rendering, un-encoded JavaScript contexts, and zero protection against DOM-based XSS or untrusted database/REST service inputs.

### DOM-Based XSS Prevention Cheat Sheet
- **Safe Sinks vs. Dangerous Sinks**:
  - **Dangerous HTML/Code Sinks**: Avoid assigning untrusted strings to `innerHTML`, `outerHTML`, `document.write()`, `document.writeln()`, `eval()`, `Function()`, or `setTimeout`/`setInterval` with string arguments.
  - **Dangerous URL/Navigation Sinks**: Avoid assigning untrusted input directly to `location`, `window.location`, or `href` attributes containing `javascript:` pseudo-protocols.
  - **Safe Sinks & Alternatives**:
    - Populate text content safely using `element.textContent` or `innerText`.
    - Construct elements dynamically using `document.createElement()`, `document.createTextNode()`, and `element.appendChild()`.
    - Append text safely using `element.insertAdjacentText("beforeend", data)`.
    - Assign non-executing HTML text attributes directly using `element.setAttribute(attrName, value)` or property assignments (`formfield.value = data`), keeping attribute names fixed and safe (`align`, `alt`, `class`, `title`, `value`, `width`, etc.).
- **Framework Escape Hatches**:
  - Framework auto-escaping protects standard templates, but explicit "escape hatches" bypass internal defenses and must never receive raw, untrusted user strings:
    - **React**: `dangerouslySetInnerHTML={{ __html: cleanData }}`.
    - **Angular**: `bypassSecurityTrustHtml()`, `bypassSecurityTrustScript()`, or `bypassSecurityTrustUrl()`.
    - **Vue**: `v-html="cleanData"`.
    - **Lit / Polymer**: `unsafeHTML`, `inner-h-t-m-l`, or `htmlLiteral`.
  - Always pass untrusted data through a vetted HTML sanitizer (e.g., DOMPurify) before passing it to any framework escape hatch.
- **Trusted Types**:
  - Enable Trusted Types in Chromium-based browsers via HTTP header: `Content-Security-Policy: require-trusted-types-for 'script'`.
  - Trusted Types locks down DOM XSS sinks (`innerHTML`, `outerHTML`, `document.write`, `script.src`), causing them to reject plain string assignments and requiring values to pass through a vetted `TrustedHTML` or `TrustedScriptURL` policy. It eliminates entire classes of DOM-based XSS vulnerabilities.
- **Most Common Mistakes & Guidelines**:
  - **Passing Untrusted Input to **`setAttribute()`** for Event Handlers**: Calling `element.setAttribute("onclick", data)` or setting `x.href = "javascript:..."` implicitly coerces strings into executable JavaScript code, bypassing standard JavaScript encoding. Assign trusted callback functions directly to properties (`element.onclick = function() {... }`) instead.
  - **Using **`eval()`** to Parse JSON**: Never pass JSON strings to `eval()`; use `JSON.parse()` to deserialize JSON safely without executing arbitrary code.
  - **Placing Untrusted Data on the Left Side of Expressions**: Avoid `window[untrustedInput] = value`, which allows attackers to overwrite internal window properties or global attributes. Use explicit allow-lists to constrain object key access.
  - **Double Encoding in DOM Execution Contexts**: Applying HTML attribute encoding inside JavaScript DOM execution contexts (e.g., `setAttribute("value", encodeForHTMLAttribute(val))`) causes double-encoding bugs (rendering `&amp;` on screen). Use JavaScript string encoding when operating within client-side JavaScript execution contexts.

### Content Security Policy (CSP) Cheat Sheet
- **Recommended Strict CSP Policies**:
  - **Nonce-Based Strict Policy (Optimal for Dynamic Apps)**: `Content-Security-Policy: script-src 'nonce-{RANDOM}' 'strict-dynamic'; object-src 'none'; base-uri 'none';`
  - **Hash-Based Strict Policy (Optimal for Static Pages with Inline Scripts)**: `Content-Security-Policy: script-src 'sha256-{HASHED_INLINE_SCRIPT}' 'strict-dynamic'; object-src 'none'; base-uri 'none';`
  - **Basic Non-Strict Fallback Policy**: `Content-Security-Policy: default-src 'self'; frame-ancestors 'self'; form-action 'self';` *Tightened variant*: `Content-Security-Policy: default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'; form-action 'self';`
- **Directive Breakdown & What Each Directive Does**:
  - `script-src`: Restricts the locations and mechanisms from which JavaScript can execute. In strict policies, it uses `'nonce-{RANDOM}'` or `'sha256-...'` alongside `'strict-dynamic'`. Granular script controls include `script-src-elem` (script tags/blocks) and `script-src-attr` (inline event handlers).
  - `'strict-dynamic'`: Tells Level 3 browsers that any script dynamically created and executed by an already-trusted script (possessing a valid nonce or hash) is automatically trusted, removing the need to explicitly whitelist every third-party domain or library.
  - `object-src 'none'`: Completely blocks plugins and executable components like Flash, Java applets, `<object>`, and `<embed>` elements to eliminate legacy payload execution vectors.
  - `base-uri 'none'`: Restricts URLs that can be defined in HTML `<base>` elements, preventing attackers from injecting `<base>` tags to redirect relative script or stylesheet paths to external servers.
  - `default-src`: Acts as a fallback for all fetch directives when an explicit directive is omitted.
  - `connect-src`: Restricts origins that can be targeted by `fetch()`, `XMLHttpRequest`, WebSockets (`ws://`/`wss://`), EventSource, and Beacon calls.
  - `img-src` / `style-src` / `font-src` / `media-src`: Restrict execution/loading origins for images, CSS stylesheets (`style-src-elem` / `style-src-attr`), web fonts, and audio/video files.
  - `frame-ancestors`: Specifies which external origins are permitted to embed the page inside a `<frame>`, `<iframe>`, `<embed>`, or `<object>` tag (obsoleting `X-Frame-Options`).
  - `form-action`: Limits the target origins to which HTML forms (`<form action="...">`) can submit data.
  - `upgrade-insecure-requests`: Instructs the browser to automatically rewrite and upgrade all HTTP subresource requests to HTTPS before fetching.
  - `report-to` / `report-uri`: Directs CSP violation reports to a logging endpoint (`report-to` uses the modern CSP3 Reporting API with the `Reporting-Endpoints` header; `report-uri` is deprecated but retained for backward compatibility).
- **Most Common CSP Mistakes**:
  - **Relying on domain allowlists**: Defining domain allowlists (`script-src https://example.com https://cdn.com`) is easily bypassed via JSONP endpoints, open redirects, or user-supplied uploads on those host domains.
  - **Automated regex nonce injection**: Replacing script tags using regex middleware automatically injects nonces into attacker-injected `<script>` tags as well; nonces must be added via templating engines during rendering.
  - **Relying solely on CSP for XSS defense**: Treating CSP as a primary control rather than a second layer of defense in depth alongside contextual output encoding and framework auto-escaping.
  - **Overusing **`'unsafe-inline'`** or **`'unsafe-eval'`: Re-enabling string evaluation or un-nonced inline scripts completely breaks CSP XSS mitigation.
  - **Delivering policies via **`<meta>`** tags for header-only features**: Placing CSP in a `<meta http-equiv="Content-Security-Policy">` tag cannot enforce `frame-ancestors`, `sandbox`, or violation reporting (`report-to`).
  - **Using obsolete header names**: Deploying `X-Content-Security-Policy` or `X-WebKit-CSP`, which are buggy, inconsistent, and deprecated.
  - **Caching HTML responses containing nonces**: Reusing nonces across multiple user requests through cached HTML breaks both nonce uniqueness and security.

### Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet
- **CSRF Defenses & Applicability**:
  - **Synchronizer Token Pattern**:
    - *Mechanism*: Generates a cryptographically random, secret, unpredictable token on the server unique per user session or request. The token is included in the HTML response payload (e.g., hidden form field or meta tag) and submitted back in state-changing requests via POST parameters or custom HTTP headers.
    - *Applicability*: Recommended for stateful web applications that maintain server-side user sessions. Per-request tokens reduce attack windows but can break browser "Back" button navigation; per-session tokens balance usability and security.
  - **Double-Submit Cookie Pattern (Signed / HMAC Variant - RECOMMENDED)**:
    - *Mechanism*: Generates a CSRF token using an HMAC hash over a session-bound value (e.g., server session ID or a random UUID in a JWT) and a CSPRNG salt (`csrfToken = hmacHex + "." + randomHex`). The token is set in a cookie (`csrf_token`) and submitted back explicitly via a custom header or form field. The server splits the token, re-computes the HMAC using the current session ID, and compares the values using constant-time comparison.
    - *Applicability*: Recommended for stateless web applications and APIs where maintaining server-side token state is undesirable.
    - *Naive Variant (DISCOURAGED)*: Unsigned random value matching between a cookie and request parameter. Vulnerable to cookie injection attacks from subdomains or HTTP MITM.
  - **SameSite Cookie Attribute**:
    - *Values*: `Strict` (blocks cookie on all cross-site requests), `Lax` (allows cookie on top-level GET navigations, blocks on cross-site POST/PUT/DELETE), or `None` (allows all cross-site requests, requires `Secure`).
    - *Applicability*: Mandatory defense-in-depth attribute for all session cookies. Rarely sufficient on its own unless strict conditions are met (no state-changing GET endpoints, complete domain ownership, `SameSite=Strict` or `Lax` with `__Host-` prefixes).
  - **Fetch Metadata Request Headers**:
    - *Mechanism*: Inspects browser-supplied `Sec-Fetch-Site` (`same-origin`, `same-site`, `cross-site`, `none`). The server rejects state-changing methods (`POST`, `PUT`, `DELETE`) when `Sec-Fetch-Site: cross-site`. Allows top-level navigation (`Sec-Fetch-Mode: navigate`, `GET`).
    - *Applicability*: Modern web applications targeting current browsers, backed by standard Origin/Referer verification as a mandatory fallback for legacy clients.
  - **Custom Request Headers for AJAX/APIs**:
    - *Mechanism*: Appends custom headers (`X-CSRF-Token`, `X-XSRF-Token`, `CSRF-Token`) to asynchronous requests.
    - *Applicability*: Single-Page Applications (SPAs) and API endpoints that submit data via JavaScript rather than HTML `<form>` tags. Cross-origin browser requests trigger mandatory CORS preflight (`OPTIONS`), preventing unauthorized forgery.
- **Most Common CSRF Mistakes**:
  - **Allowing state changes via GET requests**: Permitting state mutations over `GET` bypasses `SameSite=Lax` and standard link-based CSRF protections.
  - **Using the Naive Double-Submit Cookie pattern**: Verifying raw, un-signed cookie values against request parameters allows attackers on subdomains or non-secure origins to overwrite cookies and forge valid requests.
  - **Validating CSRF tokens against cookie values instead of request payloads**: Comparing cookies to cookies or relying on auto-sent cookies without explicit header/payload checks defeats CSRF protection.
  - **Comparing HMAC signatures with non-constant-time functions**: Using standard `==` equality instead of constant-time string comparison (`crypto.timingSafeEqual` / `hmac.compare_digest`) introduces timing attacks.
  - **Transmitting tokens in URLs or GET parameters**: Leaks CSRF tokens in browser history, proxy access logs, and HTTP `Referer` headers.
  - **Failing to protect against Client-Side CSRF**: Allowing untrusted input (e.g., URL hash fragments) to select fetch endpoints or parameters in client-side JavaScript, which circumvents `SameSite` and synchronizer tokens.
  - **Exempting routes without alternative verification**: Disabling CSRF middleware on webhook or API endpoints without implementing signature verification or origin checks.

### HTTP Security Response Headers Cheat Sheet
- **Full List of Recommended Security Response Headers & Exact Values**:
  - `Strict-Transport-Security`** (HSTS)**:
    - *Recommended Value*: `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
    - *Purpose*: Enforces HTTPS connections exclusively for 2 years (63,072,000 seconds) and forces clients to reject invalid TLS certificates.
  - `X-Content-Type-Options`:
    - *Recommended Value*: `X-Content-Type-Options: nosniff`
    - *Purpose*: Disables MIME-type sniffing, forcing browsers to respect the declared `Content-Type` header and preventing MIME confusion attacks.
  - `X-Frame-Options`:
    - *Recommended Value*: `X-Frame-Options: DENY` (or `SAMEORIGIN` if framing from the same origin is strictly required)
    - *Purpose*: Legacy clickjacking defense preventing the page from rendering in `<frame>`, `<iframe>`, `<embed>`, or `<object>` tags. Superseded by CSP `frame-ancestors 'none'`.
  - `Referrer-Policy`:
    - *Recommended Value*: `Referrer-Policy: strict-origin-when-cross-origin` (or `no-referrer` for sensitive APIs)
    - *Purpose*: Strips path and query string data on cross-origin requests and suppresses the `Referer` header entirely on HTTPS-to-HTTP downgrades.
  - `Content-Type`:
    - *Recommended Value*: `Content-Type: text/html; charset=UTF-8` (or `application/json` for APIs)
    - *Purpose*: Declares explicit media type and character set (`charset=UTF-8`), preventing XSS execution in HTML documents.
  - `Cache-Control`:
    - *Recommended Value*: `Cache-Control: no-store` for sensitive responses
    - *Purpose*: Prohibits shared and private HTTP caches from storing sensitive response data.
  - `Permissions-Policy`** (formerly **`Feature-Policy`):
    - *Recommended Value*: `Permissions-Policy: geolocation=(), camera=(), microphone=()` (or explicit empty allowlists for unused features)
    - *Purpose*: Explicitly disables or restricts access to sensitive browser hardware and software APIs.
  - `Cross-Origin-Opener-Policy`** (COOP)**:
    - *Recommended Value*: `Cross-Origin-Opener-Policy: same-origin`
    - *Purpose*: Isolates the top-level browsing context exclusively to same-origin documents, preventing cross-origin window references and Spectre side-channel leaks.
  - `Cross-Origin-Embedder-Policy`** (COEP)**:
    - *Recommended Value*: `Cross-Origin-Embedder-Policy: require-corp`
    - *Purpose*: Blocks cross-origin resources from loading unless they explicitly grant permission via CORP or CORS.
  - `Cross-Origin-Resource-Policy`** (CORP)**:
    - *Recommended Value*: `Cross-Origin-Resource-Policy: same-site` (or `same-origin`)
    - *Purpose*: Restricts resource inclusion across origins, protecting responses from speculative side-channel attacks.
  - `Clear-Site-Data` (for Sign-Out Endpoints):
    - *Recommended Value*: `Clear-Site-Data: "cache", "cookies", "storage"`
    - *Purpose*: Forces browsers to purge local caches, cookies, and web storage upon user logout.
  - **Secure File Download Headers**:
    - *Recommended Values*: `Content-Disposition: attachment; filename="file.ext"`, `Content-Type: application/octet-stream`, `X-Content-Type-Options: nosniff`.
- **Headers That Should Be Removed or Disabled**:
  - `X-Powered-By`: Remove entirely (exposes backend technology stack e.g., Express, PHP, ASP.NET).
  - `Server`: Remove entirely or set to a non-informative generic string (e.g., `Server: webserver`).
  - `X-AspNet-Version` / `X-AspNetMvc-Version`: Disable in framework configurations (`enableVersionHeader="false"`, `DisableMvcResponseHeader = true`).
  - `X-XSS-Protection`: Set explicitly to `X-XSS-Protection: 0` (or remove). Obsolete XSS auditor that introduced security flaws in older browsers.
  - `Expect-CT`: Remove entirely. Certificate Transparency is mandatory in modern browsers; header is deprecated.
  - `Public-Key-Pins`** (HPKP)**: Remove entirely. Obsolete and dangerous header unsupported by modern browsers; use CAA records and Certificate Transparency instead.
- **Most Common Mistakes for HTTP Headers**:
  - **Enabling **`X-XSS-Protection: 1; mode=block`: Can create new XSS vulnerabilities in safe pages by incorrectly disabling legitimate inline scripts.
  - **Deploying HSTS without pre-validation**: Setting HSTS with a long `max-age` before validating certificate setup can render the application inaccessible if certificates expire or break.
  - **Omitting **`charset=UTF-8`** from **`Content-Type: text/html`: Leaves HTML responses vulnerable to UTF-7 or non-standard encoding XSS bypasses.
  - **Confusing **`no-cache`** with **`no-store`: `no-cache` allows responses to be cached and forces revalidation; sensitive responses require `no-store` to prevent cache storage entirely.
  - **Expecting **`Cache-Control: no-store`** to clear browser Cache API**: `Cache-Control` governs HTTP caching, not client-side application storage or Cache API entries (requires explicit script purging or `Clear-Site-Data`).
  - **Configuring duplicate response headers in web servers**: Failing to unset headers before setting them across separate contexts in web servers (e.g., Apache `mod_headers`) results in duplicated response headers.
