# Authentication, sessions and tokens

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [Multifactor Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- [Forgot Password](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- [Credential Stuffing Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html)
- [JSON Web Token](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html)
- [OAuth2](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html)

## Rules

### Password Storage Cheat Sheet

#### Actionable Rules & Parameters
- **Algorithm Selection & Configurations**: Passwords must **never** be stored in plain text or hashed using fast algorithms (like SHA-256 or MD5). Use slow, memory-hard algorithms with the following exact parameters:
  - **Argon2id (Preferred)**: Minimum configuration of **19 MiB of memory (**`m=19456`), **2 iterations (**`t=2`), and **1 degree of parallelism (**`p=1`). Other balanced equivalent minimums include 46 MiB (`m=47104`, `t=1`), 12 MiB (`m=12288`, `t=3`), or 9 MiB (`m=9216`, `t=4`). Do **not** use the `Argon2i` variant.
  - **scrypt (Secondary)**: Use when Argon2id is unavailable. Minimum CPU/memory cost parameter N=2^17 (128 MiB), block size r=8 (1024 bytes), and parallelization p=1.
  - **bcrypt (Legacy)**: Use only for legacy systems where Argon2id and scrypt are unsupported. Set a **work factor/cost of 10 or higher**. Strict **password limit of 72 bytes** must be enforced due to standard implementation caps.
  - **PBKDF2 (FIPS-140 Compliance)**: Preferred when FIPS-140 validation is strictly required. Use **600,000 or more iterations** with `HMAC-SHA-256` (or 220,000 iterations for `HMAC-SHA-512`). `PBKDF2-HMAC-SHA1` (1,400,000 iterations) is legacy-only.
- **Salting**: Cryptographically random, unique salts must be generated for each password (handled automatically by modern hashing libraries) to prevent precomputed rainbow table attacks.
- **Peppering**: Store an application-wide secret (pepper) in a secure secret manager or HSM outside the database. If pre-hashing passwords before `bcrypt`, compute `bcrypt(base64(hmac-sha384(password, pepper)), salt, cost)` to avoid null-byte issues.

#### Common Mistakes
- Using fast general-purpose cryptographic hash functions (SHA-256, SHA-512, MD5) for password storage.
- Truncating passwords past 72 bytes in `bcrypt` without proper base64 HMAC pre-hashing, leading to null-byte truncation vulnerabilities or "password shucking".
- Hardcoding pepper values directly in database columns alongside password hashes.

### Authentication Cheat Sheet

#### Actionable Rules & Requirements
- **Password Strength & Length Rules**:
  - Enforce a **minimum length of 8 characters** if MFA is enabled, or **at least 15 characters** if MFA is disabled.
  - Set a **maximum password length of at least 64 characters** (up to 128+) to support long passphrases.
  - **Allow all characters**, including Unicode space and special characters, without imposing arbitrary character composition rules (e.g., forcing uppercase, lowercase, numbers, or symbols).
  - Screen passwords against breach blocklists (e.g., Have I Been Pwned API or local wordlists). Do **not** mandate arbitrary periodic password rotations.
- **Generic Error Messages**: Login, password reset, and account registration forms must respond with generic messages (e.g., *"Login failed; Invalid user ID or password"*) to prevent account enumeration.
- **Timing Attack Prevention**: Execute identical execution logic and password hashing routines regardless of whether the username exists, maintaining constant-time responses.
- **Protection & Lockout**: Implement login throttling and account lockout controls. Lockout counters must track by account rather than source IP. Use an exponential lockout duration ( doubling delays, max 2–3 attempts before lockout).
- **Re-authentication**: Require explicit re-authentication (password or MFA) before modifying high-risk account state (e.g., changing payment methods, changing email addresses, or changing security settings).

#### Common Mistakes
- Disclosing account existence through verbose error messages (e.g., *"User does not exist"*).
- Restricting password lengths or forbidding special Unicode characters/spaces.
- Mandating periodic password expiration, which causes users to pick predictable patterns.
- Relying on legacy security questions or knowledge-based authentication.

### Session Management Cheat Sheet

#### Actionable Rules & Parameters
- **Entropy & Session ID Properties**:
  - Session IDs must contain **at least 64 bits of entropy** generated via a Cryptographically Secure Pseudorandom Number Generator (CSPRNG).
  - Encoded length must be **at least 16 hexadecimal characters**. For custom session tokens, generate at least 128 bits of CSPRNG entropy.
  - Session IDs must be opaque, meaningless values containing no PII or user role details. Use generic cookie names (e.g., `id`) to prevent framework fingerprinting.
- **Cookie Security Attributes**:
  - `Secure`: Mandatory to ensure cookies are only transmitted over HTTPS.
  - `HttpOnly`: Mandatory to block access via JavaScript DOM (`document.cookie`), mitigating XSS theft.
  - `SameSite`: Set to `Strict` or `Lax` to prevent Cross-Site Request Forgery (CSRF).
  - **Cookie Name Prefixes**: Use `__Host-` prefix (e.g., `Set-Cookie: __Host-id=...; Secure; HttpOnly; SameSite=Strict; Path=/`) which forces `Secure`, `Path=/`, and forbids a `Domain` attribute to block subdomain injection.
- **Timeouts & Lifecycle**:
  - **Idle Timeout**: Enforce 2–5 minutes for high-value applications, or 15–30 minutes for lower-risk systems.
  - **Absolute Timeout**: Force a hard session limit (4–8 hours for typical workday workflows) regardless of ongoing user activity.
  - **Session Renewal**: Immediately generate a new session ID upon any login or privilege escalation to prevent session fixation attacks.
- **Server-Side Token Storage**: Store SHA-256 hashes of session tokens in the backend database/cache rather than raw, unhashed tokens.

#### Common Mistakes
- Storing session tokens in browser `localStorage` or `sessionStorage` (making them vulnerable to XSS extraction).
- Failing to regenerate session IDs upon authentication (enabling session fixation).
- Passing session IDs inside URL query parameters.
- Relying solely on client-side browser closing to terminate a session rather than enforcing server-side invalidation.

### Multifactor Authentication (MFA) Cheat Sheet

#### Actionable Rules & Parameters
- **Factor Requirements**: MFA requires at least two distinct factor types: *Something you know* (password), *Something you have* (OTP token/FIDO2 key), or *Something you are* (biometrics).
- **Software TOTP (RFC 6238)**:
  - Use the standard **30-second time-step** and standard HMAC-SHA1/SHA256 algorithms.
  - Provide initial seeds via QR code over HTTPS using CSPRNG generation.
- **OTP Handling & Storage**:
  - One-Time Passwords (OTPs) must be single-use, enforce a short TTL, have strict attempt limits, and be invalidated immediately after verification.
  - Hash OTPs before storing them in memory/database to prevent log disclosure.
- **Authenticator Guidance**:
  - **Restricted Authenticators**: SMS and phone calls are designated as *restricted* by NIST SP 800-63B-4 due to SS7 interception, SIM swapping, and porting attacks. Do **not** use SMS for applications handling PII or high financial risk.
  - **Phishing Resistance**: Prefer WebAuthn/FIDO2/Passkeys or hardware U2F tokens.
- **MFA Fatigue / Push Bombing Mitigation**: Require challenge-response number matching (entering a displayed code into the app) rather than simple "Approve" push prompts. Rate-limit push notifications.

#### Common Mistakes
- Relying on SMS as a primary or sole MFA factor for high-risk, healthcare, or financial systems.
- Allowing un-throttled, infinite OTP verification attempts.
- Using simple push notifications without number matching, leaving users vulnerable to MFA fatigue attacks.
- Accepting knowledge-based security questions for MFA recovery.

### Forgot Password Cheat Sheet

#### Actionable Rules & Requirements
- **Enumeration Prevention**:
  - Return a generic message regardless of whether the user exists (e.g., *"If that email address is in our database, we will send you an email to reset your password"*).
  - Ensure uniform processing time between valid and invalid email lookups (use asynchronous workers or constant execution flows).
- **Token Security**:
  - Generate reset tokens using a CSPRNG.
  - Enforce single-use token validity, tie tokens strictly to a single account, set short expiration times, and store tokens hashed in the database.
- **URL Reset Links**:
  - Pass reset tokens via HTTPS query parameters.
  - Set `Referrer-Policy: no-referrer` on the password reset page to avoid leaking tokens via the HTTP `Referer` header to external resources.
  - Do **not** construct reset URLs using the incoming HTTP `Host` header (prevents Host Header Injection attacks); hardcode or allowlist hostnames.
- **Reset Flow Execution**:
  - Require the user to type the new password twice.
  - Do **not** automatically log the user in after password reset; force manual login through the normal endpoint.
  - Automatically invalidate or offer to revoke all active sessions upon password reset.

#### Common Mistakes
- Leaking valid user accounts via differential status responses or timing variations on reset requests.
- Generating reset tokens from predictable data (e.g., `md5(username + timestamp)`).
- Generating reset URLs dynamically using the untrusted `Host` header.
- Automatically authenticating the user immediately after resetting their password.

### Credential Stuffing Prevention Cheat Sheet

#### Actionable Rules & Controls
- **Primary Defense**: Enforce Multi-Factor Authentication (MFA), particularly phishing-resistant options (FIDO2/WebAuthn).
- **Defense-in-Depth Layering**:
  - **Rate Limiting & Throttling**: Enforce granular rate limits per account, per IP, and per device fingerprint.
  - **IP Intelligence & Monitoring**: Monitor and correlate login traffic from hosting providers, residential proxies, VPNs, and known botnets.
  - **JavaScript & Headless Detection**: Require clients to evaluate JavaScript (e.g., generating dynamic client tokens) to block basic HTTP POST automation scripts. Detect automated frameworks like Headless Chrome or PhantomJS.
  - **Breached Credential Screening**: Screen user passwords against known breach datasets (e.g., Pwned Passwords) during account creation and password updates.
  - **Security Notifications**: Send out-of-band notifications to users when logins occur from unrecognized devices, IPs, or locations.

#### Common Mistakes
- Relying solely on simple IP address blocking (easily bypassed using distributed proxy networks like Sentry MBA).
- Using CAPTCHA as the sole primary defense rather than a secondary step-up mechanism.
- Creating multi-step login flows that accidentally expose user enumeration indicators prior to password submission.

### JSON Web Token (JWT) Cheat Sheet

#### Actionable Rules & Validation Steps
- **Algorithm Verification**: Explicitly reject tokens with `"alg": "none"`. Hardcode and enforce an allowlist of accepted algorithms (e.g., `["ES256"]`).
- **Key Type Confusion**: Never allow a public key (used for `RS256` or `ES256`) to be interpreted as a symmetric HMAC secret (`HS256`) during signature verification.
- **Header Parameter Trust**: Do **not** trust key material or key locations provided in token headers (`jwk`, `jku`, `x5u`, `x5c`, `kid`) unless they match pre-established, out-of-band trust anchors or explicit domain allowlists. Sanitize `kid` parameters to prevent path or SQL injection.
- **Mandatory Claim Validation**:
  - `exp` (Expiration): Enforce expiration checks strictly.
  - `nbf` (Not Before): Reject tokens presented before their active time.
  - `iss` (Issuer): Perform exact string/allowlist match against expected issuer URI.
  - `aud` (Audience): Ensure the service's explicit client identifier is present in `aud`.
- **Type Isolation**: Set explicit media typing (`typ` header, e.g., `"at+jwt"` for access tokens) and validate `typ` at endpoints to prevent cross-token type confusion (e.g., using a reset token as an access token).
- **Symmetric Secret Entropy**: HMAC secrets must be generated via CSPRNG with **at least 160 bits of entropy** (ideally ≥256 bits) and match the hash output size (e.g., 256 bits for `HS256`). Never use user passwords as HMAC keys.
- **Confidentiality & JWE**: JWS signatures do **not** provide confidentiality (payloads are merely Base64URL-encoded). For sensitive payload data, use JSON Web Encryption (JWE) or Nested JWTs (`cty: "JWT"`), signing first, then encrypting.

#### Common Mistakes
- Accepting `"alg": "none"` signatures in JWT libraries.
- Passing public keys into verification logic that accepts HMAC algorithms (enabling algorithm confusion).
- Failing to validate `iss` and `aud` claims, allowing token replay across different services.
- Attempting to use JWTs for "stateless" session management without implementing a token revocation mechanism (such as Token Status Lists or a `jti` denylist).

### OAuth2 Cheat Sheet

#### Actionable Rules & Flow Requirements
- **Mandatory Flow**: All client types (including Single Page Apps, mobile apps, and server-side web apps) **must use the Authorization Code Grant with PKCE (**`response_type=code`).
- **PKCE Implementation**:
  - Clients must generate a high-entropy `code_verifier` and transmit the `code_challenge` using the `S256` SHA-256 transformation method.
  - Authorization servers must enforce `code_verifier` validation at the token endpoint and block PKCE downgrade attempts.
- **Deprecated Flow Prohibition**:
  - **Implicit Grant (**`response_type=token`**) is DEPRECATED and removed in OAuth 2.1**; do not use it due to URI fragment token leakage.
  - **Resource Owner Password Credentials Grant (ROPC) is DEPRECATED**; do not collect raw user passwords in OAuth clients.
- **Redirect URI & State Validation**:
  - Perform exact string matching for redirect URIs against pre-registered endpoints. Prohibit open redirectors and enforce HTTPS schemes (HTTP allowed only for localhost loopbacks in native apps).
  - Include a cryptographically random, one-time `state` parameter tied to the user agent session (or OIDC `nonce`) to prevent CSRF.
- **Token Replay Protection & Sender Constraining**:
  - Use Proof-of-Possession (PoP) tokens to bind access tokens to client keys via **DPoP (RFC 9449)** or **mTLS (RFC 8705)**.
  - Enforce **Refresh Token Rotation**: Issue a new refresh token on every exchange and immediately invalidate the old refresh token.
- **Client Authentication**: Prefer asymmetric authentication (`private_key_jwt` or mTLS) over static client secrets.

#### Common Mistakes
- Utilizing deprecated Implicit or Resource Owner Password Credentials grants.
- Implementing PKCE using `code_challenge_method=plain` rather than `S256`.
- Permitting wildcards or loose pattern matching in client redirect URIs.
- Omitting the `state` parameter or failing to bind it to the initiator's session.
