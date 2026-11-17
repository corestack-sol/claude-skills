# APIs, errors and logging

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [REST Security](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
- [GraphQL](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [WebSocket Security](https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html)
- [Webhook Security](https://cheatsheetseries.owasp.org/cheatsheets/Webhook_Security_Cheat_Sheet.html)
- [Error Handling](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
- [Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

## Rules

### REST Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **HTTPS Only**: Secure REST services must exclusively provide HTTPS endpoints. This protects credentials, API keys, and session tokens in transit, validates server identity, and guarantees payload data integrity.
  - **Access Control per Endpoint**: Validate user authentication and authorization on **every endpoint request**. Verify permissions at the resource collection, action, and specific record levels to prevent Broken Object-Level Authorization (BOLA/IDOR) and Broken Function-Level Authorization (BFLA).
  - **JWT Handling**: Verify JWT signatures on every request, reject tokens signed with `"alg": "none"`, and validate `exp`, `nbf`, `iss`, and `aud` claims. Avoid storing JWTs in browser `localStorage`; transmit them via secure authorization headers or `HttpOnly`, `Secure`, `SameSite` cookies.
  - **API Keys**: Require API keys for every request to protected endpoints, rate-limit by API key, and revoke keys upon terms violation. Do **not** rely on API keys as the sole authorization mechanism for sensitive or high-value operations.
  - **Restricting HTTP Methods**: Implement an explicit allowlist of permitted HTTP methods (e.g., `GET`, `POST`, `PUT`). Reject unpermitted methods with `405 Method Not Allowed`. Verify that the authenticated caller is explicitly authorized to execute the specific method on the target resource.
  - **Preventing Out-of-Order API Execution**: Enforce server-side workflow state machine validation to ensure clients cannot skip prerequisite API steps (e.g., executing a checkout/payment endpoint prior to order creation).
  - **Input Validation**: Validate input length, numeric range, format, and type. Use strong types (booleans, numbers, dates) and constrain string inputs with regex allowlists. Set strict payload size limits and reject oversized requests with `413 Payload Too Large`. Ensure XML parsers explicitly disable XXE and entity expansion.
  - **Content-Type & Accept Validation**: Validate incoming `Content-Type` headers and reject unsupported formats with `415 Unsupported Media Type`. Send explicit, safe response `Content-Type` headers (e.g., `application/json; charset=utf-8`) alongside `X-Content-Type-Options: nosniff` to block MIME-type sniffing. Return `406 Not Acceptable` if the server cannot satisfy client `Accept` header preferences.
  - **Correct Response Status Codes**: Use semantically accurate HTTP status codes (e.g., `200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `405 Method Not Allowed`, `406 Not Acceptable`, `413 Payload Too Large`, `415 Unsupported Media Type`, `429 Too Many Requests`, `500 Internal Server Error`, `503 Service Unavailable`).
  - **Error Handling & Audit Logs**: Intercept errors globally to prevent stack traces, debug data, or internal system details from reaching clients. Log security events (validation failures, access denials, rate-limit triggers) with ISO timestamps, user ID, IP address, method, and URI, while strictly excluding credentials, tokens, and raw request bodies.
  - **Security Headers & CORS**: Set `X-Content-Type-Options: nosniff` and `Cache-Control: no-store` for sensitive API responses. Disable CORS headers if cross-domain calls are not required; if required, allowlist explicit origins rather than using wildcards (`*`) with credentials.
  - **Rate Limiting**: Apply rate limiting per IP, user, or API key, responding with `429 Too Many Requests` and a `Retry-After` header when limits are exceeded.
- **Most Common Mistakes**:
  - Passing sensitive API keys or tokens inside URL query strings (e.g., `https://example.com/api?apiKey=123`), which exposes them in server access logs, browser history, and `Referer` headers.
  - Returning `200 OK` HTTP status codes for responses that actually represent application errors.
  - Using wildcard CORS origins (`Access-Control-Allow-Origin: *`) on endpoints that support authenticated requests.
  - Leaking technical stack traces or internal database exceptions in `500 Internal Server Error` responses.

### GraphQL Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **Input Validation**: Validate all incoming data against strict allowlists using specific GraphQL scalar types, custom scalar validators, enums, and input schemas. Use a single character encoding to handle Unicode input safely.
  - **Query Depth Limiting**: Cap maximum query nesting depth (e.g., using `graphql-depth-limit` in JS or `MaxQueryDepthInstrumentation` in Java) to prevent deeply nested recursive queries from causing Denial of Service.
  - **Complexity & Amount Limiting**: Perform Query Cost Analysis to assign weight to fields and reject queries exceeding a maximum total cost threshold. Enforce amount limits on requested array items (e.g., restricting `posts(first: 10)`).
  - **Pagination**: Enforce mandatory pagination on all list-returning fields to constrain the maximum dataset size returned in a single query.
  - **Timeouts & Resource Controls**: Configure request execution timeouts at both the application and infrastructure layers to abort resource-intensive queries.
  - **Batching Attack Defense**: Capping or disabling request batching (submitting multiple queries in a single HTTP POST array) on sensitive mutations (such as login, OTP validation, or password reset) to block GraphQL-specific brute-force attacks. Capping the maximum number of operations batched in a single request.
  - **Introspection & GraphiQL Hardening**: Explicitly disable schema introspection queries and schema exploration tools (like GraphiQL or GraphQL Playground) in all production and publicly accessible environments.
  - **Field-Level Authorization**: Validate caller authorization within query and mutation resolver functions across both **nodes and edges**. Use Interfaces and Unions to return tailored object field subsets based on user roles.
  - **Error Masking & Logging**: Mask internal error details in production responses (e.g., setting `includeStacktraceInErrorResponses: false` in Apollo Server) while logging full stack traces internally for debugging.
  - **Server-Side Batching & Caching**: Implement DataLoader patterns to aggregate and batch backend database/HTTP calls, mitigating N+1 performance bottlenecks and DoS risks.
- **Most Common Mistakes**:
  - Leaving GraphQL schema introspection and GraphiQL IDE tools enabled in production, allowing attackers to discover all types, queries, mutations, and private fields.
  - Omitting query depth and field amount limits, exposing the server to low-bandwidth, highly nested DoS queries.
  - Enforcing authorization checks only on top-level root fields while leaving nested edges or nodes unprotected.
  - Exposing raw database stack traces and debug metadata in GraphQL error responses.
  - Allowing unthrottled query batching on authentication endpoints, allowing rapid credential brute-forcing.

### WebSocket Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **WSS Only**: Always use `wss://` (WebSocket Secure over TLS) in production to protect communication from eavesdropping and tampering. Drop support for deprecated, insecure protocol drafts (like Hixie-76 or hybi-00).
  - **Origin Header Validation**: Validate the `Origin` header during the HTTP upgrade handshake against an explicit allowlist of trusted origins. Never return `true` blindly or use loose wildcard/substring checks.
  - **Cross-Site WebSocket Hijacking (CSWSH) Defenses**: Combine `Origin` validation with CSRF tokens included in the handshake request, and enforce `SameSite=Lax` or `SameSite=Strict` on session cookies.
  - **Authentication & Authorization**:
    - *Connection-Level Auth*: Authenticate connections during the handshake or via the first WSS message before allowing further data exchange. Avoid sending access tokens in URL query parameters (which leak to access logs); send tokens in the initial WebSocket message or use secure cookies.
    - *Message-Level Auth*: Validate user authorization for **every incoming message/action** (e.g., verifying admin roles before processing `action: "delete"`) rather than assuming a connected socket has unlimited access.
  - **Message Validation & Size Limits**: Treat all incoming WebSocket messages as untrusted input. Validate payload structure against JSON schemas and enforce strict payload size limits (`maxPayload`, typically 64 KB or less). For binary payloads, verify file types using magic numbers.
  - **Rate Limiting & DoS Protection**: Implement rate limits per connection/user (e.g., starting at 100 messages/minute). Use ping/pong heartbeat frames to clean up dead connections, apply idle timeouts, cap total connection concurrency per user/IP, and enforce backpressure controls to prevent memory exhaustion.
  - **Token Handling & Session Lifetimes**: Periodically re-validate sessions on long-lived sockets (e.g., every 30 minutes). Rotate tokens over active connections, and **immediately terminate/close all active WebSocket sockets (**`ws.close(1008)`) when a user logs out or their session expires.
  - **Security Monitoring**: Log WebSocket lifecycle events (connection, disconnect, auth failures, rate-limit breaches) with user identity, IP, and origin, while strictly avoiding raw message payload or token logging.
- **Most Common Mistakes**:
  - Using unencrypted `ws://` connections in production environments.
  - Failing to validate the `Origin` header during the HTTP upgrade request, opening the application to Cross-Site WebSocket Hijacking (CSWSH).
  - Transmitting authentication tokens inside URL query strings, leaking credentials into web server logs.
  - Assuming an authenticated WebSocket connection grants blanket permission for all operations, skipping per-message authorization checks.
  - Failing to close active WebSocket sockets when a user logs out or their session is revoked.
  - Logging full WebSocket message payloads containing sensitive personal data or secrets.

### Webhook Security Cheat Sheet
- **Signature Verification**:
  - **Publisher Signing**: Generate a cryptographically random, unique signing secret for each registered webhook subscriber (32 bytes default; 24–64 bytes recommended). Compute a Hash-based Message Authentication Code (HMAC-SHA256) over the delivery timestamp, event ID, and raw request body.
  - **Subscriber Verification**: Calculate the HMAC over the **raw, unparsed request body** prior to any framework parsing or JSON re-serialization (re-ordering whitespace or keys invalidates the signature).
  - **Constant-Time Comparison**: Compare the computed HMAC against the received signature using a constant-time comparison function (e.g., `crypto.timingSafeEqual`, `hmac.compare_digest`, or `MessageDigest.isEqual`) to avoid timing attacks and signing oracles.
  - **Secret Rotation**: Support an overlap window during planned secret rotation where the publisher signs with both old and new secrets; subscribers must accept the delivery if any active secret signature matches. Return a generic HTTP `401 Unauthorized` status code if verification fails.
- **Replay Protection & Idempotency**:
  - **Authenticated Timestamps**: The publisher must include a Unix timestamp in the signed payload and generate a fresh timestamp/signature for every retry attempt.
  - **Tolerance Window**: Subscribers must reject deliveries whose authenticated timestamp drifts beyond a strict tolerance window (typically 5 minutes), maintaining server time via Network Time Protocol (NTP).
  - **Replay Caching & Deduplication**: Cache authenticated event IDs for at least twice the tolerance window (e.g., 10 minutes). Persist processed event IDs across the entire publisher retry window (e.g., 3 days) and immediately return `200 OK` for duplicate deliveries without repeating side effects.
  - **Asynchronous Execution**: Acknowledge deliveries quickly (within 10 seconds) and push payloads to an asynchronous background worker queue to absorb traffic spikes.
- **SSRF Concerns (Publisher Side)**:
  - **Scheme & IP Allowlisting**: Enforce `https://` schemes only. Resolve destination hostnames and block all non-public/internal IP ranges (loopback `127.0.0.0/8`, RFC 1918 private subnets, link-local `169.254.169.254`, and internal hostnames like `metadata.google.internal`).
  - **DNS Rebinding Prevention**: Pin connections by performing DNS resolution once, validating the IP, and connecting directly to that validated IP while passing the original hostname in the HTTP `Host` header and TLS Server Name Indication (SNI).
  - **Redirect & Egress Restrictions**: Disable automatic HTTP redirects or re-apply full SSRF IP validation on every redirect destination. Isolate webhook delivery workers inside dedicated egress proxy environments (e.g., Smokescreen).
- **Most Common Mistakes**:
  - Verifying HMAC signatures against re-serialized or formatted JSON objects rather than raw request bytes.
  - Performing signature string comparisons using standard equality operators (`==` or `===`), exposing endpoints to timing attacks.
  - Relying solely on payload signatures without validating delivery timestamps or caching event IDs, leaving endpoints vulnerable to replay attacks.
  - Resolving domain names separately from the HTTP connection socket, allowing DNS rebinding to target internal resources.
  - Logging full request payloads, signing secrets, or authorization headers.

### Error Handling Cheat Sheet
- **Safe Error Responses**:
  - **Global Exception Catching**: Intercept all unhandled exceptions globally at the framework runtime or deployment descriptor level (e.g., `@RestControllerAdvice` in Spring, ASP.NET Exception Middleware, or Servlet `web.xml`).
  - **Generic Client Payload**: Never return internal technical details to the client. Return a standardized generic message (e.g., `"An error occurred, please retry"`) formatted as JSON using RFC 7807 / RFC 9457 Problem Details (`application/problem+json`).
  - **Semantic HTTP Status Codes**: Differentiate client vs. server errors cleanly. Use 4xx codes for client-side issues (e.g., `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `413 Payload Too Large`, `429 Too Many Requests`) and 5xx codes strictly for unhandled server bugs.
  - **Internal Error Tracing**: Log complete error details—including full exception stack traces, parameter context, and runtime types—internally to secure server logs or a SIEM while stripping them entirely from client responses.
- **Most Common Mistakes**:
  - Leaking database error messages, SQL queries, system file paths, stack traces, or framework version numbers in HTTP 500 error pages.
  - Disclosing whether user accounts exist through differential error messages or execution timing during login, password reset, or registration flows.
  - Catching exceptions silently without logging them server-side, causing critical system failures or attacks to go unnoticed.
  - Exposing technical details in custom HTTP response headers.

### Logging Cheat Sheet
- **Which Security Events Must Be Logged**:
  - **Authentication & Session Lifecycle**: Successes and failures for logins, password resets, MFA prompts, session creation, renewal, destruction, privilege changes, and suspicious JWT validation failures.
  - **Validation & Access Control**: Input/output validation failures, protocol violations, parameter tampering, and authorization/access control denials.
  - **High-Risk Operations**: User administration actions, use of administrative/break-glass privileges, access to sensitive data, cryptographic key rotation, data imports/exports, file uploads, deserialization failures, and configuration changes.
  - **System & Application Events**: Runtime exceptions, application startups/shutdowns, connectivity drops, backend TLS validation failures, and out-of-order business logic attempts.
  - **Agent & Automation Systems**: Tool calls, parameter inputs, execution outcomes, and decisions in AI/MCP agent frameworks.
  - **Mandatory Event Attributes**: Every entry must record **When** (international ISO timestamp), **Where** (application ID, server IP/port, entry URL), **Who** (authenticated user ID, client IP), and **What** (event type, severity level, description).
- **What Must NEVER Be Logged**:
  - Plaintext credentials, passwords, and PINs.
  - Raw session IDs (log a salted hash of the session ID instead for session correlation).
  - Session/access tokens, API keys, signing secrets, encryption keys, and primary credentials.
  - Personally Identifiable Information (PII) and sensitive personal data (credit card / cardholder data, bank details, health data, government IDs).
  - Full request/response body payloads containing unmasked user data or secrets.
  - Application source code and database connection strings.
- **Log Injection Prevention**:
  - **Control Character Sanitization**: Sanitize all untrusted log inputs to remove or encode Carriage Return (`\r` / `CR`), Line Feed (`\n` / `LF`), and log delimiter characters, preventing attackers from forging log entries.
  - **Structured Logging**: Output log entries in a structured format (such as JSON or Syslog over TLS) so user input cannot alter log schema boundaries.
  - **Boundary Validation**: Validate data from untrusted zones before logging; if a field is malformed, safely sanitize or omit that field while retaining the overall event context.
- **Log Protection**:
  - **Storage Isolation**: Store log files on a dedicated OS disk partition or in a restricted database account with strict write-only/append-only permissions. Logs must **never** be stored in public web-accessible directories.
  - **Encrypted Transport**: Stream log data to centralized SIEM or log management services using secure protocols (e.g., Syslog over TLS).
  - **Tamper Detection & Access Control**: Employ append-only/WORM storage, cryptographically signed audit entries, or database append-only controls. Restrict read access to logs and record all log queries.
  - **Resource Limits**: Implement strict disk space quotas and log rotation to prevent log flooding attacks from causing application denial of service.
- **Most Common Mistakes**:
  - Logging sensitive user passwords, bearer tokens, or raw session tokens in plain text.
  - Writing un-sanitized user input directly into flat text log files, enabling Log Injection / Log Forging.
  - Exposing log files in public web directories.
  - Failing to set up disk space quotas or log rotation, allowing an attacker to fill the disk and crash the application.
  - Operating without server clock synchronization (NTP), which destroys the ability to perform cross-server incident timeline analysis.
