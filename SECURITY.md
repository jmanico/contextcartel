# SECURITY.md

# Context Cartel Security

This document is provisional, like ARCHITECTURE.md. REQUIREMENTS.md is the
source of truth for what the system must do; ARCHITECTURE.md is the source of
truth for component boundaries and the given stack (Node.js server, ReactJS
client, admin API/MCP API, passkey + OpenAuth auth, Terraform deployment).

This version incorporates a pre-code threat model. Where a security mechanism
is still `TO BE DECIDED`/`UNKNOWN`, that gap is preserved rather than guessed.
Critical security gaps are release gates: they MUST be resolved before public,
multi-tenant, or production deployment.

## Required Security Inputs

- `Runtime/stack: Node.js server, ReactJS client, admin API surface, MCP API, Terraform deployment, worldwide scale.` — `[GIVEN]` (ARCHITECTURE.md)
- `Authentication mechanism: passkey (end users), OpenAuth (MCP/CLI clients).` — `[GIVEN]` (ARCHITECTURE.md)
- `Authorization/role model: exact mapping from passkey/OpenAuth identities to operator, workspace owner, AI agent, CLI client, and peer-store permissions is UNKNOWN and TO BE DECIDED; deny-by-default server-side authorization is REQUIRED by REQUIREMENTS.md FR-10.1/FR-10.2.`
- `Inter-store trust model: exact identity/signing/replay mechanism is UNKNOWN and TO BE DECIDED; explicit, auditable, revocable trust and consent are REQUIRED by REQUIREMENTS.md FR-6.6/FR-8.10/FR-10.6.`
- `Data classification / retention / regulatory scope: UNKNOWN and TO BE DECIDED. Worldwide scale plus ingestion of operator-supplied files, crawled web content, workspace data, and announcements creates likely data-protection obligations; this is a flag to resolve, not a confirmed obligation.`
- `Secrets/credential storage mechanism for OpenAuth, passkey support services, crawl/API credentials, and Terraform-provisioned secrets: UNKNOWN and TO BE DECIDED.`
- `Hosting provider(s)/regions for the Terraform-managed deployment: UNKNOWN and TO BE DECIDED (ARCHITECTURE.md).`
- `MCP tool/resource/prompt authorization behavior: UNKNOWN and TO BE DECIDED; state-changing MCP capabilities are disabled by default per REQUIREMENTS.md FR-4.8.`
- `Threat model method: repository reconnaissance, document/architecture absorption, STRIDE, LINDDUN, attack-tree review, FMEA, AI/MCP context review, and consolidation using /Users/jmanico/Dropbox/github/experimental/prompts/Security Assessment Prompts/Threat Modeling/.`
- `Threat model limitation: this repository is pre-code/docs-only. Controls below are required design constraints, not verified implementation controls.`

## Threat Model Summary

### High-Value Assets

- Knowledge stores, knowledge graphs, files, evidence, concepts, graph links,
  provenance metadata, context filters, imported knowledge, crawl sources and
  results, workspace enrollments, announcements, transfer decisions, MCP
  tools/resources/prompts, admin API, CLI, web admin, audit logs, passkey and
  OpenAuth material, crawl credentials, secrets, Terraform state, and deployment
  infrastructure.

### Primary Trust Boundaries

| Boundary | Examples | Required control family |
|---|---|---|
| Client/API | Web admin, CLI, MCP clients, peer stores | Authentication, schema validation, rate limits, authorization, audit |
| MCP/AI context | AI agents retrieving graph-backed context | Tool scope, data minimization, prompt-injection separation |
| Untrusted content | Files, archives, crawled pages, workspace imports, announcements | Validation, quarantine, classification, provenance, review |
| Crawler egress | Crawler to public web | Allow-list/policy, DNS and redirect revalidation, SSRF blocks |
| Peer store | Enrollment, announcements, transfer | Trust registry, identity verification, replay protection, revocation |
| Graph write | Candidate knowledge to committed graph | Import safety pipeline, conflict handling, provenance integrity |
| Graph read | Admin API and MCP retrieval | Object-level auth, data-class filtering, query/response limits |
| Deployment control plane | Terraform, CI/CD, secrets, state | Encrypted state, least privilege, secret manager, CI gates |

### Consolidated Findings

| ID | Severity | Methodology | Threat | Required posture |
|---|---|---|---|---|
| T-01 | Critical | STRIDE / API | Broken authorization or tenant/workspace isolation leaks or mutates graph data across operators, workspaces, AI clients, CLI clients, or peer stores. | Deny-by-default RBAC/ABAC, per-object and per-data-class checks, tenant isolation. |
| T-02 | High | STRIDE / Attack Tree | Spoofed, compromised, or replayed peer-store announcements/enrollment requests poison or exfiltrate knowledge. | Revocable trust registry, authenticated messages, replay protection, scoped consent. |
| T-03 | Critical | STRIDE / AI | Undefined MCP tools/resources/prompts expose excessive data or grant state-changing abilities to AI agents. | Least-privilege MCP surface, read-only default, separate mutation scopes, per-tool audit. |
| T-04 | Critical | AI / Attack Tree | Prompt injection in crawled/imported/announced content causes downstream agents to treat data as instructions. | Strict data/instruction separation, provenance labels, no content-driven tool policy. |
| T-05 | Critical | STRIDE / FMEA | Malicious or stale content poisons evidence, concepts, graph links, or provenance. | Quarantine, immutable provenance, trust labels, conflict handling, review gates. |
| T-06 | High | STRIDE / API | Crawler can be abused for SSRF, metadata theft, internal network probing, or unsafe redirects. | Egress isolation, private-network blocks, DNS/redirect revalidation, crawl limits. |
| T-07 | High | STRIDE / FMEA | Malicious files or archives exploit parsers, escape paths, or exhaust resources. | File allow-list, size/expansion limits, path/archive validation, parser sandboxing policy. |
| T-08 | High | LINDDUN | Announcements leak sensitive metadata, source names, record existence, personal data, or retrieval pointers. | Metadata-only default, DLP/classification checks, TTL/size bounds, export policy. |
| T-09 | High | LINDDUN | Worldwide processing proceeds without classification, retention, deletion, consent, residency, or privacy review. | Data classification and privacy-impact review before non-local processing. |
| T-10 | High | STRIDE / FMEA | Ingestion, crawling, graph queries, MCP calls, or announcements exhaust compute/storage. | Abuse limits, queue controls, query-cost budgets, response-size limits. |
| T-11 | Medium/High | STRIDE | React admin or CLI renders untrusted evidence/crawled/announcement content unsafely. | Output encoding/sanitization, no executable content from graph data, strict CSP. |
| T-12 | High | STRIDE | Missing or mutable audit history allows repudiation of imports, transfers, deletions, or MCP access. | Tamper-evident audit events with actor, object, decision, correlation ID, timestamp. |
| T-13 | Medium | Repo Recon | Secrets, Terraform state, or dependency/CI configuration leaks deployment authority. | Secret manager, encrypted/locked state, least-privilege IAM, pinned dependencies, CI scanning. |

## Security Release Gates

The following MUST be resolved before any public, multi-tenant, or production
deployment:

- Authorization matrix for operators, workspace owners/operators, MCP clients,
  CLI clients, AI agents, and peer stores.
- Tenant/workspace/knowledge-store isolation model and object-level access
  checks for every graph read/write path.
- MCP tools/resources/prompts, read-only versus state-changing capabilities,
  scopes, and per-tool authorization behavior.
- Peer-store trust model, consent flow, revocation behavior, message
  authentication, replay protection, and trust-registry ownership.
- Data classification, retention, deletion/disablement, tombstone, privacy,
  data-residency, and regulatory scope.
- Crawl-source approval policy, SSRF controls, crawler egress architecture,
  robots/consent behavior, and HTTP-versus-HTTPS policy.
- File-ingestion allow-list, parser sandboxing policy, malware/content scanning
  decision, and resource limits.
- Announcement schema, TTL, size limits, data-minimization rules, and export
  policy.
- Audit event schema, tamper-evident storage, retention, access control, and
  alerting thresholds.
- Secret-management mechanism, Terraform state backend, hosting provider/region
  choices, and CI/IaC/dependency scanning gates.

## Provisional Security Rules

Durable defaults, safe to apply regardless of how the `TO BE DECIDED` items
resolve.

### HTTP / API Boundary

- Enforce TLS on every first-party external connection between clients, admin
  API, MCP facade, peer-store gateway, and deployment services.
- Crawl targets SHOULD use HTTPS. Plaintext HTTP crawling MAY be allowed only by
  explicit operator policy and MUST label resulting content as insecurely
  transported.
- Set security headers on every browser-facing response: strict CSP
  (`default-src 'self'`, no `unsafe-inline`/`unsafe-eval`), HSTS,
  `X-Content-Type-Options: nosniff`, and `frame-ancestors 'none'` unless
  embedding is explicitly required.
- Enforce a CORS allow-list; never reflect `Origin` or use `*` on authenticated
  endpoints.
- Apply rate limiting, request size limits, response size limits, query-cost
  limits, and timeouts on every admin API and MCP endpoint. Announcement,
  ingestion, crawl, graph-query, and state-changing MCP endpoints are the
  highest-risk targets for abuse.
- Validate every inbound payload (admin API body/query/headers, MCP tool
  arguments, CLI requests, peer-store messages, crawler configuration) against
  an explicit schema at the boundary; reject unknown fields rather than
  stripping them silently.

### Authentication and Authorization

- Authenticate at protocol facades and peer-store boundaries. Normalize the
  authenticated principal and client identity before invoking shared domain
  services.
- Authorization MUST be deny-by-default and enforced through the shared
  identity/policy layer before every graph read, graph write, import, export,
  crawl configuration, enrollment action, announcement action, deletion,
  disablement, and MCP tool call.
- Authorization MUST include actor role, client identity, workspace/store/graph
  scope, object ownership, source identity, data classification, requested
  operation, and tool scope where applicable.
- Any multi-store or multi-workspace deployment MUST enforce tenant isolation at
  every storage, API, cache, search, audit, and background-job boundary.
- Passkey (WebAuthn/FIDO2) is the given mechanism for end users. Do not add a
  fallback password path without an explicit decision, since it would
  reintroduce the credential-theft risk passkeys exist to remove.
- OpenAuth governs MCP/CLI clients. Every MCP tool call and CLI-to-API request
  MUST carry a verifiable identity with audience, scope, expiration, and
  revocation semantics; static API keys are not sufficient.
- State-changing MCP tools MUST require scopes and policy decisions distinct
  from read-only retrieval, and read-only clients MUST NOT receive mutation
  tools.
- Regenerate, rotate, or revoke session and token material on privilege changes
  such as workspace enrollment, scope changes, trust revocation, or compromised
  client detection.

### Input and Content Handling

- Validate all external input (files, archives, crawled content, MCP arguments,
  admin API bodies, CLI requests, inter-store announcements, enrollment
  messages) with an explicit schema at the point it enters the system.
- Treat files, crawled content, imported workspace data, and announcements as
  fully untrusted until the import safety pipeline accepts them.
- Path/archive handling in the ingestion subsystem MUST resolve and validate
  paths against a base directory and reject symlinks, path traversal, zip-slip,
  archive bombs, unsupported file types, and excessive expansion.
- File parsers and content extractors MUST run with resource limits. Parser
  sandboxing and malware/content scanning mechanisms are TO BE DECIDED, but the
  decision MUST be resolved before non-local ingestion of third-party files.
- The web crawl subsystem is an SSRF vector by design: crawl targets MUST be
  validated against an allow-list or approval policy, private/internal IP ranges
  and cloud metadata endpoints MUST be blocked, redirects MUST be revalidated,
  and DNS rebinding MUST be handled.
- Allow-list over deny-list wherever a filter, format check, content type, crawl
  target, or parser choice is possible.
- Web admin and CLI output MUST encode or sanitize untrusted graph/evidence/file
  content for the output medium. The web admin MUST NOT render raw HTML or
  executable content from graph data unless a separately reviewed sanitizer and
  policy explicitly allow it.

### Import, Graph, and Provenance Integrity

- Imported, crawled, announced, and workspace-enrolled content MUST enter
  quarantine or candidate state before graph commit unless a configured policy
  explicitly allows automatic acceptance for that source, trust level, data
  class, and operation.
- Accepted imports MUST preserve immutable provenance and decision rationale:
  source type, source identity, trust level, import path, timestamp, reviewer or
  policy decision, data classification when known, and freshness/conflict
  status.
- External or inferred content MUST NOT be promoted to first-party/operator
  approved trust level without an authorized and audited decision.
- Duplicate, stale, and contradictory evidence MUST be surfaced as conflicts or
  candidates; the system MUST NOT silently overwrite or merge content across
  different provenance or trust levels.
- Graph write paths MUST be reachable only through the import safety pipeline or
  authorized operator mutation workflows; no subsystem may bypass policy,
  provenance, classification, and audit requirements.

### AI / MCP Context Integrity

- Content served through the MCP facade can carry attacker-influenced text from
  files, crawled sites, enrolled workspaces, or peer-store announcements. Treat
  this as a prompt-injection surface.
- MCP responses MUST include source, provenance, trust level, data
  classification, and freshness/conflict status for each returned item when
  known.
- MCP responses MUST minimize data to what the requester is authorized to see
  and what the tool/resource request requires.
- Ingested, crawled, enrolled, or announced content MUST NOT control MCP tool
  selection, tool definitions, prompt templates, execution policy, authorization
  policy, or agent instructions.
- MCP state-changing tools MUST be disabled by default, separately scoped,
  audited per call, and protected by explicit policy checks.
- MCP errors MUST be generic and include a correlation ID; they MUST NOT reveal
  graph existence, authorization policy internals, stack traces, or sensitive
  source details.

### Peer Stores and Announcements

- Peer-store announcements and enrollment requests MUST be authenticated,
  integrity protected, replay protected, schema validated, scoped, rate limited,
  and checked against a revocable trust registry before processing.
- Trust registry entries MUST include peer identity, allowed scopes, consent
  status, expiration/freshness policy, and revocation state; exact identity and
  signing mechanisms are TO BE DECIDED.
- Announcement creation MUST apply data-loss-prevention/classification checks
  and MUST default to metadata-only summaries or approved retrieval pointers.
- Announcements MUST NOT include source file content, personal data, secrets,
  credentials, regulated data, or sensitive internal metadata unless an explicit
  export policy authorizes that disclosure.
- Announcement relevance decisions and accept/skip behavior SHOULD avoid leaking
  sensitive receiving-store interests to untrusted peers; exact side-channel
  controls are TO BE DECIDED.
- Revoked or expired peer trust MUST fail closed for new enrollment, new
  announcements, and new transfers. Existing imported data handling remains TO
  BE DECIDED and MUST be resolved before production use.

### Privacy and Data Protection

- Data classification MUST be defined before non-local deployment and MUST cover
  files, evidence, concepts, graph links, workspaces, crawled content,
  announcements, MCP responses, exports, logs, and audit records.
- The system MUST NOT expose restricted, private, personal, regulated, secret,
  or credential data through MCP, admin API, CLI, web admin, announcements,
  exports, or logs unless the requester is authorized for that data class and
  source scope.
- Retention, deletion/disablement, tombstone, provenance preservation, and audit
  retention rules MUST be defined before processing personal, regulated, or
  third-party workspace data outside local development.
- Privacy-impact review MUST be completed before worldwide or multi-tenant
  deployment when personal, regulated, or third-party workspace data is
  processed.
- Logs and audit records may themselves contain sensitive metadata; access to
  them MUST be authorized and minimized.

### Secrets Management

- No secrets in source, committed config, graph content, announcements, MCP
  responses, exported files, or logs.
- Use a secret manager (mechanism TO BE DECIDED) in every environment beyond
  local development.
- Generate tokens and session identifiers with a cryptographically secure random
  source, never a non-cryptographic RNG.
- Redact authorization headers, cookies, passkey/OpenAuth material, credentials,
  API tokens, crawl secrets, and Terraform secrets from logs by default.

### Logging and Error Handling

- Never return stack traces, internal error messages, raw exceptions, policy
  internals, or raw datastore errors to admin API or MCP clients.
- Return a generic error plus a correlation ID and log the detail server-side
  according to the audit/logging policy.
- Maintain structured, append-only or tamper-evident audit logs covering
  authentication, authorization, ingestion, parsing, classification, graph
  mutations, crawl configuration and fetches, enrollment, filtering,
  announcements, transfers, deletion/disablement, exports, and MCP access.
- Audit events MUST record actor identity, client identity, source
  workspace/store, action, object IDs, policy decision, data classification when
  known, correlation ID, and timestamp. Exact schema is TO BE DECIDED.
- Never write raw untrusted strings into logs without neutralizing
  newlines/control characters.
- Audit log readers MUST be authorized separately from ordinary graph readers.

### Deployment and CI/CD Safety (Terraform)

- Store Terraform state in a remote backend with encryption at rest and state
  locking; never commit state files.
- Apply least-privilege IAM to every resource Terraform provisions; no wildcard
  resource/action policies.
- Secrets belong in a secret manager referenced by Terraform, never in `.tfvars`
  or variable defaults committed to source.
- Require plan review before apply in CI; run dependency/IaC scanning
  (for example, `tfsec`/`checkov`-class tooling) as a CI gate, not a manual step.
- CI pipelines MUST NOT print secrets to logs and MUST run from pinned,
  lockfile-verified dependencies (see Third-Party Library Rules below).
- Public, multi-tenant, or production deployment MUST NOT proceed until the
  Security Release Gates in this document are resolved.

## Selected Prompt Imports

Resolved from ARCHITECTURE.md's given stack and this threat model; each replaces
the matching placeholder below.

| Area | Source | Resolution |
|---|---|---|
| Threat modeling | `/Users/jmanico/Dropbox/github/experimental/prompts/Security Assessment Prompts/Threat Modeling/` | Used repository reconnaissance, document absorption, consolidation, STRIDE, LINDDUN, attack trees, FMEA, and AI/MCP context review. |
| Architecture decisions | ARCHITECTURE.md Dependency Rules | Security is enforced through protocol facades, a shared identity/policy layer, import safety pipeline, trust registry, crawler egress controls, and graph read/write chokepoints. |
| Backend framework | Node.js `[GIVEN]` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Backend Frameworks/NodeJS/00 Secure Node.js Developer` |
| Frontend framework | ReactJS `[GIVEN]` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Client Side Frameworks/ReactJS/00 React19 Secure Generator (JS)` |
| Auth model | Passkey (users) `[GIVEN]` | https://fidoalliance.org/passkeys/ — covers the user-facing passkey flow only; OpenAuth for MCP/CLI clients and role/permission mapping remain `TO BE DECIDED`. |
| Deployment model | Terraform `[GIVEN]` | https://www.wiz.io/academy/application-security/terraform-security-best-practices/ |

## Prompt Placeholders To Resolve

- `{{CODE_QUALITY_PROMPT}}` → **Resolved**: low cyclomatic complexity, low
  cognitive complexity, and separation of concerns for every module
  (domain/service layer, admin API facade, MCP facade, identity/policy layer,
  ingestion, crawl, import safety, filter, enrollment, announcement, trust
  registry, and audit subsystems stay independently testable per
  ARCHITECTURE.md's component boundaries).
- `{{API_SECURITY_PROMPT}}` → **Resolved**: OWASP API Security Top 10
  (https://owasp.org/www-project-api-security/) + OWASP REST Security Cheat
  Sheet (https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html);
  apply broken object/function-level authorization, excessive data exposure,
  resource/rate limiting, and unsafe consumption of APIs specifically to MCP,
  peer-store, announcement, crawl, and admin API surfaces.
- `{{BACKEND_FRAMEWORK_PROMPT}}` → **Resolved** (Node.js `[GIVEN]`): see
  Selected Prompt Imports above.
- `{{FRONTEND_FRAMEWORK_PROMPT}}` → **Resolved** (ReactJS `[GIVEN]`): see
  Selected Prompt Imports above.
- `{{AUTH_PROMPT}}` → **Partially resolved**: passkey flow is covered (see
  Selected Prompt Imports); OpenAuth scopes/audience/expiry/revocation, MCP/CLI
  client identity, and operator/workspace-owner/peer-store role mapping are
  `UNKNOWN`/`TO BE DECIDED`.
- `{{DEPLOYMENT_PROMPT}}` → **Resolved** (Terraform `[GIVEN]`): see Selected
  Prompt Imports above.

## Third-Party Library Rules

- Do not add a dependency when the standard library or a few lines of
  first-party code will do.
- Prefer zero new dependencies. If a library is required, justify it in the PR
  description.
- Only use libraries that are actively maintained (commit or release within the
  last 12 months).
- Only use the latest stable major version. No deprecated, abandoned, or
  pre-release packages.
- Reject any library with known unpatched CVEs. Check before adding and on every
  update.
- Audit transitive dependencies, not just direct ones. A small direct dependency
  with a large or unvetted tree is a rejection.
- Pin exact versions with a committed lockfile. No floating ranges in
  production.
- Prefer libraries with a narrow scope, minimal dependencies of their own, and a
  clear security track record.

## Open Questions

- What authorization/role model maps passkey and OpenAuth identities onto
  operator, workspace owner, AI agent, CLI client, and peer-store permissions?
- What tenant/workspace/knowledge-store isolation model applies when multiple
  stores, workspaces, or clients share one deployment?
- How does one knowledge store establish, verify, scope, expire, and revoke
  trust with another before accepting an announcement or enrollment request?
- What message authentication, signing, replay-protection, and key-rotation
  mechanisms apply to peer-store announcements and enrollment?
- What data classification, retention, deletion/disablement, tombstone,
  provenance-preservation, and regulatory obligations apply to ingested files,
  crawled content, workspace data, audit logs, MCP responses, and announcements?
- What secret-management service backs OpenAuth credentials, passkey support
  services, crawl-target credentials, and Terraform-provisioned secrets?
- What hosting provider(s) and regions will the Terraform deployment target,
  and what does that imply for data residency?
- What MCP tools/resources/prompts are exposed, which are read-only, which can
  mutate state, and what scopes authorize each tool?
- What allow-list or verification process governs which web sources the crawl
  subsystem may reach, to bound the SSRF surface described above?
- What parser sandboxing, malware/content scanning, and file-type allow-listing
  policy applies to file ingestion?
- What announcement schema, TTL, size bounds, data-minimization rules, and
  export policy apply?
- What audit-log backend, event schema, retention duration, alert thresholds,
  and access-control model apply?
- Who owns security review/sign-off for new dependencies, and where is that
  recorded (PR template, CODEOWNERS, or elsewhere)?
