# Injection, input handling, uploads and SSRF

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
- [SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [NoSQL Security](https://cheatsheetseries.owasp.org/cheatsheets/NoSQL_Security_Cheat_Sheet.html)
- [OS Command Injection Defense](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
- [Deserialization](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
- [Prototype Pollution Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Prototype_Pollution_Prevention_Cheat_Sheet.html)
- [File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [Server Side Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Unvalidated Redirects and Forwards](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)

## Rules

### Input Validation Cheat Sheet
- **Allow-list vs. Deny-list Validation**:
  - **Allow-list Validation (Recommended)**: Explicitly define what the application accepts (allowed character sets, data types, numeric ranges, exact enum choices, and maximum string lengths) and reject everything outside those rules.
  - **Deny-list Validation (Discouraged)**: Avoid trying to identify and block malicious patterns or "clean" inputs. Blocking specific characters (like single quotes) rejects legitimate input (e.g., names with apostrophes) without making database queries safe.
  - **Free-form Text Handling**: For free-form text or comments, allow-lists should permit broad Unicode character ranges while enforcing strict string length limits.
- **Syntactic vs. Semantic Validation**:
  - **Syntactic Validation**: Verifies that input meets structured format, type, and boundary rules (e.g., integer ranges, valid date format strings, string length limits, array item counts, and object field requirements).
  - **Semantic Validation**: Verifies that input values make logical business sense within the context of the operation (e.g., verifying that a booking's end date is chronologically after its start date, or checking field consistency).
- **Enforcement Location & Parsing Order**:
  - **Server-Side Enforcement**: All input validation **must** occur server-side. Client-side validation exists strictly for immediate user interface feedback and can be easily bypassed.
  - **Parse Safely Before Validating**: Enforce strict HTTP request size limits and parser execution bounds (such as maximum JSON nesting depth or XML entity limits) before buffering or parsing payloads.
  - **Decode Before Validating**: Decode request parameters according to protocol specifications first, and validate the canonical representation that will actually be used down the pipeline to prevent inconsistent downstream decoding bypasses.
  - **Trust Boundaries**: Validate data whenever it crosses a trust boundary—including data coming from internal APIs, partner feeds, message queues, and stored database records.
- **Regular Expression (Regex) Safety**:
  - Use regular expressions for simple, structured string fields and enforce full-value matching (e.g., `re.fullmatch` in Python).
  - Cap untrusted input string lengths prior to evaluating regular expressions to prevent Regular Expression Denial of Service (ReDoS).
  - Keep regex patterns linear by avoiding nested quantifiers (e.g., `(a+)+`) or ambiguous alternations.
  - Use non-backtracking engines or match timeouts where supported, and treat match timeouts as validation failures.
- **Most Common Mistakes**:
  - Relying on client-side browser checks as a security boundary.
  - Trusting internal microservices, database records, or message queues without validating input at component boundaries.
  - Attempting to "sanitize" or strip bad characters instead of rejecting malformed data.
  - Running schema validation after the parser has already loaded untrusted data into memory without request size or depth limits.
  - Logging malformed or rejected input verbatim without escaping, introducing log injection vulnerabilities.

### Injection Prevention Cheat Sheet
- **Primary Defenses**:
  - **Rule #1: Proper Input Validation**: Implement positive, allow-list input validation with canonicalization across all user input channels.
  - **Rule #2: Use Safe APIs**: Use structured APIs that avoid calling interpreters entirely or provide parameterized query interfaces.
  - **Rule #3: Contextual Output Escaping**: If a parameterized API is unavailable, contextually escape user data using the target interpreter's specific escape syntax as a fallback.
- **Additional Defenses**:
  - **Least Privilege**: Configure application database connections, service accounts, and processes with the minimum privileges required to execute their specific tasks.
  - **Escaping as Last Resort**: Use escaping only when parameterized interfaces or input allow-listing cannot be implemented (e.g., retrofitting legacy systems), recognizing that character escaping is fragile and database-specific.
- **Most Common Mistakes**:
  - Assuming internal microservices, REST web services, or internal databases are inherently "trusted" and skipping input validation or parameterization.
  - Relying on string escaping or sanitization instead of using parameterized APIs.
  - Calling stored procedures that internally construct dynamic SQL queries via string concatenation.

### SQL Injection Prevention Cheat Sheet
- **Primary Defenses**:
  - **Option 1: Prepared Statements (Parameterized Queries)**: Define SQL structure first and bind user inputs as separate parameters (e.g., `PreparedStatement` in Java, `OleDbCommand.Parameters` in.NET, or named parameters in HQL).
  - **Option 2: Safe Stored Procedures**: Use stored procedures that parameterize inputs natively (`CallableStatement` in Java) and avoid dynamic SQL string building inside the procedure.
  - **Option 3: Allow-List Input Validation**: Map user parameters to strict allow-lists when targeting query components that do not support variable binding (e.g., table names, column names, or sort order).
- **Safe Patterns for Dynamic Table/Column Names and **`ORDER BY`:
  - **Table & Column Names**: Map untrusted parameter inputs to hardcoded, expected string constants (e.g., via a map or switch statement) before appending them to a query.
  - **Sort Order (**`ORDER BY`): Convert input into a boolean flag (e.g., `sortOrder ? "ASC": "DESC"`) or map against an enum/allow-list rather than concatenating raw input strings.
- **ORM & Query Builder Pitfalls**:
  - Concatenating user input into ORM query strings (e.g., `session.createQuery("from Inventory where productID='" + param + "'")`) introduces HQL/ORM Injection; use named parameters instead.
  - Assuming ORMs automatically prevent SQL injection when raw SQL clauses, custom filters, or un-parameterized helper functions are invoked.
- **Additional Defenses**:
  - **Least Privilege**: Restrict database user accounts to minimum necessary privileges; never grant DBA or `db_owner` permissions to application connection accounts.
  - **SQL Views**: Use database views to restrict table column exposure and enforce fine-grained access limits.
- **Most Common Mistakes**:
  - Concatenating user input directly into SQL statement strings.
  - Using character escaping as a primary defense instead of parameterized queries.
  - Executing dynamic SQL using `EXEC`, `exec`, or `sp_execute` inside database stored procedures.
  - Granting administrative permissions (`db_owner` / DBA) to application connection users.

### NoSQL Security Cheat Sheet
- **NoSQL Operator Injection & Prevention in MongoDB**:
  - **Strict Scalar Type Checking**: Validate that user inputs match expected primitive scalar types (e.g., `typeof name === 'string'`) before passing them into driver query objects. This prevents attackers from submitting nested objects that MongoDB interprets as query operators (e.g., `{ name: { $ne: "" } }`).
  - **Avoid **`eval()`** & String Queries**: Never construct queries using string concatenation or `eval()`.
  - **Disable/Restrict Server-Side Scripting**: Disallow client-controlled query operators like `$where`, `$regex`, and `$expr`, and disable server-side JavaScript execution in MongoDB when not strictly required.
  - **Driver / ODM Patterns**: Use high-level ODM/ORM drivers (e.g., PyMongo, Mongoose, Spring Data) and construct query objects with explicit type validation.
  - **Schema Validation**: Define strict schemas to constrain field types and reject unexpected objects.
- **Most Common Mistakes**:
  - Passing raw, un-validated HTTP request objects (like `req.body` or `req.query`) directly into database `find()` calls.
  - Accepting raw JSON query fragments or executing `$where` clauses containing untrusted input.
  - Relying on naive string denylists (such as checking for `"$"` via `JSON.stringify()`) instead of enforcing explicit scalar types.
  - Running NoSQL databases without authentication enabled or exposed to public network interfaces.

### OS Command Injection Defense Cheat Sheet
- **Safe Process Execution Without a Shell & Argument Handling**:
  - **Avoid OS Commands**: Use built-in programming language APIs or libraries instead of calling shell commands (e.g., use `mkdir()` instead of `system("mkdir...")`).
  - **Process APIs without Shell**: Use process execution APIs that accept the command executable and its arguments as discrete array elements (e.g., Java `ProcessBuilder("cmd", "arg1", "arg2")` or Node.js `execFile`/`spawn`) rather than raw shell strings.
  - **Argument Isolation**: Java's `ProcessBuilder` and `Runtime.exec` do not invoke a system shell (`/bin/sh` or `cmd.exe`) by default and do not evaluate shell metacharacters (`&`, `|`, `;`, `$`), preventing command chaining.
  - **PHP Escaping Helper Functions**: Use `escapeshellarg()` (which quotes user input as a single parameter) rather than `escapeshellcmd()` (which allows extra arguments to be passed).
  - **Option Terminating (**`--`): Use `--` to terminate option parsing in commands that support it, preventing argument injection.
- **Layers of Defense**:
  - **Layer 1: Parameterization**: Separate executable commands from data arguments using array-based process execution APIs.
  - **Layer 2: Input Validation**: Validate commands against an allow-list of executable names, and validate arguments using strict allow-list regular expressions (e.g., `^[a-z0-9]{3,10}$`) excluding shell metacharacters (`& |; $ > < \ ! ' " ( )`).
  - **Least Privilege**: Run processes using isolated user accounts with the minimal necessary system privileges.
- **Most Common Mistakes**:
  - Passing a single concatenated command string containing user input into shell execution functions (e.g., `child_process.exec` or PHP `system()`).
  - Using `escapeshellcmd()` in PHP instead of `escapeshellarg()`, allowing attackers to inject additional command-line arguments.
  - Allowing untrusted input to specify the executable command path instead of hardcoding the command and validating its arguments.

### Deserialization Cheat Sheet
- **Safe Deserialization Rules Per Language**:
  - **Java**:
    - Avoid `java.io.ObjectInputStream` on untrusted input.
    - Override `resolveClass()` in a custom `ObjectInputStream` to enforce a strict class allow-list (look-ahead deserialization inspection).
    - Configure JVM serialization filters (`java.io.SerializationFilter`) with class allow-lists and resource limits.
    - Mark sensitive or internal fields as `private transient` to prevent serialization/deserialization control.
    - Safe libraries: `fastjson2` (without autotype), `jackson-databind` (without polymorphism), `Kryo` v5+ (with class registration ON), `XStream` v1.4.17+ (with allowlists). Avoid `XMLDecoder`.
  - **.NET (C#)**:
    - Avoid dangerous serializers like `BinaryFormatter`.
    - Set `TypeNameHandling = TypeNameHandling.None` in Newtonsoft `JSON.NET`.
    - Do not use `JavaScriptSerializer` with a `JavaScriptTypeResolver`.
    - Prefer `DataContractSerializer` or `XmlSerializer` where the payload does not define the target type.
    - Implement a custom `SerializationBinder` allow-list when dynamic types are required.
  - **Python**:
    - **Never** deserialize untrusted data using `pickle` (`pickle.load()` / `pickle.loads()`) or `jsonpickle`.
    - For YAML, use `yaml.safe_load()` instead of `yaml.unsafe_load()` or `yaml.load()`.
  - **PHP**:
    - Avoid `unserialize()` on untrusted input; use standard JSON data formats via `json_decode()` and `json_encode()`.
  - **JavaScript**:
    - Use `JSON.parse()` to deserialize JSON data; **never** use `eval()`.
- **Language-Agnostic Safe Methods**:
  - **Alternative Data Formats**: Switch to pure data-interchange formats (JSON, Protocol Buffers, XML) rather than language-native object serialization formats.
  - **Cryptographic Signatures**: Only deserialize data if it includes a valid, server-verified HMAC signature.
- **Most Common Mistakes**:
  - Using native binary object deserializers (Java `ObjectInputStream`,.NET `BinaryFormatter`, Python `pickle`, PHP `unserialize`) on untrusted data.
  - Enabling polymorphic type resolution in JSON serializers (e.g., `TypeNameHandling.Auto` in.NET or `autotype` in FastJson).
  - Attempting to validate deserialized objects *after* calling `Deserialize()` / `readObject()`—by then, malicious gadget code execution has already taken place.

### Prototype Pollution Prevention Cheat Sheet (and JavaScript/TypeScript Security)
- **Concrete Actionable Rules**:
  - **Use **`Map`** or **`Set`: Use `new Map()` or `new Set()` for dynamic collections and dictionaries instead of plain object literals.
  - **Objects Without Prototypes**: If JavaScript objects must be used as dictionaries, instantiate them using `Object.create(null)` so they do not inherit from `Object.prototype`. If object literals are required as a last resort, use `{__proto__: null}`.
  - **Key Filtering & Sanitization**: Never pass untrusted input to recursive merge, deep clone, or path-setter functions without explicitly rejecting or dropping key segments like `__proto__`, `constructor`, and `prototype`. Drop `__proto__` keys before using `Object.assign()` (which mutates the target's prototypeSetter).
  - **Freezing Built-in Prototypes**: Use `Object.freeze()` on built-in prototypes to make properties immutable and prevent property additions. Note that freezing is shallow, so objects referenced by properties must be frozen separately.
  - `Object.seal()`** Caution**: Do not rely on `Object.seal()` to prevent property modifications, as existing writable property values can still be changed.
  - **Node.js Runtime Defense**: Execute Node.js processes with the `--disable-proto=delete` command-line flag to disable the `__proto__` property entirely as a defense-in-depth measure.
  - **Safe Parsing & Schema Enforcement**: Parse JSON using `JSON.parse()`, never `eval()`. Validate parsed objects against a strict runtime schema before use.
  - **TypeScript Runtime Realities**: Do not rely on TypeScript types or assertions (`as`, `!`) for security, as types are completely erased at runtime and enforce nothing against malformed caller data. Avoid using `any`.
- **Most Common Mistakes**:
  - Passing untrusted JSON or query objects into recursive merge or path assignment functions without filtering `__proto__` or `constructor.prototype` keys.
  - Using plain JavaScript objects as key-value dictionaries, allowing property lookup overrides on the prototype chain.
  - Relying on TypeScript type definitions alone to protect backend logic from malicious client payloads.
  - Assuming `Object.seal()` prevents property value mutation.

### File Upload Cheat Sheet
- **Concrete Actionable Rules**:
  - **Extension Validation**:
    - Enforce a strict allow-list containing only business-critical file extensions (e.g., `.docx` and `.pdf` for documents).
    - Perform extension validation **after** full protocol decoding of the filename.
    - Neutralize bypass patterns: double extensions (`.jpg.php`), null byte truncation (`.php%00.jpg`), case manipulation (`.pHp`), alternative execution extensions (`.phtml`, `.php5`, `.jsp`, `.aspx`), and Windows NTFS Alternate Data Streams (`shell.php::$DATA` or colons `:`) by rejecting colons.
  - **Content-Type Validation**: Validate the incoming HTTP `Content-Type` header against an allow-list, but treat it solely as a preliminary check since client headers can be spoofed.
  - **File Signature (Magic Bytes) Validation**: Inspect raw binary header bytes (signatures) to ensure they match the expected file type format before processing.
  - **File Size & Decompression Limits**: Set strict file size limits at the web server and application layers. Calculate sizes after decompression when extracting archive files (ZIPs) to prevent zip bombs.
  - **Filename Safety & Sanitization**:
    - **Primary Control**: Replace original filenames entirely with server-generated random identifiers (e.g., UUID/GUID).
    - If preserving user filenames: restrict filename length, allowlist safe characters (alphanumeric, hyphens, periods), block leading or sequential periods (`..` traversal), restrict leading hyphens/spaces, and reject colons.
    - Prevent Windows NTFS 8.3 short-name alias collisions by generating filenames server-side.
  - **File Storage Location**:
    - **Best**: Store uploaded files on a separate host/server to segregate duties.
    - **Second Best**: Store files outside the public web root.
    - **Third Best**: If stored inside the web root, enforce write-only permissions and disable script execution overrides (`AllowOverride None` in Apache, locked handler mappings in IIS) to prevent `.htaccess` or `web.config` uploads.
  - **Antivirus, Sandboxing & Content Disarm**:
    - Run uploads through an antivirus engine or sandbox prior to public release.
    - For images, decode and re-encode to an allowed format while stripping EXIF metadata. Run documents through Content Disarm & Reconstruction (CDR) tools.
  - **Permissions & Serving**: Authenticate and authorize users before accepting uploads. Set secure HTTP headers when serving user files (`Content-Disposition: attachment`, `Content-Type: application/octet-stream`, `X-Content-Type-Options: nosniff`).
- **Most Common Mistakes**:
  - Storing uploaded files inside the public web root with script execution enabled.
  - Trusting client-supplied `Content-Type` headers or raw filenames without validation.
  - Using extension block-lists rather than an explicit allow-list.
  - Allowing `.htaccess` or `web.config` files to be uploaded, which rebind file handlers to execute malicious code.
  - Failing to enforce upload file size caps or zip decompression bounds.

### Server Side Request Forgery (SSRF) Prevention Cheat Sheet
- **Concrete Actionable Rules**:
  - **Case 1: Identified & Trusted Applications (Allow-List Approach)**:
    - **Data Validation**: Validate input strings strictly against expected technical/business formats (e.g., regex allow-lists).
    - **IP Allow-Listing**: Ensure input is a valid IPv4/IPv6 address, then verify it against a case-sensitive string allow-list of trusted application IPs.
    - **Domain Validation**: Validate domain syntax without triggering DNS lookups during validation. Cross-check the domain against an allow-list of trusted domain names.
    - **URL Construction**: Do **not** accept complete user-supplied URLs. Accept only validated domain names or IPs, and construct target URLs server-side with fixed schemes, ports, and paths.
    - **Network Segregation**: Enforce network firewall rules to restrict outbound connections from application servers to authorized internal destinations only.
  - **Case 2: Arbitrary External Destinations (Webhooks / External Downloads)**:
    - **Scheme Allow-Listing**: Restrict schemes strictly to `https://` (or `http://` if required).
    - **IP Address Block-Listing (Deny-List)**: Validate the destination IP and block all non-public / internal address ranges:
      - **AWS IMDS**: `169.254.169.254`, `metadata.amazonaws.com`
      - **GCP Metadata**: `metadata.google.internal`, `169.254.169.254`
      - **Azure IMDS**: `169.254.169.254`
      - **Localhost**: `127.0.0.0/8`, `0.0.0.0/8`, `::1/128`
      - **RFC 1918 Private**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
      - **IPv6 Unique Local / Link-Local**: `fc00::/7`, `fe80::/10`
      - **Multicast**: `224.0.0.0/4`, `ff00::/8`
  - **DNS Rebinding Defense & Connection Pinning**:
    - Resolving a hostname and checking its IP in application code does **not** close the DNS rebinding gap because HTTP clients re-resolve hostnames when connecting.
    - **Fix**: Resolve the domain once, validate the IP address against the public IP policy, and **connect directly to that exact validated IP address** while preserving the original hostname in the HTTP `Host` header and TLS Server Name Indication (SNI).
  - **Redirect Handling**: Disable automatic HTTP redirection in web clients, or re-apply full SSRF IP validation checks to every redirect target.
  - **Cloud Metadata Hardening**: Enforce **AWS IMDSv2** (requiring session tokens via `PUT` requests) and disable legacy IMDSv1 across cloud workloads.
  - **Egress Isolation**: Route outbound requests through dedicated egress proxies (e.g., Smokescreen) that enforce destination checks at the network layer.
- **Most Common Mistakes**:
  - Performing a DNS check on a domain and then passing the domain string to an HTTP client, allowing DNS rebinding attacks to swap the IP to `127.0.0.1` or `169.254.169.254` during the actual socket connection.
  - Allowing HTTP clients to automatically follow 302 redirects to internal endpoints.
  - Accepting full URLs directly from users and relying on fragile string or regular expression parsing.
  - Leaving legacy IMDSv1 enabled in AWS cloud environments.

### Unvalidated Redirects and Forwards Cheat Sheet
- **Concrete Actionable Rules**:
  - **Avoid User-Controlled Redirects**: Avoid taking target redirect URLs directly from untrusted user input whenever possible.
  - **Server-Side Token Mapping**: If dynamic redirects are required, have the client supply a short key or token that maps server-side to an explicit list of trusted target URLs.
  - **Local Path Validation**:
    - For relative/local return URLs (e.g., post-login navigation), use framework functions that explicitly reject non-local destinations (e.g., ASP.NET Core `LocalRedirect` or `IsLocalUrl`).
    - Ensure paths begin with a single `/` and reject protocol-relative URLs (`//evil.com`) or backslash tricks (`/\evil.com`).
  - **External URL Validation**:
    - Parse target URLs using a maintained URL parser.
    - Compare the parsed scheme, canonical host, and port against an explicit allow-list.
    - Reject userinfo (`user:pass@host`) and ambiguous URL syntax.
  - **Forward Access Control**: Validate that the user is explicitly authorized to access the destination URL before executing server-side forwards (`request.getRequestDispatcher(fwd).forward(...)`), preventing attackers from forwarding to privileged administrative handlers.
  - **Interstitials**: Display an explicit warning page notifying users they are leaving the application, displaying the target domain, and requiring manual confirmation.
  - **Script Execution Termination**: In languages like PHP, always invoke `exit;` immediately after issuing a `header("Location:...")` redirect to prevent execution of downstream page logic.
- **Most Common Mistakes**:
  - Passing unvalidated query parameters (e.g., `?url=...` or `?returnUrl=...`) directly into response redirect functions.
  - Using naive substring, prefix, or regex checks that can be bypassed using userinfo or subdomains (e.g., `http://example.com.attacker.com` or `http://example.com@attacker.com`).
  - Forgetting to terminate script execution after setting a `Location` header in PHP, allowing unauthenticated users to view protected page outputs.
  - Forwarding requests internally without performing authorization checks on the target endpoint.
