# Cryptography, secrets and TLS

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Cryptographic Storage](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [Transport Layer Security](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)

## Rules

### Cryptographic Storage Cheat Sheet
- **Symmetric vs. Asymmetric Algorithms**:
  - **Symmetric Encryption**: Use **AES** with a key size of at least **128 bits** (ideally **256 bits**). Alternatively, use **ChaCha20-Poly1305** where supported.
  - **Asymmetric Encryption**: Use an established hybrid encryption scheme via a maintained library, such as **Hybrid Public Key Encryption (HPKE)** (RFC 9180), which combines key agreement, key derivation, and authenticated symmetric encryption. Note that **X25519 on Curve25519** is a key-agreement primitive and does not encrypt stored data by itself.
  - **RSA Usage & Padding**: If RSA must be used, enforce a key size of **at least 2048 bits**. Always use **RSAES-OAEP** (Optimal Asymmetric Encryption Padding) as specified in RFC 8017 Section 7.1. **Never** use raw RSA or custom prepended random bytes instead of OAEP.
  - **Post-Quantum Cryptography (PQC)**: RSA and ECC are not post-quantum secure. Where long-term confidentiality is required, migrate to standardized post-quantum mechanisms such as **ML-KEM (NIST FIPS 203)** in a hybrid construction alongside classical algorithms.
- **Cipher Modes (Preferred vs. To Avoid)**:
  - **Preferred (Authenticated Modes)**: Always preference Authenticated Encryption with Associated Data (AEAD) modes such as **GCM** (Galois/Counter Mode) or **CCM**, which guarantee integrity, authenticity, and confidentiality simultaneously.
  - **Secondary (Non-Authenticated Modes)**: Use **CTR** or **CBC** mode only if GCM/CCM are unavailable. When using CTR or CBC, you **must** implement a separate authentication layer using the **Encrypt-then-MAC (EtM)** pattern.
  - **Modes to Avoid**: **Never use ECB (Electronic Codebook) mode** for multi-block encryption. Avoid unauthenticated CBC mode due to padding oracle risks. Never write custom cryptographic algorithms.
- **Secure Random Number Generation (CSPRNG per Language)**:
  - Security-sensitive values (keys, IVs, nonces, tokens) **must** be generated using Cryptographically Secure Pseudo-Random Number Generators (CSPRNG):
    - **C**: `getrandom(2)` (Unsafe: `rand()`, `random()`).
    - **Java**: `java.security.SecureRandom`, `java.util.UUID.randomUUID()` (Unsafe: `Math.random()`, `java.util.Random`, `SplittableRandom`, `ThreadLocalRandom`).
    - **PHP**: `Random\Engine\Secure` (PHP 8.2+), `random_bytes()`, `random_int()` (PHP 7+), `openssl_random_pseudo_bytes()` (PHP 5) (Unsafe: `array_rand()`, `lcg_value()`, `mt_rand()`, `rand()`, `uniqid()`).
    - **.NET / C#**: `RandomNumberGenerator` (Unsafe: `Random()`).
    - **Python**: `secrets` module (Unsafe: `random()`).
    - **Node.js**: `crypto.randomBytes()`, `crypto.randomInt()`, `crypto.randomUUID()` (Unsafe: `Math.random()`).
    - **Go**: `crypto/rand` (Unsafe: `math/rand`).
    - **Rust**: `rand::prng::chacha::ChaChaRng` and standard Rust CSPRNGs (Unsafe: `rand::prng::XorShiftRng`).
    - **Ruby**: `SecureRandom` (Unsafe: `rand()`, `Random`).
    - **Objective-C**: `SecRandomCopyBytes`, `arc4random` family (Unsafe: `rand()`, `random()`, `GKRandomSource`).
- **Key Generation, Storage, Separation & Lifetime**:
  - **Key Generation**: Generate symmetric keys using a CSPRNG or derive them using strong KDFs (such as PBKDF2 or Argon2id) from user passphrases.
  - **Separation of Keys & Data**: Store encryption keys logically or physically separate from the encrypted data. In basic environments where separation is difficult, separate Key Encryption Keys (KEK) from Data Encryption Keys (DEK).
  - **Key Lifetimes & Rotation**: Rotate keys upon suspected compromise, employee offboarding, cryptoperiod expiration (NIST SP 800-57), or before hitting cipher mode usage limits (such as AES-GCM IV uniqueness and message limits under NIST SP 800-38D).
- **Envelope Encryption**:
  - Protect data at rest using a **Data Encryption Key (DEK)** to encrypt the payload, and wrap/encrypt the DEK using a **Key Encryption Key (KEK)** managed by a KMS, HSM, or derived via KDF.
- **Most Common Mistakes**:
  - Designing custom cryptographic algorithms or home-grown padding.
  - Using **ECB mode** or unauthenticated CBC mode without Encrypt-then-MAC.
  - Reusing Initialization Vectors (IVs) or nonces under the same key in AES-GCM, destroying integrity and confidentiality.
  - Storing plaintext encryption keys alongside the encrypted database records.
  - Relying on non-cryptographic PRNGs (`Math.random()`, `rand()`) for security tokens or keys.

### Secrets Management Cheat Sheet
- **Where Secrets May and May Not Live**:
  - **Where Secrets MUST NOT Live**:
    - **Source Code Repositories**: Never hardcode API keys, credentials, or tokens in source code files or Git commit history.
    - **Container Definitions & Images**: Do not bake secrets into Dockerfiles using `ENV` or `ARG` instructions, or embed them in container image layers.
    - **Environment / Command Logs**: Never print secrets to `stdout`/`stderr`, console logs, or shell history files (e.g., `~/.bash_history`).
    - **Shared External Storage**: Never expose secrets via shared storage volumes or unencrypted configuration files.
  - **Where Secrets MAY/SHOULD Live**:
    - **Centralized Secrets Managers**: Store secrets in dedicated management systems such as HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, Google Secret Manager, or CyberArk Conjur.
    - **In-Memory / Mounted Volumes**: Inject secrets at runtime via shared in-memory volumes (`tmpfs`) or sidecar containers (e.g., Vault Agent / Secrets Store CSI driver).
    - **Dynamic Ephemeral Leases**: Prefer dynamically generated, short-lived secrets created on demand with strict lease lifetimes.
- **Secret Rotation & Automation**:
  - **Automated Pipelines**: Automate key and credential rotation through secrets management pipelines or serverless functions.
  - **AWS Lambda Rotation Pattern**: Validate current credentials before creating pending credentials (`AWSPENDING`), test the pending version against the target resource, and promote it to `AWSCURRENT`.
  - **Dynamic Secrets**: Generate application database credentials dynamically on pod startup with short leases. Note that stopping an application container does **not** revoke dynamic secrets; they remain active until explicitly revoked or expired by the secret store.
  - **Break-Glass Procedures**: Maintain encrypted, offline/cold backup credentials in a secondary system for emergency disaster recovery.
- **Detection & Incident Response for Leaked Secrets**:
  - **Secret Scanning**: Implement automated scanning tools (e.g., `gitleaks`, `git-secrets`) in IDEs, pre-commit hooks, and CI/CD pull request checks.
  - **Incident Response Steps**:
    - **Revocation**: Immediately revoke the exposed credentials at the provider.
    - **Rotation**: Issue new credentials via automated pipelines.
    - **Deletion & Scrubbing**: Remove exposed secrets from code, squash/rewrite Git commit history (aware of team impact), or sanitize log entries.
    - **Post-Incident Investigation**: Review audit logs to determine who accessed the secret, when it was used, and what resources were exposed.
- **Most Common Mistakes**:
  - Hardcoding secrets inside source code, configuration files, or Docker build arguments.
  - Printing secrets in plaintext to application logs or CI/CD console output.
  - Using a single "master secret" across all environments (Development, Staging, Production).
  - Relying on manual rotation, which leads to forgotten credentials and human error.
  - Over-granting access permissions to secrets managers instead of enforcing strict least privilege.

### Transport Layer Security (TLS) Cheat Sheet
- **TLS Protocol Versions**:
  - **Default Protocol**: Web applications **must default to TLS 1.3** and may support **TLS 1.2** for compatibility.
  - **Forbidden Protocols**: **TLS 1.0 and TLS 1.1 are formally deprecated (RFC 8996) and MUST be disabled**. SSLv2 and SSLv3 must always be disabled.
  - **Downgrade Defense**: Enable the `TLS_FALLBACK_SCSV` extension to prevent protocol downgrade attacks.
- **Cipher Suite Guidance**:
  - **TLS 1.3**: Use standard AEAD cipher suites (`TLS_AES_256_GCM_SHA384`, `TLS_CHACHA20_POLY1305_SHA256`, `TLS_AES_128_GCM_SHA256`).
  - **TLS 1.2**: Prefer **ECDHE cipher suites with AEAD** (e.g., `ECDHE-ECDSA-AES128-GCM-SHA256`) to ensure forward secrecy.
  - **Prohibited Ciphers**: Disable Null ciphers, Anonymous ciphers (`TLS_*_anon_*`), EXPORT ciphers (`TLS_*_EXPORT_*`), static RSA key transport (`TLS_RSA_*`), static Diffie-Hellman (`TLS_DH_*`), and CBC-mode ciphers.
  - **Compression**: Explicitly **disable TLS compression** to mitigate CRIME side-channel attacks.
- **Certificate Requirements**:
  - **Key Strength & Algorithms**: Use RSA keys of **2048 bits or greater**, or ECDSA (P-256 / Curve25519) keys.
  - **Signature Hashing**: Enforce **SHA-256** for certificate signature algorithms; block MD5 and SHA-1.
  - **Domain Name Matching**: Ensure the primary Fully Qualified Domain Name (FQDN) is set in the `commonName` (CN) and all FQDNs are included in the `subjectAlternativeName`** (SAN)** extension (modern browsers like Chrome ignore `CN` and require `SAN`).
  - **Certification Authorities & CAA**: Use trusted CAs and publish DNS **Certification Authority Authorization (CAA) records** to restrict which CAs are permitted to issue certificates for your domain.
  - **Wildcard Certificates**: Exercise extreme caution with wildcard certificates—compromising a single private key compromises all subdomains.
- **HTTP Strict Transport Security (HSTS)**:
  - **Recommended Header**: `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`.
  - **What it Does**: Forces browsers to interact with the site exclusively over HTTPS for 2 years (63,072,000 seconds), includes all subdomains, and qualifies the domain for browser preload lists.
  - **Precaution**: Ensure all subdomains strictly support valid HTTPS before deploying long `max-age` values to avoid rendering subdomains inaccessible.
- **Most Common Mistakes**:
  - Supporting legacy TLS 1.0, TLS 1.1, or SSL protocols.
  - Including ciphers without Perfect Forward Secrecy (static RSA or static DH) or unauthenticated CBC ciphers.
  - Omitting `subjectAlternativeName` (SAN) entries from TLS certificates.
  - Mixing HTTPS and HTTP content ("mixed content") on web pages.
  - Enabling HSTS with `includeSubDomains` before verifying certificate validity across all internal/external subdomains.
