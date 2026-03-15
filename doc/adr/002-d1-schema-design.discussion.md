# ADR-002 D1 Schema Design — Discussion Log

## Session 2026-02-17 — Schema as Handoff Material

### Starting Point

With SkillRank's implementation delegated to a colleague, the D1 database
schema needed to be specified precisely enough for independent development.
This ADR was created as part of the handoff package alongside CONTRIBUTING.md
and the API spec.

### Decision Journey

The schema design was driven by SkillRank's domain model (ADR-001): hubs are
the primary ranked entities, skills are metadata under each hub, and signals
are the feedback channel from Prism.

> **[Agent]** proposed a five-table schema: `hubs`, `skills`, `signals`,
> `rank_history`, and `crawl_logs`. The design prioritized simplicity — flat
> tables with minimal joins, JSON for flexible fields (tags, metadata,
> components).

Key design decisions:

1. **Skills use composite IDs** (`hub_id:skill_slug`) — this encodes the
   hub-skill relationship directly in the primary key, eliminating the need
   for a separate junction table.

2. **Signals table is append-only** — aligned with the V1 "store only"
   strategy from Skillet ADR-002. Signals are never updated or deleted; new
   readings simply append. This makes the table safe for concurrent writes
   and provides a natural audit trail.

3. **Rank history captures snapshots** — each ranking computation produces a
   row with the score and its component breakdown (as JSON). This enables
   trend analysis and debugging without requiring the implementer to
   reconstruct past rankings.

4. **Three initial hub seeds** — ClawHub, SkillKit Marketplace, and Cursor
   Marketplace. These are the minimum viable hub registry for testing the
   PageRank model.

> **[Human]** reviewed the handoff package holistically and approved the
> approach. The schema, combined with the API spec and CONTRIBUTING.md,
> provides enough context for a colleague to implement CRUD endpoints and
> the crawling pipeline independently.

### Key Human Insights

1. **Delegation as architectural validation** — The decision to hand off
   SkillRank implementation was itself a test of the architecture: if the
   ADRs, schema, and API spec are sufficient for someone else to build it
   without constant clarification, the architecture is well-defined. This is
   a pragmatic application of the "documentation as specification" principle.

### Downstream Effects

**Implements**: SkillRank ADR-001 — the schema directly maps the Hub-as-Domain
model into storage.

**Implements**: Skillet ADR-002 — the `signals` table is designed to receive
the bidirectional feedback defined in the contract.

**Interacts**: SkillRank API spec (`doc/api-spec.md`) — the CRUD endpoints
map 1:1 to these tables.

### Open Questions

- **[Colleague]** Should `PRAGMA foreign_keys = ON` be enforced in D1?
  Default is off in SQLite.
- **[Colleague]** Tags stored as JSON string — acceptable for V1 scale, but
  may need a junction table if tag-based queries become frequent.
