# ADR 001: Build Own Framework, Do Not Extend gitagent

**Date:** 2026-03-20
**Status:** Accepted
**Decision Makers:** Keith Stegbauer

## Context

gitagent (github.com/open-gitagent/gitagent) is an open standard that defines AI agents as files in a git repository. It aligns with our "agents in git repos" model and has framework adapters for Claude Code, OpenAI, CrewAI, LangChain, AutoGen, and Google ADK.

We evaluated whether to extend gitagent rather than build our own framework.

## Decision

Build our own framework. Do not extend or fork gitagent.

## Rationale

### Maturity Risk
- Released 2026-02-24 (< 1 month old at time of evaluation)
- Spec version 0.1.0 — expect breaking changes
- 5 contributors, mostly from one company (Lyzr AI) — bus factor risk
- 33 commits total

### Missing Core Features
Our core differentiators do not exist in gitagent and would require such heavy modification that the result would be a fork:
- **No multiple inheritance** — single `extends` field only
- **No constraint graph** — no architectural interdependence modeling
- **No AI-driven conflict resolution** — deterministic merge rules only
- **No LLM synthesis** — agents are hand-authored static files
- **No observability** — no token, clock, or human-minutes tracking

### Philosophical Conflict
gitagent believes agents are hand-authored static files ("clone a repo, get an agent"). We believe agents are synthesized by AI and governed through PR-based human review. These are fundamentally different design philosophies.

### Platform Transparency Gap
gitagent's CLI is cross-platform (Node.js), but agent scripts/hooks inside the file structure can be OS-specific. Our framework requires platform transparency as a root contract rule enforced through inheritance and quality gates.

### Low Value of Extension
The only non-trivial thing gitagent provides is the adapter layer (export to multiple frameworks). This is a thin translation layer — not worth coupling architecture to an unstable project.

## What We Learn From gitagent

- File structure conventions (agent.yaml, SOUL.md, RULES.md) are good naming patterns
- RULES.md union-merge with parent (cannot weaken parent rules) is a sound inheritance rule
- DUTIES.md conflict matrix concept could inform constraint declarations
- Adapter pattern for framework portability is worth studying

## Consequences

- We own our spec and can evolve it freely
- We must build our own framework adapters if we want multi-framework export
- We should monitor gitagent's evolution — if it matures, adapter compatibility could be added later as an export format

## References

See `docs/research/gitagent-analysis.md` for the full evaluation.
