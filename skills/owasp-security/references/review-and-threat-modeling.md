# Secure code review and threat modeling

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Secure Code Review](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)
- [Threat Modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)

## Rules

### Secure Code Review Cheat Sheet
- **How to Prepare**:
  - **For All Reviews**: Understand application architecture and business requirements, gather threat models and previous security findings, identify critical assets and high-risk functions, and review security requirements and documentation.
  - **For Baseline Reviews (Entire Codebase)**: Map complete application boundaries and dependencies (attack surface analysis), analyze the overall security architecture, review security incident history, and audit all third-party libraries.
  - **For Diff-Based Reviews (Pull Requests / Code Changes)**: Identify modified files and affected components, assess the impact on existing security controls, understand the purpose of changes, and prioritize high-risk modifications.
- **What to Prioritize**:
  - **High-Risk Code Patterns**: Prioritize input processing and validation functions, database query construction and ORM usage, file operations and path handling, authentication and session management logic, authorization and access control checks, cryptographic operations and key management, error handling/logging mechanisms, and configuration loading.
  - **Areas Requiring Human Expertise**: Focus manual review efforts on areas where human context provides the highest value: business logic flaws (complex state management and workflows), context-specific vulnerabilities, complex authorization models, race conditions (concurrency issues), cryptographic misuse, and high-level security architecture anti-patterns.
- **Checklist of What to Look for Per Category**:
  - **Input Validation**: Ensure server-side validation is performed on all inputs regardless of client checks; use allowlists over blocklists; apply context-appropriate output encoding (HTML, JS, CSS, URL); enforce content-based file upload validation and size limits; use parameterized queries or prepared statements for SQL queries; enforce input length limits; handle Unicode properly; and avoid disclosing sensitive info in error responses.
  - **Authentication & Session Management**: Verify strong password hashing (e.g., Argon2id), secure session ID entropy and CSPRNG generation, cookie security flags (`Secure`, `HttpOnly`, `SameSite`), session renewal upon authentication, idle/absolute timeouts, and generic error messages during login and password reset.
  - **Authorization**: Validate permissions on every request; enforce least privilege and deny-by-default; conduct object-level authorization checks (preventing IDOR); enforce checks at trust boundaries; and ensure static resources are protected.
  - **Cryptography**: Check for strong encryption algorithms (AES-GCM, ChaCha20-Poly1305), secure key separation and storage, avoiding ECB mode, and using CSPRNGs for random number generation.
  - **Business Logic**: Verify state management and explicit state transitions on the server; check for race conditions and concurrency locks; validate transaction integrity and rollbacks; enforce action caps and resource limits; and ensure authorization checks occur at every workflow step.
  - **Configuration & Security Monitoring**: Ensure secure defaults, disable debugging modes in production, sanitize and redact sensitive data from logs, and log security-relevant events without logging raw untrusted input.
- **How to Trace Data from Source to Sink (Data Flow Analysis)**:
  - **Identify Sources**: User inputs, file uploads, API requests, database reads, and environment variables.
  - **Follow Processing**: Track data through validation, transformation, business logic, and caching layers.
  - **Check Sinks**: Examine where data terminates—database queries, file writes, HTML rendering, log outputs, and external API calls.
  - **Validate Boundaries**: Verify that input validation and context-appropriate output encoding occur at every trust boundary crossing.
  - **Trust Zones**: Ensure security controls are explicitly enforced whenever data moves between different trust zones.
  - **Data Classification**: Verify that sensitive data receives appropriate protection across its entire lifecycle.
- **How to Rate and Report Findings**:
  - **Finding Report Structure**: Include Vulnerability Title, Severity (Critical/High/Medium/Low), CWE Number/Name, Location (File:Line or Function), Detailed Description, Security Impact & Attack Scenarios, Steps to Reproduce / Proof of Concept, Specific Fix Guidance with Code Examples, References, Status, Assignee, and Target Due Date.
  - **Review Summary Structure**: Executive summary recording Application Name, Commit Hash/Version, Reviewer Names, Date, Scope Reviewed, Findings Breakdown by Severity, Key Priority Recommendations, and Overall Risk Assessment.
- **Most Common Mistakes in Code Review**:
  - Relying entirely on automated SAST/DAST tools and skipping manual review for complex business logic, state management, and authorization rules.
  - Performing diff-based pull request reviews in isolation without assessing the impact of changes on existing security controls or overall application architecture.
  - Trusting client-side validation or assuming data coming from internal APIs, microservices, or databases is inherently safe.
  - Failing to trace data flow completely from untrusted source to execution sink.
  - Logging un-sanitized user input verbatim, which leads to log injection vulnerabilities.

### Threat Modeling Cheat Sheet
- **The Four Key Questions (Threat Modeling Manifesto)**:
  - **What are we working on?** (System Modeling & Decomposition)
  - **What can go wrong?** (Threat Identification)
  - **What are we going to do about it?** (Response and Mitigations)
  - **Did we do a good enough job?** (Review and Validation)
- **System Decomposition with DFDs & Trust Boundaries**:
  - Visualizing the target system using Data Flow Diagrams (DFDs) to represent external entities, processes, data stores, and data flows.
  - Explicitly marking **Trust Boundaries** on the DFD (where data moves between different trust levels or network zones) to highlight potential attack surface entry points.
  - Brainstorming sessions can supplement DFDs to increase team engagement, align domain vocabulary, and discover business logic nuances.
  - Maintaining and storing DFD artifacts continuously alongside the codebase as the architecture evolves.
- **STRIDE Threat Categories, Examples & Mitigations**:
  - **Spoofing** (*Violates Authentication*):
    - *Example*: An attacker steals a session token or JWT to impersonate a legitimate user.
    - *Mitigation*: Enforce strong multi-factor authentication, secure cookie flags (`HttpOnly`, `Secure`, `SameSite`), and cryptographic token signature verification.
  - **Tampering** (*Violates Integrity*):
    - *Example*: An attacker alters HTTP request parameters or database records to bypass business logic.
    - *Mitigation*: Enforce server-side input validation, parameterized database queries, HMAC signature verification, and digital signatures.
  - **Repudiation** (*Violates Accounting / Non-repudiation*):
    - *Example*: An attacker manipulates or deletes application logs to conceal unauthorized actions.
    - *Mitigation*: Implement append-only, structured security logging, time synchronization across servers via NTP, and tamper-evident audit trails.
  - **Information Disclosure** (*Violates Confidentiality*):
    - *Example*: An attacker extracts sensitive customer data from a database or reads stack traces in 500 error pages.
    - *Mitigation*: Encrypt data at rest (AES-GCM), enforce TLS 1.3 in transit, and sanitize error responses to mask system internals.
  - **Denial of Service** (*Violates Availability*):
    - *Example*: An attacker locks out users or exhausts server memory by flooding endpoints with requests.
    - *Mitigation*: Implement rate limiting, body size limits, resource quotas, and asynchronous processing worker queues.
  - **Elevation of Privileges** (*Violates Authorization*):
    - *Example*: An attacker tampers with a JWT role claim to elevate their permissions to administrator.
    - *Mitigation*: Perform server-side authorization checks on every request (ABAC/ReBAC), deny access by default, and validate JWT signatures.
- **How to Prioritize Threats**:
  - Evaluate risk as the product of threat likelihood and business impact (Risk = Likelihood × Impact).
  - Factor in the engineering effort required to implement a fix when setting final remediation priorities.
  - Select complementary threat modeling techniques based on project scope: **Attack Trees** for high-value goals, **Misuse/Abuse Cases** for workflow abuse, **LINDDUN** for privacy harms, **PASTA** for business alignment, or **OCTAVE** for organizational asset risk.
- **Threat Response Options**:
  - **Mitigate**: Implement technical or operational controls to reduce the likelihood or impact of the threat.
  - **Eliminate**: Completely remove the vulnerable feature or component causing the threat.
  - **Transfer**: Allocate financial or contractual responsibility to another party (noting that transfer does not reduce the actual likelihood of an attack, and residual risk ownership must be documented).
  - **Accept**: Formally accept the residual risk without technical changes due to business constraints or low impact.
- **How to Validate the Model ("Did we do a good enough job?")**:
  - Stakeholder Review: Ensure the model is reviewed collaboratively by development, security, and business stakeholders.
  - System Accuracy: Verify that DFDs accurately represent the deployed system architecture and trust boundaries.
  - Coverage Check: Confirm that every identified threat has an explicitly documented response strategy.
  - Mitigation Testing: Ensure that proposed mitigations effectively reduce risk to acceptable levels and can be tested through automated unit/integration tests.
  - Documentation: Formally document decisions and keep threat models current as the system evolves.
- **Most Common Mistakes in Threat Modeling**:
  - Treating threat modeling as a one-time exercise during initial design rather than maintaining it throughout the SDLC.
  - Focusing exclusively on infrastructure threats while ignoring business logic flaws, process vulnerabilities, and workflow abuse.
  - Assuming that transferring risk (e.g., via insurance or third-party vendors) removes the requirement for security controls.
  - Failing to verify that agreed-upon mitigations are actually implemented and validated with automated security tests.
  - Overcomplicating models by applying multiple methodologies without a clear business scope or purpose.
