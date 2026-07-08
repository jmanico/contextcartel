# CLAUDE.md

@REQUIREMENTS.md
@ARCHITECTURE.md
@SECURITY.md
@DESIGN.md

## Project

Context Cartel: a file-based knowledge store that models evidence/concepts as a
knowledge graph, exposes it to AI engines via MCP, refines it via web crawling,
and transfers relevant knowledge between enrolled workspaces and peer stores.
Web + API application. See REQUIREMENTS.md for full scope, actors, and FRs.

## Repo state

Bootstrap / pre-code. Only requirement, architecture, security, and design docs
exist — no source tree, package manifest, build, lint, or test tooling yet.

- Build/test/lint commands: `NOT YET DEFINED` — do not invent npm scripts,
  test runners, or CI config. Propose them only when asked to scaffold the
  project, and update this file once they exist.
- Directory/module layout: `NOT YET DEFINED` — follow ARCHITECTURE.md's
  component boundaries (domain/service layer, REST facade, MCP facade,
  ingestion/crawl/filter/enrollment/announcement subsystems) once code exists.
- Branching, commit, and PR conventions: `NOT YET DEFINED` beyond standard
  git hygiene (small, reviewable commits; no direct pushes to `main` without
  asking first).
- Storage, hosting, and MCP transport specifics: intentionally unchosen — see
  ARCHITECTURE.md "What this architecture deliberately does not choose yet".

## Working style

- Bootstrap mode: this repo defines *what* to build and *how it must behave*,
  not *how it's coded*. Don't scaffold source code, pick libraries, or resolve
  an open `TO BE DECIDED`/`UNKNOWN` item unless explicitly asked — flag the gap
  instead of guessing.
- Every requirement (in docs or as a GitHub issue) MUST use RFC 2119 keywords
  (MUST/MUST NOT/SHOULD/MAY), matching REQUIREMENTS.md's existing FR style.
- When a new requirement, component, or decision is introduced, cross-check it
  against REQUIREMENTS.md (FR coverage), ARCHITECTURE.md (component/dependency
  rules), SECURITY.md (provisional security rules), and DESIGN.md (component
  needs) — keep all four internally consistent rather than adding a one-off
  answer in only one file.
- Do not add dependencies, choose a datastore/hosting provider, or write
  production code against an unresolved `TO BE DECIDED` item without calling
  it out first.

## GitHub issues

Every new issue MUST be written as a structured requirement using
REQUIREMENT_TEMPLATE.md (all sections filled in or explicitly marked
`UNKNOWN`/`TO BE DECIDED`) — no free-form feature requests or bug reports
without this structure. Each issue must be independently testable per its
Acceptance Criteria.
