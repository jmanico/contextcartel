# DESIGN.md

## Required Design Inputs

- `Brand personality: TO BE DECIDED. REQUIREMENTS.md defines actors and workflows but no tone, voice, or personality. logo.svg and the palette below are a neutral, functional starting point (structured, technical, trustworthy) — not a confirmed brand direction.`
- `Primary audience: derived from REQUIREMENTS.md "Primary users / actors" — knowledge store administrators/operators and workspace owners/operators (human, technical, operate the store day-to-day); AI engines/agents consuming context via MCP (machine clients, not visual-UI consumers); peer knowledge stores exchanging announcements (machine-to-machine).`
- `Platform targets: MCP server, CLI, web for admin.`
- `Existing brand assets: TO BE DECIDED. No brand assets predate this repository; logo.svg (added alongside this document) is the first visual asset.`

## Brand and Logo

- `logo.svg` (repo root) is the primary mark: a small knowledge graph (five colored nodes on a partial ring, evoking a "C") connected to a central file/document icon, on a dark rounded-square badge.
- Rationale, tied directly to REQUIREMENTS.md: the ring of linked nodes represents the **knowledge graph** (FR-1); the central document represents **file-based ingestion** as the graph's source (FR-2); the ring's open side reads as a "C" for Context Cartel.
- Single mark, no wordmark: the SVG contains no text/type, so it needs no font license and stays legible at favicon size. Pairing with a wordmark ("Context Cartel" set in a system font) is TO BE DECIDED for header use.
- Verified legible at both a large header size (200px) and a small favicon size (32px and 16px) — the badge, node colors, and document shape all remain distinct at 16px.
- Minimum size: do not render below ~24px; below that the individual node colors stop being distinguishable.
- Clear space: keep at least one node-radius of empty margin around the badge on all sides.
- Do not stretch, recolor per-instance, rotate, or add a drop shadow. A monochrome (single-color) variant for constrained contexts (e.g. print, watermarks) is TO BE DECIDED.

## Color Palette

Palette is currently defined only by `logo.svg`; it has not been extended into a full UI design-token set.

| Role | Hex | Used for |
|---|---|---|
| Badge background | `#0F1629` | Logo badge fill |
| Badge inner border | `#26314A` | Subtle inset ring in the logo |
| Graph edge | `#3A465F` | Lines connecting nodes/file in the logo |
| Document | `#F3ECDD` | File/source icon at the graph's center |
| Document fold | `#D8CBAA` | Folded-corner detail on the document |
| Accent — amber | `#F5A623` | Graph node |
| Accent — blue | `#4CC2F1` | Graph node |
| Accent — purple | `#7C5CFC` | Graph node |
| Accent — green | `#34C795` | Graph node |

- Whether the four accent colors carry consistent semantic meaning in the product UI (e.g., a fixed color per entity type: Evidence, Concept, Workspace, Announcement) is **TO BE DECIDED** — REQUIREMENTS.md does not define entity-to-color mapping, and inventing one here would be a UI decision, not a brand one.
- Light/dark mode support and any additional UI tokens (surface, text, border, success/error/warning states) are **TO BE DECIDED**.

## Typography

- The logo itself contains no type (by design — avoids font licensing and keeps favicon rendering crisp).
- No brand typeface is defined (`Existing brand assets: TO BE DECIDED`).
- Where UI text is needed, default to the platform's native system font stack rather than inventing a brand typeface:
  - Web admin: system-ui font stack (e.g. `system-ui, -apple-system, "Segoe UI", sans-serif`).
  - CLI: the user's terminal font (no font choice applies).
- A full type scale (sizes, weights, line-heights) is **TO BE DECIDED** and out of scope until a UI framework/direction is chosen.

## Layout and Spacing

- No UI framework is chosen here (out of scope per instructions); spacing guidance below is framework-agnostic.
- Suggested base unit: 8px grid for the web admin surface, consistent with the logo's internal proportions (built on a 200-unit grid with ~14-unit node radii and ~28-unit spacing bands).
- Platform-specific notes from `Platform targets`:
  - **Web admin**: needs a responsive layout (REQUIREMENTS.md doesn't specify device targets — assume desktop-first, responsive down to tablet, TO BE DECIDED for mobile).
  - **CLI**: "layout" means consistent column alignment, indentation, and predictable output structure across commands (ingest, crawl, enroll, announce — per FR-2, FR-5, FR-6, FR-8), not visual spacing.
  - **MCP server**: has no visual layout; its equivalent of "layout" is consistent, predictable structure in tool/resource/prompt schemas and naming (FR-4.5), so AI clients can reliably parse responses.
- Concrete grid, breakpoints, and container widths are **TO BE DECIDED**.

## Components

Derived from REQUIREMENTS.md workflows and FR sections — listed as component *needs*, not designed:

- **Knowledge graph explorer** — browse files, evidence, concepts, and links (FR-9.1); inspect provenance of a given node (FR-9.3).
- **Ingestion panel** — submit file-based source material and view import/skip/fail results (FR-2.1, FR-2.6).
- **Crawl source manager** — configure web crawl targets and review crawl results (FR-5.1, FR-5.6).
- **Workspace enrollment manager** — enroll a workspace, review import/filter results (FR-6.1, FR-6.5).
- **Context filter review queue** — review relevance-filtered import decisions (FR-7.4).
- **Announcement feed** — publish outgoing announcements and read incoming ones (FR-8.1, FR-8.4).
- **Activity/audit log** — recent ingestion, crawl, enrollment, filtering, announcement, and transfer activity (FR-9.2).
- **CLI equivalents** of each panel above, as commands/subcommands.
- **MCP surface** — not a visual component; its "component" is the set of exposed tools/resources/prompts (FR-4.5), still TO BE DECIDED.

No visual designs, wireframes, or a UI framework are specified here — this is a checklist grounded in required workflows, for whoever designs the actual screens/commands next.

## Accessibility

- Logo contrast: document-on-badge (`#F3ECDD` on `#0F1629`) measures ~15:1, comfortably exceeding WCAG AAA (7:1) for any adjacent text.
- Do not rely on node color alone to convey meaning anywhere in the product UI (e.g., entity type, status): the four accent colors are not guaranteed distinguishable for all forms of color vision deficiency. Pair color with a label, icon, or pattern wherever color becomes meaningful (not just decorative, as it is in the logo itself).
- Graph/visual explorer views need a non-visual equivalent (text list, table, or screen-reader-accessible description of nodes and relationships) to satisfy FR-9.1/FR-9.3 for users who can't consume a visual graph.
- CLI output should respect the `NO_COLOR` convention and remain fully meaningful with color stripped (no color-only signaling of success/failure/relevance).
- Web admin: standard keyboard-navigability and screen-reader labeling expectations apply; no specific WCAG conformance level is set — **TO BE DECIDED**.

## Open Questions

- Should the four accent colors carry a fixed, consistent meaning across the product (e.g., one color per entity type), or are they decorative only in the logo?
- Is a wordmark ("Context Cartel" in type) needed alongside the mark for the web admin header, and if so, in what typeface?
- Is dark mode, light mode, or both required for the web admin surface?
- What WCAG conformance level (A/AA/AAA) is the target for the web admin UI?
- Is there a parent organization or existing brand system this needs to align with, or is Context Cartel's brand being defined from scratch?
- What is the primary audience's technical sophistication for the web admin (affects information density and how much of the graph/provenance detail to surface by default)?
- Are there mobile/tablet requirements for the web admin, or is it desktop-only?
- Should the MCP tool/resource naming and description style follow a documented style guide, and who owns that decision?
- Any localization/internationalization requirements for CLI output or web admin text?
- Has `logo.svg`'s composition (or name "Context Cartel") been checked against existing trademarks before any public use?
