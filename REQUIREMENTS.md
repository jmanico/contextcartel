# REQUIREMENTS.md

# Context Cartel Requirements

## Required Requirement Inputs

- `Project purpose: Context Cartel is a file-based knowledge store that represents evidence and concepts as a knowledge graph, exposes that knowledge to AI engines through an MCP server, updates/refines content through web crawling, and enables relevant information transfer between enrolled workspaces and knowledge stores.`
- `Primary users / actors: Knowledge store administrators/operators; workspace owners/operators; AI engines/AI agents consuming context through MCP; peer knowledge stores that publish and read announcements.`
- `Core workflows: Ingest file-based source material; identify evidence and concepts; link evidence and concepts in a knowledge graph; expose graph-backed context through MCP; crawl web sources to update/refine content; enroll other workspaces into an existing knowledge graph; filter imports for relevance based on context; publish announcements from one knowledge store to other stores; read announcements from other stores to support information transfer.`
- `Business objects / data entities: Knowledge Store; Knowledge Graph; File; Evidence; Concept; Graph Link/Relationship; Workspace; Workspace Enrollment; Context Filter; Import Candidate; Imported Knowledge; MCP Server Endpoint/Resource; Web Crawl Source; Crawled Content; Announcement; Announcement Reader; Information Transfer.`
- `External integrations: MCP-compatible AI engines/agents; file storage or file system sources; web crawling targets; other Context Cartel workspaces/knowledge stores participating in enrollment, announcements, and information transfer.`
- `Authentication / roles: UNKNOWN. The project notes identify administrators/operators, workspace owners/operators, AI engines/agents, and peer knowledge stores as actors, but authentication, authorization, trust model, and role permissions are TO BE DECIDED.`
- `Regulatory or privacy constraints: UNKNOWN. The project notes imply handling files, crawled content, evidence, concepts, workspace data, and inter-store announcements, but data classification, retention, consent, privacy, and regulatory constraints are TO BE DECIDED.`

## Functional Requirements

### FR-1 Knowledge Store and Graph Model

- `FR-1.1` The system MUST maintain a named knowledge store called Context Cartel.
- `FR-1.2` The system MUST represent knowledge as a graph of concepts, evidence, and relationships.
- `FR-1.3` The system MUST store or reference source files as first-class inputs to the knowledge graph.
- `FR-1.4` The system MUST associate evidence with its originating file or source when that origin is known.
- `FR-1.5` The system MUST support links between evidence and concepts.
- `FR-1.6` The system SHOULD support links between related concepts where relationships are known.
- `FR-1.7` The system MUST NOT create graph links without preserving enough metadata to distinguish known, inferred, or imported relationships; exact metadata fields are TO BE DECIDED.

### FR-2 File-Based Ingestion

- `FR-2.1` A knowledge store operator MUST be able to provide file-based source material for ingestion.
- `FR-2.2` The system MUST process ingested files to identify candidate evidence and concepts.
- `FR-2.3` The system MUST add accepted evidence and concepts from files into the knowledge graph.
- `FR-2.4` The system MUST preserve the relationship between each imported item and the file context it came from when available.
- `FR-2.5` The system SHOULD avoid duplicating identical or substantially equivalent evidence and concepts during ingestion; duplicate matching rules are TO BE DECIDED.
- `FR-2.6` The system MUST report ingestion results to the operator, including what was imported, skipped, or failed where known.

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
- `FR-4.4` MCP responses SHOULD preserve source attribution for returned evidence when known.
- `FR-4.5` The system MUST define which MCP tools, resources, or prompts are exposed; exact MCP interface details are TO BE DECIDED.
- `FR-4.6` The system MUST handle unsupported, malformed, or unauthorized MCP requests with a clear failure response; authorization behavior is UNKNOWN.

### FR-5 Web Crawling and Content Refresh

- `FR-5.1` A knowledge store operator MUST be able to configure web crawling targets or sources; configuration format is TO BE DECIDED.
- `FR-5.2` The system MUST crawl configured web sources to discover candidate content for the knowledge store.
- `FR-5.3` The system MUST evaluate crawled content for relevance before importing it into the knowledge graph.
- `FR-5.4` The system MUST link imported crawled content to its originating web source when available.
- `FR-5.5` The system SHOULD use crawled content to update or refine existing concepts and evidence when relevant.
- `FR-5.6` The system MUST report crawl results, including discovered, imported, skipped, and failed items where known.
- `FR-5.7` Crawl scheduling, crawl frequency, robots handling, and source allow/deny rules are TO BE DECIDED.

### FR-6 Workspace Enrollment

- `FR-6.1` A workspace owner/operator MUST be able to enroll another workspace into an existing knowledge graph.
- `FR-6.2` Enrollment MUST identify the source workspace and target knowledge graph.
- `FR-6.3` The system MUST support importing relevant knowledge from an enrolled workspace into the target knowledge graph.
- `FR-6.4` The system MUST apply context-based filtering before importing workspace knowledge.
- `FR-6.5` The system MUST report enrollment/import results, including imported, skipped, or failed items where known.
- `FR-6.6` Trust, consent, access control, and revocation behavior for workspace enrollment are UNKNOWN and TO BE DECIDED.

### FR-7 Context-Based Import Filtering

- `FR-7.1` The system MUST evaluate candidate imports against the target knowledge graph context before import.
- `FR-7.2` The system MUST import only candidate knowledge that is relevant according to configured or inferred context filters.
- `FR-7.3` The system MUST record why candidate knowledge was imported or skipped when that explanation is available.
- `FR-7.4` Operators SHOULD be able to review filtered import decisions; review and override behavior is TO BE DECIDED.
- `FR-7.5` Filtering criteria, relevance thresholds, and context matching semantics are TO BE DECIDED.

### FR-8 Announcements and Inter-Store Information Transfer

- `FR-8.1` Each knowledge store MUST be able to publish an announcement describing available data for potential information transfer.
- `FR-8.2` An announcement MUST identify the publishing knowledge store.
- `FR-8.3` An announcement MUST describe the announced data sufficiently for another store to evaluate relevance; exact announcement schema is TO BE DECIDED.
- `FR-8.4` Each knowledge store MUST be able to read announcements from other knowledge stores.
- `FR-8.5` The system MUST evaluate received announcements for relevance to the receiving knowledge store context.
- `FR-8.6` The system MAY use relevant announcements to initiate a candidate import or information-transfer workflow.
- `FR-8.7` The system MUST NOT import announced data automatically unless the configured import policy allows it; import policy behavior is TO BE DECIDED.
- `FR-8.8` The system MUST record received announcements and any resulting transfer decisions where known.

### FR-9 Operator Visibility and Control

- `FR-9.1` Operators MUST be able to see the current knowledge store contents at the level of files, evidence, concepts, and links.
- `FR-9.2` Operators MUST be able to see recent ingestion, crawl, enrollment, filtering, announcement, and transfer activity.
- `FR-9.3` Operators SHOULD be able to inspect why a concept, evidence item, link, or imported item exists in the graph.
- `FR-9.4` Operators SHOULD be able to remove or disable imported knowledge; exact deletion, tombstone, and provenance behavior is TO BE DECIDED.
- `FR-9.5` Operator interface type is UNKNOWN.

## Open Questions

- What file types must be supported for ingestion in the first version?
- Where are files stored or referenced from, and are they copied into Context Cartel or indexed in place?
- What is the exact schema for concepts, evidence, graph links, files, workspaces, announcements, and imports?
- Are graph links created automatically, manually, or through an operator review workflow?
- What does “fine tune content” mean in product terms: update graph content, refine extracted concepts, tune retrieval behavior, or tune AI models?
- What MCP tools, resources, prompts, and response schemas must the server expose?
- Which AI engines/agents are target MCP clients?
- What authentication and authorization model is required for operators, MCP clients, workspaces, and peer knowledge stores?
- How is trust established between knowledge stores that exchange announcements or data?
- Can announcements contain actual data, metadata only, summaries only, or retrieval pointers?
- What policy determines whether announced data is automatically imported, queued for review, or ignored?
- How are relevance filters configured, and what threshold is required before importing content?
- How should the system explain why content was imported or skipped?
- How are conflicts, duplicates, stale data, and contradictory evidence handled?
- What web sources may be crawled, and what crawl frequency, robots, consent, and allow/deny rules apply?
- What privacy, retention, deletion, and regulatory obligations apply to files, crawled content, workspace data, and announcements?
- What audit history is required for ingestion, graph changes, imports, announcements, and MCP access?
- What operator interface is required for browsing, reviewing, editing, deleting, and approving graph content?
