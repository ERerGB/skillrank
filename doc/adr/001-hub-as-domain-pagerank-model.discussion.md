# ADR-001 Hub-as-Domain PageRank Model — Discussion Log

## Session 2026-02-17 — Identity Crisis and Correction

### Starting Point

SkillRank was originally conceived as a PageRank-inspired ranking engine for
AI Agent Skill Hubs — treating hubs as domains, not individual skills as pages.
During the broader Skillet/Prism/SkillRank architecture sessions, [Agent]
gradually drifted the definition: SkillRank's AGENTS.md and ADR-001 were
rewritten to describe it as a "Skill discovery engine" rather than a "Hub
ranking engine." This identity drift went unnoticed through several iterations
until [Human] caught it.

### Decision Journey

The drift began during discussions about how Prism's Scout subsystem should
integrate with SkillRank. [Agent] reasoned that since Scout discovers content,
and SkillRank indexes skills, they could merge into a unified discovery layer.
This reasoning subtly redefined SkillRank from "rank hubs" to "find skills" —
a fundamental scope change.

> **[Human]** reviewed the updated AGENTS.md and immediately identified the
> problem: "SkillRank 的原始目的——一个 PageRank 启发的 AI Agent Skill Hub
> 排名引擎——是正确的，不应该被重新定义。" This was a sharp correction that
> prevented what would have been architectural scope creep.

The correction required reverting multiple files:
- `AGENTS.md`: "What is SkillRank" section restored to Hub-as-Domain model
- `ADR-001`: Reverted from "Skill discovery" framing back to "Hub ranking"
- `README.md`: Updated downstream consumer reference from SkillNet to Skillet

> **[Agent]** then identified that the revert also needed to preserve a new
> addition: the `/signal` endpoint for receiving Prism's semantic feedback.
> This was additive (a legitimate extension of the PageRank model) rather than
> a redefinition.

The final state cleanly separates concerns: SkillRank ranks hubs (its original
purpose), and the `/signal` endpoint is an input channel that enriches ranking
signals — not a pivot toward skill discovery.

### Key Human Insights

1. **"原始目的是正确的，不应该被重新定义"** — The ability to detect subtle
   scope drift across multiple editing sessions demonstrates the importance of
   having a human owner who holds the original vision. [Agent] had rationalized
   the drift through seemingly logical incremental changes, but [Human] saw
   that the accumulated effect was a category error: ranking infrastructure ≠
   discovery product.

2. **Separation of ranking and discovery** — [Human] maintained that SkillRank
   is pure infrastructure ("no UI, no business logic"), and discovery is a
   product concern that belongs in Skillet. This boundary discipline prevents
   SkillRank from becoming a monolith.

### Downstream Effects

**Preserved**: The core PageRank model — hubs are domains, skills are metadata.
No change to the fundamental ranking algorithm.

**Extended**: Added `POST /signal` endpoint specification as a legitimate
extension for receiving Prism's semantic feedback. V1 stores signals without
using them in ranking; V2 integrates them into PageRank weights.

**Interacts**: Skillet ADR-002 (Bidirectional Feedback Contract) — the
`/signal` endpoint is defined there as the SkillRank side of the bidirectional
interface. The identity correction ensured this interface feeds into *ranking*,
not *discovery*.

### Open Questions

- **[Unowned]** With only ~5-10 hubs currently known, is PageRank meaningful?
  May need additional scoring dimensions beyond pure link-graph analysis.
- **[Human]** SkillRank implementation delegated to colleague — handoff
  materials prepared (CONTRIBUTING.md, D1 schema ADR, API spec).
