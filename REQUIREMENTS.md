# REQUIREMENTS.md

# Context Cartel Requirements

## Required Requirement Inputs

- `Project purpose: Context Cartel is a file-based knowledge store that represents evidence and concepts as a knowledge graph, exposes that knowledge to AI engines through an MCP server, updates/refines content through web crawling, and enables relevant information transfer between enrolled workspaces and knowledge stores.`
- `Primary users / actors: Knowledge store administrators/operators; workspace owners/operators; AI engines/AI agents consuming context through MCP; peer knowledge stores that publish and read announcements.`
- `Core workflows: Ingest file-based source material; identify evidence and concepts; link evidence and concepts in a knowledge graph; expose graph-backed context through MCP; crawl web sources to update/refine content; enroll other workspaces into an existing knowledge graph; filter imports for relevance based on context; publish announcements from one knowledge store to other stores; read announcements from other stores to support information transfer.`
- `Business objects / data entities: Knowledge Store; Knowledge Graph; File; Evidence; Concept; Graph Link/Relationship; Workspace; Workspace Enrollment; Context Filter; Import Candidate; Imported Knowledge; MCP Server Endpoint/Resource; Web Crawl Source; Crawled Content; Announcement; Announcement Reader; Information Transfer.`
- `External integrations: MCP-compatible AI engines/agents; file storage or file system sources; web crawling targets; other Context Cartel workspaces/knowledge stores participating in enrollment, announcements, and information transfer.`
- `Authentication / roles: Authentication mechanism is GIVEN in ARCHITECTURE.md (passkey for end users, OpenAuth for MCP/CLI clients). The exact authorization matrix is TO BE DECIDED, but the threat model requires deny-by-default server-side authorization for every actor, workspace, graph, source, data class, and operation before any public or multi-tenant deployment; tracked in SECURITY.md.`
- `Regulatory or privacy constraints: UNKNOWN. The project notes imply handling files, crawled content, evidence, concepts, workspace data, and inter-store announcements, but data classification, retention, consent, privacy, residency, and regulatory constraints MUST be resolved before any non-local deployment that processes third-party, personal, restricted, or regulated data; tracked in SECURITY.md.`

## Functional Requirements

### FR-1 Knowledge Store and Graph Model

- `FR-1.1` The system MUST maintain a named knowledge store called Context Cartel.
- `FR-1.2` The system MUST represent knowledge as a graph of concepts, evidence, and relationships.
- `FR-1.3` The system MUST store or reference source files as first-class inputs to the knowledge graph.
- `FR-1.4` The system MUST associate evidence with its originating file or source when that origin is known.
- `FR-1.5` The system MUST support links between evidence and concepts.
- `FR-1.6` The system SHOULD support links between related concepts where relationships are known.
- `FR-1.7` The system MUST NOT create graph links without preserving immutable provenance metadata sufficient to distinguish known, inferred, crawled, enrolled, announced, imported, and operator-approved relationships; exact metadata fields are TO BE DECIDED.
- `FR-1.8` The system MUST track source identity, source type, trust level, import path, timestamp, and decision rationale for every evidence item, concept, and graph link when that information exists.
- `FR-1.9` The system MUST NOT silently overwrite, merge, or promote graph content when provenance, trust level, data classification, freshness, or conflict status differs; conflict-resolution behavior is TO BE DECIDED.

### FR-2 File-Based Ingestion

- `FR-2.1` A knowledge store operator MUST be able to provide file-based source material for ingestion.
- `FR-2.2` The system MUST process ingested files to identify candidate evidence and concepts.
- `FR-2.3` The system MUST add accepted evidence and concepts from files into the knowledge graph.
- `FR-2.4` The system MUST preserve the relationship between each imported item and the file context it came from when available.
- `FR-2.5` The system SHOULD avoid duplicating identical or substantially equivalent evidence and concepts during ingestion; duplicate matching rules are TO BE DECIDED.
- `FR-2.6` The system MUST report ingestion results to the operator, including what was imported, skipped, or failed where known.
- `FR-2.7` The system MUST define and enforce allowed file types, maximum file sizes, maximum archive expansion, and parser handling rules before file ingestion is enabled outside local development; exact limits are TO BE DECIDED.
- `FR-2.8` The system MUST validate file paths, archive entries, and links against the configured ingestion boundary and MUST reject path traversal, symlink escape, zip-slip, archive bomb, and unsupported-content cases.
- `FR-2.9` The system MUST treat ingested file content as untrusted until validation, extraction, classification, provenance tagging, relevance evaluation, and policy checks have completed.
- `FR-2.10` The system MUST fail closed for unsupported, malformed, unsafe, or partially parsed files and MUST report the failure without importing partial evidence or concepts unless an explicit recovery policy allows it.

### FR-3 Evidence and Concept Linking

- `FR-3.1` The system MUST allow evidence to be linked to one or more concepts.
- `FR-3.2` The system MUST allow concepts to be linked to multiple supporting evidence items.
- `FR-3.3` The system SHOULD identify candidate relationships between concepts and evidence during ingestion or update workflows.
- `FR-3.4` The system MUST distinguish accepted links from candidate links when candidate review exists; review workflow details are TO BE DECIDED.
- `FR-3.5` The system MUST allow graph queries or retrieval flows to return relevant concepts with their supporting evidence when available.

### FR-4 MCP Server Exposure

- `FR-4.1` The system MUST expose an MCP server for AI engines or AI agents.
- `FR-4.2` MCP-compatible clients MUST be able to request graph-backed context from the knowledge store.
- `FR-4.3` MCP responses MUST include relevant evidence, concepts, and links when available for the requested context.
- `FR-4.4` MCP responses MUST preserve source attribution, provenance, trust level, data classification, and freshness/conflict status for returned evidence, concepts, and links when known.
- `FR-4.5` The system MUST define which MCP tools, resources, or prompts are exposed; exact MCP interface details are TO BE DECIDED.
- `FR-4.6` The system MUST handle unsupported, malformed, or unauthorized MCP requests with a clear failure response; exact failure semantics are TO BE DECIDED.
- `FR-4.7` The system MUST authorize every MCP tool, resource, prompt, graph retrieval, and state-changing operation server-side according to requester identity, client identity, workspace scope, graph scope, data classification, and tool scope.
- `FR-4.8` MCP tools that mutate state, trigger imports, enroll workspaces, publish announcements, delete/disable knowledge, or export data MUST require permissions distinct from read-only retrieval and MUST be disabled by default until explicitly enabled by policy.
- `FR-4.9` The MCP facade MUST treat graph content, crawled content, imported content, and announcements as untrusted data rather than instructions, tool definitions, or execution policy.
- `FR-4.10` MCP requests and responses MUST be subject to rate limits, query-cost limits, and response-size limits; exact limits are TO BE DECIDED.

### FR-5 Web Crawling and Content Refresh

- `FR-5.1` A knowledge store operator MUST be able to configure web crawling targets or sources; configuration format is TO BE DECIDED.
- `FR-5.2` The system MUST crawl configured web sources to discover candidate content for the knowledge store.
- `FR-5.3` The system MUST evaluate crawled content for relevance before importing it into the knowledge graph.
- `FR-5.4` The system MUST link imported crawled content to its originating web source when available.
- `FR-5.5` The system SHOULD use crawled content to update or refine existing concepts and evidence when relevant.
- `FR-5.6` The system MUST report crawl results, including discovered, imported, skipped, and failed items where known.
- `FR-5.7` Crawl scheduling, crawl frequency, robots handling, source allow/deny rules, and crawl-source approval ownership MUST be defined before the crawler is enabled outside local development; exact values are TO BE DECIDED.
- `FR-5.8` The crawler MUST validate crawl targets before each request and after each redirect, and MUST block private/internal address ranges, localhost, link-local addresses, cloud metadata endpoints, and DNS-rebinding attempts.
- `FR-5.9` The crawler MUST enforce maximum crawl depth, redirect count, response size, runtime, frequency, and content type limits; exact limits are TO BE DECIDED.
- `FR-5.10` Crawled content MUST remain candidate or quarantined content until source validation, content validation, classification, provenance tagging, relevance evaluation, and import policy checks have completed.
- `FR-5.11` Crawl targets SHOULD use HTTPS; plaintext HTTP crawling MAY be allowed only by explicit operator policy and MUST mark resulting content as originating from an insecure transport.

### FR-6 Workspace Enrollment

- `FR-6.1` A workspace owner/operator MUST be able to enroll another workspace into an existing knowledge graph.
- `FR-6.2` Enrollment MUST identify the source workspace and target knowledge graph.
- `FR-6.3` The system MUST support importing relevant knowledge from an enrolled workspace into the target knowledge graph.
- `FR-6.4` The system MUST apply context-based filtering before importing workspace knowledge.
- `FR-6.5` The system MUST report enrollment/import results, including imported, skipped, or failed items where known.
- `FR-6.6` Workspace enrollment MUST require explicit, auditable, and revocable trust establishment and consent between the source workspace owner/operator and the target knowledge graph operator before import.
- `FR-6.7` Workspace enrollment MUST be scoped to allowed source workspaces, target graphs, data classifications, import categories, and operations; exact scope semantics are TO BE DECIDED.
- `FR-6.8` Revoking a workspace enrollment MUST stop future imports and access mediated by that enrollment; behavior for already imported knowledge, tombstones, and provenance preservation is TO BE DECIDED.
- `FR-6.9` The system MUST audit enrollment creation, consent, scope changes, import decisions, failures, and revocation.

### FR-7 Context-Based Import Filtering

- `FR-7.1` The system MUST evaluate candidate imports against the target knowledge graph context before import.
- `FR-7.2` The system MUST import only candidate knowledge that is relevant according to configured or inferred context filters.
- `FR-7.3` The system MUST record why candidate knowledge was imported or skipped when that explanation is available.
- `FR-7.4` Operators SHOULD be able to review filtered import decisions; review and override behavior is TO BE DECIDED.
- `FR-7.5` Filtering criteria, relevance thresholds, and context matching semantics are TO BE DECIDED.
- `FR-7.6` Context filtering MUST consider source trust, data classification, provenance, freshness, duplicate status, conflict status, and import policy in addition to relevance before importing candidate knowledge.
- `FR-7.7` The system MUST NOT automatically accept external candidate knowledge into the committed graph unless a configured policy explicitly permits automatic acceptance for the candidate's source, trust level, data class, and operation.
- `FR-7.8` Operator overrides of filtering decisions MUST be authorized, audited, and recorded with rationale.

### FR-8 Announcements and Inter-Store Information Transfer

- `FR-8.1` Each knowledge store MUST be able to publish an announcement describing available data for potential information transfer.
- `FR-8.2` An announcement MUST identify the publishing knowledge store.
- `FR-8.3` An announcement MUST describe the announced data sufficiently for another store to evaluate relevance; exact announcement schema is TO BE DECIDED.
- `FR-8.4` Each knowledge store MUST be able to read announcements from other knowledge stores.
- `FR-8.5` The system MUST evaluate received announcements for relevance to the receiving knowledge store context.
- `FR-8.6` The system MAY use relevant announcements to initiate a candidate import or information-transfer workflow.
- `FR-8.7` The system MUST NOT import announced data automatically unless the configured import policy allows it; import policy behavior is TO BE DECIDED.
- `FR-8.8` The system MUST record received announcements and any resulting transfer decisions where known.
- `FR-8.9` Announcements MUST be metadata-only by default and MUST NOT include source file content, personal data, secrets, credentials, or regulated data unless an explicit export policy authorizes that disclosure.
- `FR-8.10` Received announcements MUST be authenticated, integrity protected, replay protected, schema validated, scoped, rate limited, and checked against a revocable trust relationship before processing.
- `FR-8.11` Announcements MUST carry expiration or freshness metadata, size bounds, source identity, data classification, and trust labels; exact schema fields are TO BE DECIDED.
- `FR-8.12` The system MUST audit announcement publication, receipt, validation, policy decisions, transfer initiation, skips, failures, and revocation effects.

### FR-9 Operator Visibility and Control

- `FR-9.1` Operators MUST be able to see the current knowledge store contents at the level of files, evidence, concepts, and links.
- `FR-9.2` Operators MUST be able to see recent ingestion, crawl, enrollment, filtering, announcement, and transfer activity.
- `FR-9.3` Operators SHOULD be able to inspect why a concept, evidence item, link, or imported item exists in the graph.
- `FR-9.4` Operators SHOULD be able to remove or disable imported knowledge; exact deletion, tombstone, and provenance behavior is TO BE DECIDED.
- `FR-9.5` Operator interface type is web admin (React) and CLI, per ARCHITECTURE.md's component boundaries and DESIGN.md's Platform targets; detailed screens, commands, and workflows within each are TO BE DECIDED (see DESIGN.md Open Questions).
- `FR-9.6` The system MUST maintain a structured audit log for authentication, authorization decisions, ingestion, graph mutations, crawl activity, enrollment, filtering, announcement, transfer, deletion/disablement, export, and MCP access.
- `FR-9.7` Audit events MUST record actor identity, client identity, source workspace/store, action, object identifiers, policy decision, data classification when known, correlation ID, and timestamp; exact event schema is TO BE DECIDED.
- `FR-9.8` Audit logs MUST be append-only or tamper-evident, access-controlled, and retained according to a defined policy; retention duration is TO BE DECIDED.
- `FR-9.9` Web admin and CLI surfaces MUST render untrusted file, evidence, crawled, imported, and announcement content as data only, with encoding or sanitization appropriate to the output medium.

### FR-10 Security, Privacy, and Trust Controls

- `FR-10.1` The system MUST define and enforce a deny-by-default role and permission model for knowledge store operators, workspace owners/operators, MCP/CLI clients, AI agents, and peer knowledge stores.
- `FR-10.2` The system MUST authorize every REST, MCP, CLI, ingestion, crawl, enrollment, announcement, import, export, graph retrieval, mutation, and deletion/disablement action server-side.
- `FR-10.3` The system MUST enforce workspace, knowledge-store, graph, and tenant isolation wherever multiple stores, workspaces, or clients share a deployment.
- `FR-10.4` The system MUST classify files, evidence, concepts, graph links, workspace data, crawled content, announcements, audit logs, exports, and MCP responses before any non-local deployment.
- `FR-10.5` The system MUST NOT expose restricted, private, secret, credential, personal, or regulated data through MCP, REST, CLI, web admin, announcements, logs, or exports unless the requester is authorized for that data class and source scope.
- `FR-10.6` The system MUST require explicit, revocable trust establishment and consent before accepting workspace enrollment, peer-store announcements, or inter-store transfer.
- `FR-10.7` The system MUST define retention, deletion/disablement, tombstone, and provenance-preservation behavior for imported knowledge, audit records, and personal or regulated data before processing that data outside local development.
- `FR-10.8` The system MUST enforce abuse limits for file size, archive expansion, crawl depth/frequency, redirect chains, announcement volume, graph query cost, MCP request rate, API request rate, and response size.
- `FR-10.9` The system MUST NOT store secrets, credentials, passkey material, OpenAuth client secrets, crawl credentials, or Terraform secrets in source files, committed configuration, graph content, announcements, MCP responses, or logs.
- `FR-10.10` Public, multi-tenant, or production deployment MUST NOT proceed until the authorization model, inter-store trust model, data classification policy, retention/deletion policy, MCP authorization behavior, crawl-source policy, audit-log policy, secret-management mechanism, and hosting/data-residency choices are resolved.

## Open Questions

- What file types must be supported for ingestion in the first version?
- Where are files stored or referenced from, and are they copied into Context Cartel or indexed in place?
- What is the exact schema for concepts, evidence, graph links, files, workspaces, announcements, and imports?
- Are graph links created automatically, manually, or through an operator review workflow?
- What does “fine tune content” mean in product terms: update graph content, refine extracted concepts, tune retrieval behavior, or tune AI models?
- What MCP tools, resources, prompts, and response schemas must the server expose?
- Which AI engines/agents are target MCP clients?
- Can announcements contain actual data, metadata only, summaries only, or retrieval pointers?
- What policy determines whether announced data is automatically imported, queued for review, or ignored?
- How are relevance filters configured, and what threshold is required before importing content?
- How should the system explain why content was imported or skipped?
- How are conflicts, duplicates, stale data, and contradictory evidence handled?
- What web sources may be crawled, and what crawl frequency, robots, consent, and allow/deny rules apply?
- What audit history is required for ingestion, graph changes, imports, announcements, and MCP access?

Security-, trust-, and privacy-related open questions (authorization/role model, inter-store trust, data retention and regulatory scope) are owned by and tracked in SECURITY.md Open Questions, and FR-10.10 treats them as release gates rather than optional follow-up work.
