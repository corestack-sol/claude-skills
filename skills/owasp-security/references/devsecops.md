# Containers, CI/CD and supply chain

Synthesized with NotebookLM from the OWASP Cheat Sheet Series (CC BY-SA 4.0). It is a
digest, not the source: when a decision is high-stakes or a value looks surprising,
open the cheat sheet below and confirm it there.

## Sources

- [Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [CI CD Security](https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html)
- [GitHub Actions Security](https://cheatsheetseries.owasp.org/cheatsheets/GitHub_Actions_Security_Cheat_Sheet.html)
- [Software Supply Chain Security](https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html)

## Rules

### Docker Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **Daemon Socket Isolation**: Never expose `/var/run/docker.sock` to containers or over TCP (`-H tcp://0.0.0.0:XXX`). Mounting `docker.sock` as a read-only volume (`-v /var/run/docker.sock:/var/run/docker.sock:ro`) **does not** make the Docker API read-only. Anyone who can write requests to the socket gains full host root privileges.
  - **Non-Root Execution & Rootless Mode**: Configure containers to run as an unprivileged user using the Dockerfile `USER` directive, runtime `-u <uid>` flag (e.g., `docker run -u 4000`), or Kubernetes `runAsUser: 4000`. Run Docker in rootless mode (`dockerd-rootless.sh`) or enable user namespaces (`--userns-remap=default`) so the container daemon and runtime run without host-root privileges.
  - **Linux Kernel Capabilities**: Drop all default kernel capabilities by default (`--cap-drop ALL` or Kubernetes `capabilities.drop: ["ALL"]`) and explicitly re-add only required capabilities (e.g., `--cap-add CHOWN`). **Never** run containers with the `--privileged` flag, as it grants all host kernel capabilities.
  - **Privilege Escalation Defense**: Pass `--security-opt=no-new-privileges` at runtime (or set `allowPrivilegeEscalation: false` in Kubernetes security contexts) to block processes from acquiring new privileges via `setuid` or `setgid` binaries.
  - **Read-Only Root Filesystem**: Enforce `--read-only` on the root filesystem (or `readOnlyRootFilesystem: true` in Kubernetes). Mount temporary writable paths using `--tmpfs /tmp` and append `:ro` (or `readonly: true`) to mounted volumes.
  - **Resource Limits**: Mitigate DoS attacks by capping memory (`--memory`), CPU usage (`--cpus`), file descriptors (`--ulimit nofile=<num>`), process counts (`--ulimit nproc=<num>`), and restart attempts (`--restart=on-failure:5`).
  - **Host Firewall / UFW Conflicts**: Docker manages its own `iptables`/`nftables` rules directly and **bypasses host firewalls like UFW entirely**. Publishing ports with `-p 8000:8000` opens the port on all interfaces regardless of UFW `DENY` rules. Always bind published ports specifically to localhost (`-p 127.0.0.1:8000:8000`) or deploy `ufw-docker` rules.
  - **Runtime Security Modules**: Keep default `seccomp`, `AppArmor`, or `SELinux` profiles enabled. Implement behavioral monitoring (e.g., Falco, Tetragon, or Cilium eBPF) to catch anomalous container executions.
- **Most Common Mistakes**:
  - Mounting `/var/run/docker.sock` into containers under the false assumption that a read-only volume mount secures the Docker API.
  - Running containers with the `--privileged` flag or as the default `root` user.
  - Publishing ports with `-p 8000:8000` on UFW-protected Linux hosts, unintentionally exposing services to the public internet.
  - Disabling default `seccomp` or `AppArmor` profiles to troubleshoot application permission errors.

### CI/CD Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **Pipeline & Execution Environment Isolation**: Run builds in temporary, isolated nodes or ephemeral containers that are completely destroyed after each job execution. Restrict OS user accounts executing pipelines to non-root privileges. Require manual approvals before executing production deployments.
  - **Secure SCM & Branch Protection**: Prohibit auto-merge rules in SCM platforms (GitHub, GitLab, Bitbucket). Enforce mandatory code reviews, signed commits, protected branch rules, and strict access controls on repository visibility changes.
  - **Secrets Management & Least Privilege**: Adopt a "deny by default" access model. Never hardcode secrets in code, CI configs, or Docker build arguments. Use centralized secrets managers (e.g., Vault, AWS Secrets Manager) and scan for leaks using `gitleaks` or `git-secrets`. Do not share credentials across pipelines of differing sensitivity levels.
  - **Managing Third-Party Code & Lockfiles**: Enforce deterministic builds using version pinning and lockfile enforcement (`npm ci` or `yarn install --frozen-lockfile`). Verify package checksums/hashes against pinned lockfiles. Route internal packages through scoped names (e.g., `@org/package`) and single private artifact feeds to block dependency confusion.
  - **Logging & Monitoring**: Log pipeline actions in structured formats (e.g., JSON or syslog). Ensure console outputs and command execution logs automatically redact/mask passwords, API keys, and session tokens.
- **Most Common Mistakes**:
  - Running CI/CD build agents with `--privileged` Docker flags or host root access.
  - Allowing auto-merge rules on pipelines without requiring un-bypassable human code reviews.
  - Printing unmasked secrets to console build logs or storing secrets in shell command histories.
  - Running `npm install` instead of `npm ci` in build scripts, allowing unpinned transitive dependency updates.

### GitHub Actions Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **Untrusted Input Injection Prevention**: Never expand untrusted context expressions directly in inline `run:` shell scripts (e.g., `${{ github.event.pull_request.title }}` or `${{ github.event.issue.body }}`). Always pass context values through intermediate environment variables (`env: TITLE: ${{ github.event.pull_request.title }}`) to prevent command injection.
  - **Dangerous Trigger Hardening**:
    - `pull_request_target`: Runs in the base repository context with a write-permission `GITHUB_TOKEN` and access to repository secrets. **Never** check out or execute untrusted code from PR forks within a `pull_request_target` workflow.
    - `workflow_run`: Grants elevated permissions and secrets; subject to cache and artifact poisoning. Replace with `workflow_call` and reusable workflows.
    - `issue_comment`: Vulnerable to TOCTOU attacks where an attacker modifies a PR between approval and execution. Validate actor authorization and bind checkout strictly to trusted commit SHAs (`/ok-to-test <trusted_sha_commit>`) or use label-based triggers with `github.event.pull_request.head.sha`.
  - **Immutable Version Pinning**: Pin all third-party GitHub Actions and reusable workflows to full 40-character commit SHAs (e.g., `uses: actions/checkout@a5ac7e51b41094...`), never tags or branch names. Verify that the pinned commit actually belongs to the target repository to block impostor commit attacks (auditable via Zizmor).
  - **Minimize **`GITHUB_TOKEN`** Permissions**: Set `permissions: {}` at the top workflow level to revoke all default privileges. Explicitly grant minimum required permissions (e.g., `contents: read`) at the individual job level.
  - **OIDC & Credentials**: Eliminate long-lived static credentials (e.g., AWS access keys or PATs) in favor of OpenID Connect (OIDC / "Trusted Publishing") to obtain short-lived, workflow-bound tokens.
  - **Checkout & Runner Security**: Execute `actions/checkout` with `persist-credentials: false` to prevent Git tokens from persisting in the runner environment. Restrict runner egress traffic using tools like `step-security/harden-runner`. **Never** use self-hosted runners on public repositories.
- **Most Common Mistakes**:
  - Checking out and running PR code inside `pull_request_target` or `workflow_run` workflows.
  - Directly referencing untrusted GitHub event inputs inside inline `run:` scripts (`echo "${{ github.event.issue.title }}"`).
  - Pinning actions to mutable tags (e.g., `@v4`) rather than immutable 40-character commit SHAs.
  - Leaving default `GITHUB_TOKEN` read/write permissions enabled repository-wide.
  - Forgetting `persist-credentials: false` on `actions/checkout`, leaving write credentials on the disk.

### Software Supply Chain Security Cheat Sheet
- **Concrete Actionable Rules**:
  - **Artifact Signing & Provenance**:
    - Generate verifiable build provenance (SLSA 1.0 compliant) using build platforms like `slsa-github-generator` and verify provenance downstream using `slsa-verifier`.
    - Digitally sign compiled binaries, container images, and npm packages using Sigstore (`cosign` / `sigstore-js`).
    - When verifying Sigstore signatures, validate both the expected certificate issuer (`https://token.actions.githubusercontent.com`) and anchored, regex-escaped certificate identity URIs.
  - **Software Bill of Materials (SBOM)**: Automatically generate an SBOM in CycloneDX or SPDX format during build pipelines (e.g., `npx @cyclonedx/cyclonedx-npm --validate > sbom.json`). Continuously cross-reference the SBOM against NVD, OSV, and CISA KEV vulnerability databases.
  - **Dependency Pinning & Verification**: Lock all direct and transitive dependencies using version pinning and lockfiles (`package-lock.json`, `Pipfile.lock`). Strictly enforce lockfiles in CI using `npm ci` or `yarn install --frozen-lockfile`. Compare package hashes against known-good checksums.
  - **Private Repositories & Scoped Names**: Use private artifact registries (or local proxies like Verdaccio) with upstream caching. Enforce scoped package naming (`@yourorg/package-name`) and bind scopes to your private registry in `.npmrc` to prevent dependency confusion.
- **Most Common Mistakes**:
  - Relying on unverified, unsigned third-party build artifacts without checking SLSA provenance or cryptographic signatures.
  - Building production releases without generating or archiving an SBOM.
  - Failing to enforce `npm ci` in build pipelines, allowing un-pinned transitive dependencies to update dynamically.
  - Storing internal packages in un-scoped public namespaces, exposing the organization to dependency confusion attacks.
