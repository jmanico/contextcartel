# SECURITY.md

# Context Cartel Security

This document is provisional, like ARCHITECTURE.md. REQUIREMENTS.md is the source of truth for what the system must do; ARCHITECTURE.md is the source of truth for component boundaries and the given stack (Node.js server, ReactJS client, REST/MCP API, passkey + OpenAuth auth, Terraform deployment). Where those leave a security decision open, that gap is preserved here as `TO BE DECIDED`/`UNKNOWN` rather than resolved by assumption.

## Required Security Inputs

- `Runtime/stack: Node.js server, ReactJS client, REST or MCP API, Terraform deployment, worldwide scale.` — `[GIVEN]` (ARCHITECTURE.md)
- `Authentication mechanism: passkey (end users), OpenAuth (MCP/CLI clients).` — `[GIVEN]` (ARCHITECTURE.md)
- `Authorization/role model: how passkey and OpenAuth identities map to operator / workspace owner / AI agent / peer-store roles — UNKNOWN, TO BE DECIDED (REQUIREMENTS.md FR-4.6, FR-6.6).`
- `Inter-store trust model: how one knowledge store verifies the identity of a peer store publishing announcements or enrolling a workspace — UNKNOWN, TO BE DECIDED (REQUIREMENTS.md FR-6.6, FR-8).`
- `Data classification / retention / regulatory scope: UNKNOWN, TO BE DECIDED (REQUIREMENTS.md). Worldwide scale plus ingestion of operator-supplied files, crawled web content, and workspace data makes this a likely candidate for data-protection regimes such as GDPR — this is a flag to resolve, not a confirmed obligation.`
- `Secrets/credential storage mechanism for the OpenAuth flow and any crawl/API credentials: UNKNOWN, TO BE DECIDED.`
- `Hosting provider(s)/regions for the Terraform-managed deployment: UNKNOWN, TO BE DECIDED (ARCHITECTURE.md).`
- `MCP tool/resource/prompt authorization behavior: UNKNOWN, TO BE DECIDED (REQUIREMENTS.md FR-4.6).`

## Provisional Security Rules

Durable defaults, safe to apply regardless of how the items above resolve.

### HTTP / API Boundary
- Enforce TLS on every external connection; no plaintext HTTP between clients, the REST/MCP facades, and any crawl target.
- Set security headers on every response: strict CSP (`default-src 'self'`, no `unsafe-inline`/`unsafe-eval`), HSTS, `X-Content-Type-Options: nosniff`, frame-ancestors none unless embedding is required.
- Enforce a CORS allow-list; never reflect `Origin` or use `*` on authenticated endpoints.
- Apply rate limiting, request size limits, and timeouts on every REST and MCP endpoint — the announcement, ingestion, and crawl endpoints are the highest-risk targets for abuse.
- Validate every inbound payload (REST body/query/headers, MCP tool arguments) against an explicit schema at the boundary; reject unknown fields rather than stripping them silently.

### Authentication and Authorization
- Enforce authentication at the API boundary (REST and MCP facades), never deep in domain logic, per ARCHITECTURE.md's dependency rules — this is what lets passkey and OpenAuth evolve independently.
- Passkey (WebAuthn/FIDO2) is the given mechanism for end users; do not add a fallback password path without an explicit decision, since it would reintroduce the credential-theft risk passkeys exist to remove.
- OpenAuth governs MCP/CLI clients; every MCP tool call and CLI-to-API request MUST carry a verifiable identity, not just a static API key.
- Authorization MUST be deny-by-default and checked server-side on every request; role-to-permission mapping (operator vs. workspace owner vs. AI agent vs. peer store) is `TO BE DECIDED` and MUST be resolved before any multi-tenant or public deployment.
- Regenerate/rotate session and token material on privilege changes (e.g., workspace enrollment, revocation).

### Input Validation
- Validate all external input (files, crawled content, MCP arguments, REST bodies, inter-store announcements) with an explicit schema at the point it enters the system; treat crawled and announced content as fully untrusted.
- Path/archive handling in the ingestion subsystem (FR-2) MUST resolve and validate paths against a base directory and check for symlinks/zip-slip before reading.
- The web crawl subsystem (FR-5) is an SSRF vector by design: crawl targets MUST be validated against an allow-list or, at minimum, block requests to private/internal IP ranges and cloud metadata endpoints, and MUST NOT follow redirects into private address space.
- Allow-list over deny-list wherever a filter, format check, or content-type check is possible.

### AI / MCP Context Integrity
- Content served through the MCP facade (evidence, concepts, crawled or enrolled data) can carry attacker-influenced text from crawled sites or peer-store announcements. Treat this as a prompt-injection surface: never let ingested/crawled/announced content control MCP tool selection, execution, or agent instructions — it is data, not instructions, when returned to an AI client.
- Preserve source/provenance metadata on everything returned over MCP (FR-4.4, FR-1.7) so a consuming agent — and an operator auditing it later — can distinguish trusted first-party input from crawled or imported content.

### Secrets Management
- No secrets in source, config committed to the repo, or logs. Use a secret manager (mechanism `TO BE DECIDED`) in every environment beyond local dev.
- Generate tokens and session identifiers with a cryptographically secure random source, never a non-cryptographic RNG.
- Redact authorization headers, cookies, and credential fields from all logs by default.

### Logging and Error Handling
- Never return stack traces, internal error messages, or raw exceptions to REST/MCP clients; return a generic error plus a correlation ID and log the detail server-side.
- Maintain a structured, append-only audit log covering ingestion, crawl, enrollment, filtering, announcement, and MCP access (REQUIREMENTS.md FR-9.2) — this is a functional requirement with direct security value (incident reconstruction, abuse detection).
- Never write raw untrusted strings into logs without neutralizing newlines/control characters (log injection).

### Deployment and CI/CD Safety (Terraform)
- Store Terraform state in a remote backend with encryption at rest and state locking; never commit state files.
- Apply least-privilege IAM to every resource Terraform provisions; no wildcard resource/action policies.
- Secrets belong in a secret manager referenced by Terraform, never in `.tfvars` or variable defaults committed to source.
- Require plan review before apply in CI; run dependency/IaC scanning (e.g., `tfsec`/`checkov`-class tooling) as a CI gate, not a manual step.
- CI pipelines MUST NOT print secrets to logs and MUST run from pinned, lockfile-verified dependencies (see Dependency Rules below).

## Selected Prompt Imports

Resolved from ARCHITECTURE.md's given stack; each replaces the matching placeholder below.

| Area | Source | Resolution |
|---|---|---|
| Architecture decisions | ARCHITECTURE.md Dependency Rules | Security enforced at the API boundary (REST + MCP), with a single context-filter chokepoint (FR-7.1) gating all writes into the knowledge graph — no new architectural security decision needed here, only enforcement. |
| Backend framework | Node.js `[GIVEN]` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Backend Frameworks/NodeJS/00 Secure Node.js Developer` |
| Frontend framework | ReactJS `[GIVEN]` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Client Side Frameworks/ReactJS/00 React19 Secure Generator (JS)` |
| Auth model | Passkey (users) `[GIVEN]` | https://fidoalliance.org/passkeys/ — covers the user-facing passkey flow only; OpenAuth for MCP/CLI clients has no equivalent public reference here and its role/permission mapping remains `TO BE DECIDED`. |
| Deployment model | Terraform `[GIVEN]` | https://www.wiz.io/academy/application-security/terraform-security-best-practices/ |

## Prompt Placeholders To Resolve

- `{{CODE_QUALITY_PROMPT}}` → **Resolved**: low cyclomatic complexity, low cognitive complexity, and separation of concerns for every module (domain/service layer, REST facade, MCP facade, ingestion, crawl, filter, enrollment, announcement subsystems stay independently testable per ARCHITECTURE.md's component boundaries).
- `{{API_SECURITY_PROMPT}}` → **Resolved** (API style is REST or MCP `[GIVEN]`): OWASP API Security Top 10 (https://owasp.org/www-project-api-security/) + OWASP REST Security Cheat Sheet (https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); apply the API-Security-Top-10 categories (broken object/function-level authorization, excessive data exposure, lack of resource/rate limiting) specifically to the MCP facade, since MCP is functionally a third API surface alongside REST here.
- `{{BACKEND_FRAMEWORK_PROMPT}}` → **Resolved** (Node.js `[GIVEN]`): see Selected Prompt Imports above.
- `{{FRONTEND_FRAMEWORK_PROMPT}}` → **Resolved** (ReactJS `[GIVEN]`): see Selected Prompt Imports above.
- `{{AUTH_PROMPT}}` → **Partially resolved**: passkey flow is covered (see Selected Prompt Imports); the OpenAuth side of the auth model, and the operator/workspace-owner/peer-store role mapping on top of both mechanisms, are `UNKNOWN`/`TO BE DECIDED`.
- `{{DEPLOYMENT_PROMPT}}` → **Resolved** (Terraform `[GIVEN]`): see Selected Prompt Imports above.

## Dependency Rules

- Do not add a dependency when the standard library or a few lines of first-party code will do.
- Prefer zero new dependencies. If a library is required, justify it in the PR description.
- Only use libraries that are actively maintained (commit or release within the last 12 months).
- Only use the latest stable major version. No deprecated, abandoned, or pre-release packages.
- Reject any library with known unpatched CVEs. Check before adding and on every update.
- Audit transitive dependencies, not just direct ones. A small direct dependency with a large or unvetted tree is a rejection.
- Pin exact versions with a committed lockfile. No floating ranges in production.
- Prefer libraries with a narrow scope, minimal dependencies of their own, and a clear security track record.

## Open Questions

- What authorization/role model maps passkey and OpenAuth identities onto operator, workspace owner, AI agent, and peer-store permissions?
- How does one knowledge store establish and verify trust with another before accepting an announcement or an enrollment request?
- What data classification, retention, deletion, and regulatory obligations (e.g., GDPR-class regimes) apply to ingested files, crawled content, workspace data, and announcements, given worldwide scale?
- What secret-management service backs OpenAuth credentials, crawl-target credentials, and Terraform-provisioned secrets?
- What hosting provider(s) and regions will the Terraform deployment target, and what does that imply for data residency?
- What authorization behavior applies to malformed or unauthorized MCP requests (FR-4.6)?
- What allow-list or verification process governs which web sources the crawl subsystem may reach, to bound the SSRF surface described above?
- Who owns security review/sign-off for new dependencies, and where is that recorded (PR template, CODEOWNERS, or elsewhere)?
