# ARCHITECTURE.md

# Context Cartel Architecture

This document is provisional. REQUIREMENTS.md is the source of truth for what the system must do; DESIGN.md is the source of truth for the visual/frontend design language the client architecture must support. Where either leaves a decision open, that gap is preserved here as `TO BE DECIDED` rather than resolved by assumption.

## Required Architecture Inputs

- `Requirements source: REQUIREMENTS.md`
- `System purpose: Context Cartel is a file-based knowledge store that represents evidence and concepts as a knowledge graph, exposes that knowledge to AI engines through an MCP server, updates/refines content through web crawling, and enables relevant information transfer between enrolled workspaces and knowledge stores.`
- `Primary use cases: Ingest file-based source material; identify evidence and concepts and link them in a knowledge graph; expose graph-backed context through MCP; crawl web sources to update/refine content; enroll other workspaces into an existing knowledge graph with context-based import filtering; publish and read inter-store announcements to support information transfer; give operators visibility into and control over store contents and activity.`
- `Target users / actors: Knowledge store administrators/operators; workspace owners/operators; AI engines/AI agents consuming context through MCP; peer knowledge stores that publish and read announcements.`
- `Runtime environment: web application`
- `Server framework: Node.js`
- `Client framework: ReactJS`
- `API style and integration model: MCP is required for AI engines/agents (REQUIREMENTS.md FR-4). The web admin and CLI require a server API surface, likely REST or equivalent; exact admin API protocol remains TO BE DECIDED.`
- `Authentication and session model: User: passkey, MCP/CLI: OpenAuth`
- `Data model expectations: Core entities per REQUIREMENTS.md: Knowledge Store, Knowledge Graph, File, Evidence, Concept, Graph Link/Relationship (with provenance metadata distinguishing known/inferred/imported — FR-1.7), Workspace, Workspace Enrollment, Context Filter, Import Candidate, Imported Knowledge, MCP Server Endpoint/Resource, Web Crawl Source, Crawled Content, Announcement, Announcement Reader, Information Transfer. Exact field-level schemas are TO BE DECIDED (REQUIREMENTS.md Open Questions).`
- `Deployment model: terraform`
- `Scale expectations: worldwide`
- `Security expectations: threat-model-driven baseline is now defined in SECURITY.md and REQUIREMENTS.md FR-10; unresolved critical security decisions are release gates, not optional implementation details.`

## Initial Architecture (Provisional)

**Assumption labels used below:** `[ASSUMPTION]` = not stated in REQUIREMENTS.md/DESIGN.md, inferred to make the architecture coherent; `[GIVEN]` = taken directly from explicit human-provided inputs above; `[TBD]` = open gap, intentionally not resolved.

### Component boundaries

1. **Web admin client (React)** `[GIVEN: Client framework]`
   - Serves the operator-facing surfaces enumerated in DESIGN.md's Components section.
   - Must provide a non-visual (text/table) equivalent of the graph explorer per DESIGN.md Accessibility section (FR-9.1/FR-9.3) — `[ASSUMPTION]` this is a rendering concern inside the same client, not a separate service.
   - Consumes the backend through the admin API surface `[TBD: exact REST/equivalent protocol]`; it MUST NOT access the knowledge graph store or crawl/enrollment/announcement subsystems directly.
   - Renders untrusted file, evidence, crawled, imported, and announcement content as data only, consistent with REQUIREMENTS.md FR-9.9 and SECURITY.md's React/admin content rules.

2. **Server (Node.js)** `[GIVEN: Server framework]`
   - Hosts the knowledge store/graph domain logic (FR-1), ingestion pipeline (FR-2), linking (FR-3), crawl subsystem (FR-5), enrollment (FR-6), context-based filtering (FR-7), and announcements (FR-8).
   - Exposes protocol facades over one shared domain/service layer:
     - An admin API surface for the web admin client and any CLI; exact protocol is `[TBD]`.
     - An MCP server endpoint for AI engines/agents (FR-4). `[ASSUMPTION]` these are protocol facades over one shared domain/service layer, not independently implemented backends — REQUIREMENTS.md treats "expose graph-backed context through MCP" as one of several ways to reach the same knowledge graph.
   - Exact MCP tools/resources/prompts exposed (FR-4.5) are `[TBD]` per REQUIREMENTS.md Open Questions.

3. **Identity, policy, and audit layer** — enforces passkey/OpenAuth authentication binding, normalized principal identity, deny-by-default authorization, data-class/source/workspace/graph scoping, and structured audit emission for admin API, MCP, CLI, peer-store, ingestion, crawl, enrollment, announcement, and graph operations (REQUIREMENTS.md FR-4.7/FR-9.6/FR-10.1/FR-10.2).
   - `[TBD]` exact role/permission model, policy engine shape, audit backend, and OpenAuth scope model.
   - This layer is a security chokepoint, not a separate business-domain implementation.

4. **CLI** — DESIGN.md's Platform targets name a CLI alongside MCP server and web, resolving REQUIREMENTS.md FR-9.5 (operator interface type = web admin + CLI). `[ASSUMPTION]` the CLI is a thin client over the same admin API surface as the web admin, not a separate implementation of domain logic.

5. **Import safety pipeline** — a shared gate for all untrusted or external candidate knowledge before committed graph writes.
   - Applies to file ingestion, web crawl results, enrolled workspace imports, announcement-driven transfers, and any future external source.
   - Pipeline order is source adapter -> schema/path/URL validation -> quarantine/candidate state -> extraction/parsing -> data classification -> provenance/trust labeling -> relevance filtering -> policy/review decision -> graph write.
   - Exact quarantine storage, parser sandboxing, malware scanning, data-classification implementation, and review workflow are `[TBD]`.

6. **Ingestion subsystem** (FR-2) — accepts operator-supplied files, extracts candidate evidence/concepts, reports import/skip/fail results. Where files are stored/referenced from is `[TBD]` (REQUIREMENTS.md Open Question). It MUST hand candidate content to the import safety pipeline before any graph write.

7. **Web crawl subsystem** (FR-5) — configurable crawl targets, discovers and evaluates candidate content, links imports to source, reports results.
   - MUST run with constrained network egress and target revalidation so crawler workers cannot reach private/internal/cloud-metadata addresses (REQUIREMENTS.md FR-5.8).
   - Crawl scheduling/frequency/robots/allow-deny rules are `[TBD]`.
   - Crawled content MUST hand off to the import safety pipeline before any graph write.

8. **Context-based import filter** (FR-7) — a shared evaluation step used by ingestion, crawl, workspace enrollment (FR-6.4), and announcement-driven transfer (FR-8.5), since these flows all require candidate evaluation before import. `[ASSUMPTION]` this is implemented as one reusable filtering component rather than duplicated per-caller. Filtering criteria and thresholds are `[TBD]`, but the filter MUST include source trust, data classification, provenance, freshness, duplicate/conflict status, and import policy in addition to topical relevance.

9. **Trust registry** — records approved peer stores, workspace enrollments, crawl sources, scopes, consent status, source identities/signing keys or equivalent identities, expiration/freshness policy, and revocation state. Exact storage and identity mechanism are `[TBD]`; the component boundary is required by REQUIREMENTS.md FR-6.6/FR-8.10/FR-10.6.

10. **Workspace enrollment subsystem** (FR-6) — enrolls a source workspace into a target knowledge graph, applies trust registry checks, hands candidates to the import safety pipeline/context filter, reports results, and emits audit events. Exact trust/consent/access-control/revocation mechanics are `[TBD]`, but the security behavior is required by FR-6.6 through FR-6.9.

11. **Announcement / peer-store gateway subsystem** (FR-8) — publishes outgoing announcements identifying the store and describing available data; reads and evaluates incoming announcements from peer stores; verifies peer identity/trust/replay state; may hand off to the enrollment/import path when policy allows (FR-8.6/FR-8.7). Announcement schema and inter-store trust mechanism are `[TBD]`.

12. **Knowledge graph store** — the persistent representation of Files, Evidence, Concepts, and Links with immutable provenance metadata (FR-1.7/FR-1.8). No specific data store technology is chosen here, per instructions not to select one without an explicit basis in REQUIREMENTS.md or human input — this remains `[TBD]`.

### Cross-cutting, explicitly deferred

- **Authentication/session model** is `[GIVEN]`: end users authenticate via passkey; MCP and CLI clients authenticate via OpenAuth. How these two schemes map onto authorization/roles for the actors in REQUIREMENTS.md (operator vs. workspace owner vs. AI agent vs. peer store) is `[TBD]`, but deny-by-default policy enforcement is required by REQUIREMENTS.md FR-10.1/FR-10.2.
- **Security expectations** are no longer wholly deferred: SECURITY.md contains the threat-model baseline. Exact mechanisms remain `[TBD]` only where explicitly marked, and SECURITY.md/REQUIREMENTS.md define which unresolved decisions block deployment.
- **Deployment model** is `[GIVEN]` as Terraform-managed infrastructure; the specific cloud provider(s), regions, and topology needed to satisfy `[GIVEN: worldwide scale]` are `[TBD]`.
- **Data classification, retention, privacy/regulatory constraints** are `[TBD]` and are release gates before non-local processing of third-party, personal, restricted, or regulated data.

### What this architecture deliberately does not choose yet

Per instructions, the following are left open rather than guessed: specific database/storage technology, hosting provider, message queue or job-scheduling technology for crawling, MCP transport/framework specifics, UI component library, CI/CD tooling, secret manager, audit-log backend, policy-engine implementation, trust-registry storage, parser sandbox technology, malware scanning provider, and data-classification tooling.

## Trust Boundaries and Chokepoints

The threat model identifies the following boundaries as architectural constraints:

| Boundary | Crossing flows | Required chokepoint |
|---|---|---|
| Client/API boundary | Web admin, CLI, MCP clients, peer stores | Authentication, schema validation, rate limits, authorization policy, audit emission |
| OpenAuth/MCP boundary | AI engines/agents requesting context or tools | Per-tool authorization, data minimization, read-only default, prompt-injection separation |
| Untrusted content boundary | Files, archives, crawled content, enrolled workspace data, announcements | Import safety pipeline with quarantine/candidate state before graph writes |
| Crawler egress boundary | Server/crawler to public web | Source allow-list/policy, redirect and DNS revalidation, private-network/metadata blocking |
| Peer-store boundary | Announcement publish/read, enrollment, inter-store transfer | Trust registry, peer identity verification, replay protection, revocation checks |
| Graph write boundary | Candidate imports to committed knowledge graph | Classification, provenance, relevance filtering, policy/review gate, conflict handling |
| Graph read boundary | Admin API and MCP retrieval | Object-level authorization, data-class filtering, response-size/query-cost limits, provenance labels |
| Deployment control-plane boundary | Terraform, CI/CD, secrets, state | Remote encrypted/locked state, least privilege, secret manager, plan/scanner gates |

## Requirement Traceability

| Architecture component | Requirement group(s) | Status |
|---|---|---|
| Knowledge graph store | FR-1 (Knowledge Store and Graph Model) | Component identified; schema `TO BE DECIDED` |
| Identity, policy, and audit layer | FR-4.7, FR-9.6 through FR-9.8, FR-10 | Component required; exact role matrix, policy engine, and audit backend `TO BE DECIDED` |
| Import safety pipeline | FR-2.7 through FR-2.10, FR-5.10, FR-7, FR-10.8 | Component required; quarantine, scanner, parser sandbox, and review workflow `TO BE DECIDED` |
| Ingestion subsystem | FR-2 (File-Based Ingestion) | Component identified; file storage location `TO BE DECIDED` |
| Linking logic (shared with graph store) | FR-3 (Evidence and Concept Linking) | Component identified; candidate-vs-accepted link review workflow `TO BE DECIDED` |
| MCP server facade | FR-4 (MCP Server Exposure) | Component identified; tool/resource/prompt surface and exact failure semantics `TO BE DECIDED`; per-tool authorization required by FR-4.7 |
| Web crawl subsystem | FR-5 (Web Crawling and Content Refresh) | Component identified; scheduling/robots/allow-deny rules `TO BE DECIDED` |
| Workspace enrollment subsystem | FR-6 (Workspace Enrollment) | Component identified; exact trust/consent/revocation mechanisms `TO BE DECIDED`; baseline behavior required by FR-6.6 through FR-6.9 |
| Context-based import filter | FR-7 (Context-Based Import Filtering) | Component identified; filtering criteria/thresholds `TO BE DECIDED`; must consider trust/classification/provenance/conflict |
| Trust registry | FR-6.6 through FR-6.8, FR-8.10, FR-10.6 | Component required; peer identity and revocation mechanism `TO BE DECIDED` |
| Announcement / peer-store gateway subsystem | FR-8 (Announcements and Inter-Store Information Transfer) | Component identified; announcement schema, replay defense, signing/identity mechanism, and import policy `TO BE DECIDED` |
| Web admin client + CLI (activity/audit views) | FR-9 (Operator Visibility and Control) | Component identified; operator interface type beyond "web + CLI" and deletion/tombstone behavior `TO BE DECIDED` |
| Authentication/session layer (passkey / OpenAuth) | Cross-cutting: FR-4.7, FR-10.1, FR-10.2 | Mechanism `GIVEN`; role/permission mapping `TO BE DECIDED` |
| Terraform-managed deployment | Non-functional: worldwide scale | Mechanism `GIVEN`; topology/provider `TO BE DECIDED` |
| Security architecture | FR-10 and SECURITY.md | Baseline defined; specific mechanisms remain `TO BE DECIDED` where marked |

## Dependency Rules

- The web admin client and CLI MUST depend only on the server's admin API surface; neither may access the knowledge graph store or crawl/enrollment/announcement subsystems directly.
- The MCP facade and admin API facade MUST both depend on a shared domain/service layer for graph, ingestion, filtering, enrollment, and announcement logic; protocol-specific code MUST NOT duplicate domain rules.
- Authentication MUST happen at the protocol facades, and authorization MUST be enforced through the shared identity/policy layer before graph reads, graph writes, imports, exports, enrollment, announcements, and state-changing MCP tools.
- The import safety pipeline MUST sit between any external or untrusted candidate source (ingestion, crawl, enrollment, announcement) and the knowledge graph store — no subsystem may write imported evidence/concepts/links directly into the graph without validation, classification, provenance tagging, relevance evaluation, and policy/review decision.
- Context-based filtering logic per FR-6.4/FR-7/FR-8.5 MUST be implemented once and reused by ingestion, crawl, enrollment, and announcement-driven import flows.
- The announcement subsystem MUST NOT trigger an automatic import into the knowledge graph store; it MAY only hand off a candidate to the enrollment/import filtering path, gated by import policy (FR-8.7).
- The peer-store gateway MUST check the trust registry before processing enrollment requests, received announcements, or transfer workflows; revoked or expired trust relationships MUST fail closed.
- The MCP facade MUST format graph content as untrusted data with provenance and MUST NOT allow returned evidence, concepts, announcements, or crawled content to alter tool definitions, prompts, or execution policy.
- The crawl subsystem MUST run with constrained network egress separate from the main server runtime and MUST be unable to reach private/internal/cloud-metadata addresses.
- The API boundary (admin API and MCP facades) is the sole point where domain logic (graph, ingestion, crawl, enrollment, announcement) may be reached from outside the server; this is also where SECURITY.md's authentication-enforcement rule applies, since it is the only place both auth schemes (passkey, OpenAuth) can be checked without leaking protocol details into domain code.
- Audit events MUST be emitted for every security-relevant decision and mutation before or atomically with the corresponding state change.
- No component may assume a specific storage, hosting, or messaging technology until those choices are made; domain logic MUST be written against abstractions so the `TO BE DECIDED` items above can be resolved without cascading rewrites.
