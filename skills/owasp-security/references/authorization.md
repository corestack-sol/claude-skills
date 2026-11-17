# Authorization and access control

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [Insecure Direct Object Reference Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [Mass Assignment](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [Multi Tenant Security](https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html)
- [Business Logic Security](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html)

## Rules

### Authorization Cheat Sheet
- **Concrete Actionable Rules**:
  - **Deny by Default**: Always configure applications to deny access by default when no explicit access control rule matches. Prefer explicit code configurations over implicit framework defaults.
  - **Enforce Least Privilege**: Define clear trust boundaries during design by enumerating user types, resources, and allowed operations. Periodically audit active privileges after deployment to prevent "privilege creep".
  - **Enforcement Location**: Enforce access control checks **server-side**, at the gateway, or within serverless functions. Never rely on client-side logic or UI visibility for authorization.
  - **Per-Request & Per-Object Validation**: Validate permissions on **every request** for the **specific object or functionality** being accessed. Access to one object of a given type does not grant access to all objects of that type.
  - **Static Resource Protection**: Ensure static assets, documents, and files pass through authorization checks before being served.
  - **Safe Exit Handling**: Ensure the application exits safely upon authorization failures, centralizing failure logic and preventing unhandled exceptions from leaving the system in an insecure state.
- **Access Control Models (RBAC vs. ABAC vs. ReBAC)**:
  - **Role-Based Access Control (RBAC)**: Assigns permissions to roles and roles to users. Suffer from "role explosion" in large systems and cannot natively handle object-level or horizontal access decisions.
  - **Attribute-Based Access Control (ABAC)**: Evaluates subject, object, and environmental attributes (e.g., job role, time of day, device type, location) using Boolean policies. Preferred over RBAC for fine-grained authorization, least privilege, and multi-tenant systems.
  - **Relationship-Based Access Control (ReBAC)**: Grants access based on direct relationships between users and objects (e.g., post creator/owner). Ideal for fine-grained object ownership and graph-like permission structures.
- **Testing Authorization**:
  - Implement automated unit and integration tests to catch "low-hanging fruit" vulnerabilities, verifying that default-deny policies, fail-safe exits, and ABAC/ReBAC rules are strictly enforced.
  - Supplement automated tests with manual penetration testing to identify complex logical bypasses.
- **Most Common Mistakes**:
  - Relying on client-side checks or UI hiding to restrict access.
  - Using static RBAC for fine-grained object-level access, resulting in role explosion or horizontal authorization bypasses.
  - Relying on third-party framework defaults without verifying their exact behavior.
  - Exposing internal lookup IDs without performing per-object ownership checks on every request.

### Insecure Direct Object Reference (IDOR) Prevention Cheat Sheet
- **Concrete Actionable Rules**:
  - **Object-Level Authorization**: Verify the user's permission for the specific requested object on **every operation** (read, update, delete, export).
  - **Scoped Database Queries**: Restrict database queries to records associated with the authenticated user or tenant (e.g., `@current_user.projects.find(params[:id])` in Rails or `findByIdAndOwnerId(id, currentUser().getId())` in Spring).
  - **Explicit Ownership Checks**: If querying globally, explicitly verify that `object.getOwnerId()` matches `currentUser().getId()`, throwing an `AccessDeniedException` if mismatched.
  - **Session Identity Binding**: Derive the authenticated user's identity directly from the validated session or JWT token—never trust user identifiers submitted in request bodies, headers, or URL parameters.
  - **Multi-Step Flow Identifiers**: Pass object identifiers securely through server-side sessions during multi-step workflows to prevent user tampering.
  - **Identifier Complexity (Defense-in-Depth)**: Replace sequential or enumerable primary keys with random, non-enumerable values (e.g., UUIDs or cryptographically random strings). **Do not encrypt IDs** as a primary control, as secure implementation is error-prone.
- **Testing IDOR Controls**:
  - Create multiple test accounts with different authorization scopes (e.g., User A and User B).
  - Authenticate as User A and attempt to read, modify, or delete User B's objects by changing identifiers across all CRUD endpoints.
  - Verify that the application returns access-denied errors regardless of whether identifiers are predictable or random UUIDs.
- **Most Common Mistakes**:
  - Assuming UUIDs or complex identifiers replace the need for object-level authorization checks.
  - Executing unscoped, global database lookups based on raw user-supplied primary keys (e.g., `Project.find(params[:id])`).
  - Accepting the target user's identity or owner ID directly from request payloads.

### Mass Assignment Cheat Sheet
- **Concrete Actionable Rules**:
  - **Data Transfer Objects (DTOs)**: Create dedicated input DTOs that expose **only** the specific fields a user is authorized to edit during a given request.
  - **Field Allow-Listing**: When binding HTTP parameters directly to domain models or ORM entities, explicitly define allow-lists of bindable fields:
    - **Spring MVC**: Use `@InitBinder` with `binder.setAllowedFields("userid", "password", "email")`.
    - **Node.js / Mongoose**: Use `_.pick(req.body, ['userid', 'password', 'email'])`.
    - **PHP Laravel / Eloquent**: Define `protected $fillable = ['userid', 'password', 'email'];` on the model.
  - **Prohibit Block-Listing**: Do **not** rely on block-lists (e.g., `setDisallowedFields` or `$guarded`), as newly added or overlooked sensitive fields remain bindable.
- **Most Common Mistakes**:
  - Binding raw HTTP request payloads directly to domain or ORM models without filtering.
  - Relying on block-lists rather than explicit allow-lists.
  - Assuming that missing empty constructors or UI form field constraints prevent parameter autobinding.

### Multi-Tenant Security Cheat Sheet
- **Concrete Isolation Rules Across Layers**:
  - **Context & Identification Layer**:
    - Establish and verify tenant context early in the request lifecycle using middleware.
    - Treat client-supplied headers (e.g., `X-Tenant-ID`) as unverified selectors; always validate tenant membership against the server-verified user identity before binding request-local context.
  - **Database Layer**:
    - For shared-table Row-Level Security (RLS in PostgreSQL), enforce policies using `ENABLE ROW LEVEL SECURITY` and `FORCE ROW LEVEL SECURITY`.
    - **Crucial Rule**: **Never** connect ordinary tenant request paths using a database role with `SUPERUSER` or `BYPASSRLS` attributes, as both completely bypass RLS.
    - Scope tenant settings to the current transaction (`SET LOCAL` or `set_config('app.current_tenant', tenant_id, true)`) so session pooling does not leak tenant context across requests.
  - **Cache Layer**:
    - Include the tenant identifier in every tenant-scoped cache key (e.g., `tenant:{tenant_id}:{resource}:{id}`).
    - Explicitly namespace shared global entries (e.g., `global:{resource}:{version}`). Always re-authorize the user before returning cached data.
  - **Storage & Blob Layer**:
    - Partition object storage using tenant-specific prefixes or buckets (`storage/{tenant_id}/...`).
    - Validate tenant authorization on the server before issuing short-lived signed download/upload URLs.
  - **Queue & Asynchronous Layer**:
    - Bind authenticated tenant context to outgoing messages at the producer stage.
    - Require asynchronous consumers to re-authenticate the message producer, re-establish tenant context, and verify permissions before executing delayed jobs. Apply per-tenant rate limits to prevent noisy-neighbor queue starvation.
  - **Logging & Audit Layer**:
    - Enrich every log and security event with server-verified `tenant_id` and `user_id` context.
    - Enforce tenant authorization when reading audit trails, preventing tenant admins from viewing global or cross-tenant logs.
- **Most Common Mistakes**:
  - Trusting client-supplied tenant identifiers without verifying active user membership.
  - Serving tenant request paths using PostgreSQL roles with `SUPERUSER` or `BYPASSRLS` privileges.
  - Setting database session-level variables without `SET LOCAL`, leading to cross-tenant connection pool contamination.
  - Storing tenant-scoped data in shared cache keys without tenant namespaces.

### Business Logic Security Cheat Sheet
- **Concrete Actionable Controls**:
  - **Server-Side Value Re-Derivation**: Never accept prices, subtotals, taxes, or permission flags from the client. Accept product IDs and quantities only, re-calculating all financial figures from trusted server database records.
  - **Explicit Server State Machines**: Enforce multi-step workflows (checkouts, approvals, password resets) as explicit server-side state machines. Reject step skipping, reject replays of completed steps, and set explicit expiration TTLs on partial states.
  - **Atomic Race Condition Defense**: Treat check-then-act operations (e.g., balance check before debit, coupon validation before redemption) as critical sections. Enforce atomicity using:
    - `SELECT... FOR UPDATE` inside a database transaction.
    - Conditional updates: `UPDATE accounts SET balance = balance -:amt WHERE balance >=:amt` (check affected rows).
    - Unique database constraints on `(user_id, coupon_id)`.
  - **Contextual Authorization**: Validate business context rules (e.g., "users cannot approve their own expense reports" or "orders cannot be canceled after shipment") directly inside business logic handlers.
  - **Feature-Level Abuse Controls**: Implement rate limiting, action caps, and anomaly detection specifically on value-dispensing features (referrals, promo codes, password resets).
- **Testing Business Logic**:
  - Write automated tests specifically targeting state machine rules, out-of-order endpoint execution, step replays, and boundary values.
  - Conduct concurrency testing using multiplexed HTTP/2 or parallel requests to detect race conditions in check-then-act workflows.
  - Exercise the "dishonest user" perspective during threat modeling.
- **Most Common Mistakes**:
  - Trusting client-submitted pricing or total fields.
  - Using hidden HTML form fields or client-readable cookies to maintain workflow state.
  - Assuming microsecond time gaps between checks and updates are unexploitable race conditions.
  - Relying solely on technical vulnerability scanners, which cannot detect broken business logic or missing workflow state validation.
