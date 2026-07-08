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
- `API style and integration model: REST or MCP`
- `Authentication and session model: User: passkey, MCP/CLI: OpenAuth`
- `Data model expectations: Core entities per REQUIREMENTS.md: Knowledge Store, Knowledge Graph, File, Evidence, Concept, Graph Link/Relationship (with provenance metadata distinguishing known/inferred/imported — FR-1.7), Workspace, Workspace Enrollment, Context Filter, Import Candidate, Imported Knowledge, MCP Server Endpoint/Resource, Web Crawl Source, Crawled Content, Announcement, Announcement Reader, Information Transfer. Exact field-level schemas are TO BE DECIDED (REQUIREMENTS.md Open Questions).`
- `Deployment model: terraform`
- `Scale expectations: worldwide`
- `Security expectations: TO BE DECIDED, we will focus on that next`

## Initial Architecture (Provisional)

**Assumption labels used below:** `[ASSUMPTION]` = not stated in REQUIREMENTS.md/DESIGN.md, inferred to make the architecture coherent; `[GIVEN]` = taken directly from explicit human-provided inputs above; `[TBD]` = open gap, intentionally not resolved.

### Component boundaries

1. **Web admin client (React)** `[GIVEN: Client framework]`
   - Serves the operator-facing surfaces DESIGN.md lists as component needs: knowledge graph explorer, ingestion panel, crawl source manager, workspace enrollment manager, context filter review queue, announcement feed, activity/audit log.
   - Must provide a non-visual (text/table) equivalent of the graph explorer per DESIGN.md Accessibility section (FR-9.1/FR-9.3) — `[ASSUMPTION]` this is a rendering concern inside the same client, not a separate service.
   - Consumes the backend via REST or MCP `[GIVEN: API style]` — which of the two (or both, for different consumers) fronts the web admin specifically is `[TBD]`.

2. **Server (Node.js)** `[GIVEN: Server framework]`
   - Hosts the knowledge store/graph domain logic (FR-1), ingestion pipeline (FR-2), linking (FR-3), crawl subsystem (FR-5), enrollment (FR-6), context-based filtering (FR-7), and announcements (FR-8).
   - Exposes two integration surfaces per `[GIVEN: API style]`:
     - A REST (or equivalent) API for the web admin client and any CLI.
     - An MCP server endpoint for AI engines/agents (FR-4). `[ASSUMPTION]` these are two protocol facades over one shared domain/service layer, not two independently-implemented backends — REQUIREMENTS.md treats "expose graph-backed context through MCP" as one of several ways to reach the same knowledge graph.
   - Exact MCP tools/resources/prompts exposed (FR-4.5) are `[TBD]` per REQUIREMENTS.md Open Questions.

3. **CLI** — DESIGN.md's Platform targets name a CLI alongside MCP server and web; REQUIREMENTS.md's operator-visibility requirements (FR-9) don't specify interface type (FR-9.5 is UNKNOWN). `[ASSUMPTION]` the CLI is a thin client over the same REST/MCP surface as the web admin, not a separate implementation of domain logic.

4. **Ingestion subsystem** (FR-2) — accepts operator-supplied files, extracts candidate evidence/concepts, reports import/skip/fail results. Where files are stored/referenced from is `[TBD]` (REQUIREMENTS.md Open Question).

5. **Web crawl subsystem** (FR-5) — configurable crawl targets, discovers and evaluates candidate content, links imports to source, reports results. Crawl scheduling/frequency/robots/allow-deny rules are `[TBD]`.

6. **Context-based import filter** (FR-7) — a shared evaluation step used by both workspace enrollment (FR-6.4) and announcement-driven transfer (FR-8.5), since both require "evaluate candidate imports/announcements against the target knowledge graph context before import." `[ASSUMPTION]` this is implemented as one reusable filtering component rather than duplicated per-caller, because REQUIREMENTS.md describes the same relevance-evaluation behavior in both FR-6 and FR-7/FR-8. Filtering criteria and thresholds are `[TBD]`.

7. **Workspace enrollment subsystem** (FR-6) — enrolls a source workspace into a target knowledge graph, applies the context filter, reports results. Trust/consent/access-control/revocation are `[TBD]` (FR-6.6, UNKNOWN).

8. **Announcement subsystem** (FR-8) — publishes outgoing announcements identifying the store and describing available data; reads and evaluates incoming announcements from peer stores; may hand off to the enrollment/import path when policy allows (FR-8.6/FR-8.7). Announcement schema and inter-store trust model are `[TBD]`.

9. **Knowledge graph store** — the persistent representation of Files, Evidence, Concepts, and Links with provenance metadata (FR-1.7). No specific data store technology is chosen here, per instructions not to select one without an explicit basis in REQUIREMENTS.md or human input — this remains `[TBD]`.

### Cross-cutting, explicitly deferred

- **Authentication/session model** is `[GIVEN]`: end users authenticate via passkey; MCP and CLI clients authenticate via OpenAuth. How these two schemes map onto authorization/roles for the actors in REQUIREMENTS.md (operator vs. workspace owner vs. peer store) is `[TBD]` — REQUIREMENTS.md marks authentication/roles as UNKNOWN beyond naming the actors.
- **Security expectations** beyond the authentication model above are explicitly `[TBD]` per the human-provided input ("TO BE DECIDED, we will focus on that next").
- **Deployment model** is `[GIVEN]` as Terraform-managed infrastructure; the specific cloud provider(s), regions, and topology needed to satisfy `[GIVEN: worldwide scale]` are `[TBD]`.
- **Data classification, retention, privacy/regulatory constraints** are `[TBD]` (REQUIREMENTS.md: UNKNOWN).

### What this architecture deliberately does not choose yet

Per instructions, the following are left open rather than guessed: specific database/storage technology, hosting provider, message queue or job-scheduling technology for crawling, MCP transport/framework specifics, UI component library, and CI/CD tooling.

## Requirement Traceability

| Architecture component | Requirement group(s) | Status |
|---|---|---|
| Knowledge graph store | FR-1 (Knowledge Store and Graph Model) | Component identified; schema `TO BE DECIDED` |
| Ingestion subsystem | FR-2 (File-Based Ingestion) | Component identified; file storage location `TO BE DECIDED` |
| Linking logic (shared with graph store) | FR-3 (Evidence and Concept Linking) | Component identified; candidate-vs-accepted link review workflow `TO BE DECIDED` |
| MCP server facade | FR-4 (MCP Server Exposure) | Component identified; tool/resource/prompt surface `TO BE DECIDED`; authorization behavior `TO BE DECIDED` |
| Web crawl subsystem | FR-5 (Web Crawling and Content Refresh) | Component identified; scheduling/robots/allow-deny rules `TO BE DECIDED` |
| Workspace enrollment subsystem | FR-6 (Workspace Enrollment) | Component identified; trust/consent/revocation `TO BE DECIDED` |
| Context-based import filter | FR-7 (Context-Based Import Filtering) | Component identified; filtering criteria/thresholds `TO BE DECIDED` |
| Announcement subsystem | FR-8 (Announcements and Inter-Store Information Transfer) | Component identified; announcement schema and import policy `TO BE DECIDED` |
| Web admin client + CLI (activity/audit views) | FR-9 (Operator Visibility and Control) | Component identified; operator interface type beyond "web + CLI" and deletion/tombstone behavior `TO BE DECIDED` |
| Authentication/session layer (passkey / OpenAuth) | Cross-cutting: FR-4.6, FR-6.6 authorization notes | Mechanism `GIVEN`; role/permission mapping `TO BE DECIDED` |
| Terraform-managed deployment | Non-functional: worldwide scale | Mechanism `GIVEN`; topology/provider `TO BE DECIDED` |
| Security architecture | Non-functional | Explicitly deferred — `TO BE DECIDED` |

## Dependency Rules

- The web admin client and CLI MUST depend only on the server's REST/MCP-facing API surface; neither may access the knowledge graph store or crawl/enrollment/announcement subsystems directly.
- The MCP facade and the REST facade MUST both depend on a shared domain/service layer for graph, ingestion, filtering, enrollment, and announcement logic; MCP-specific and REST-specific code MUST NOT duplicate domain rules (in particular, context-based filtering logic per FR-6.4/FR-7/FR-8.5 MUST be implemented once and reused by both enrollment and announcement-driven import flows).
- The context-based import filter MUST sit between any external candidate source (ingestion, crawl, enrollment, announcement) and the knowledge graph store — no subsystem may write imported evidence/concepts/links directly into the graph without passing through relevance evaluation, per FR-7.1.
- The announcement subsystem MUST NOT trigger an automatic import into the knowledge graph store; it MAY only hand off a candidate to the enrollment/import filtering path, gated by import policy (FR-8.7).
- Authentication MUST be enforced at the API boundary (REST and MCP facades), not deep inside domain logic, so that the two distinct schemes (passkey for users, OpenAuth for MCP/CLI) can be swapped or extended without touching graph/ingestion/crawl/enrollment/announcement logic.
- No component may assume a specific storage, hosting, or messaging technology until those choices are made; domain logic MUST be written against abstractions so the `TO BE DECIDED` items above can be resolved without cascading rewrites.
