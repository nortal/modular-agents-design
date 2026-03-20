# ADR 003: LLM Synthesis Over Static Agent Definitions

**Date:** 2026-03-20
**Status:** Accepted
**Decision Makers:** Keith Stegbauer

## Context

We needed to decide how agents are created for a consuming project:

1. **Static definitions:** Agent files are hand-authored and cloned/installed directly (gitagent's approach, and all surveyed frameworks).
2. **LLM synthesis:** Agent definitions are read from source repos, and an LLM synthesizes local agents adapted to the project context, governed through PR-based human review.

## Decision

Agents are synthesized by Claude Code (local, with a future path to SaaS), not hand-authored as static files.

## Rationale

### Non-determinism is a feature
As models improve, re-synthesis produces improved agents. Quality gates and human review ensure correctness regardless of variation. Static files lock you to the quality of the original author.

### Constraint-aware generation
The synthesis step can reason about the full constraint graph — generating agents that are coherent with each other, not just individually correct. Static files can't adapt to the architectural context of the consuming project.

### Platform adaptation
Even with the platform transparency root contract rule (ADR 002), synthesis can make smarter choices about which cross-platform patterns to use based on the project's existing tooling.

### Multiple inheritance conflict resolution
When an agent inherits from multiple parents with conflicting definitions, the LLM can reason about the conflict and propose a resolution. Static file merge rules (like gitagent's) can only apply deterministic strategies (child overrides parent, union merge) — they can't reason about which resolution is architecturally appropriate.

### Human-in-the-loop governance
Synthesized agents go through PR review with quality gates. This provides:
- Auditability (every agent has a PR trail)
- Accountability (a human approved every active agent)
- Evolvability (re-synthesis + re-review when requirements change)

## Trade-offs Accepted

- **Requires LLM access** — synthesis needs a Claude API call (or local Claude Code). This is a runtime dependency that static files don't have.
- **Non-reproducible** — two synthesis runs from the same input may produce different agents. Mitigated by quality gates and the PR serving as a snapshot.
- **Slower initial setup** — synthesis takes more time than cloning a file. Acceptable given the quality and governance benefits.

## Synthesis Engine Interface

The interface between the framework and the synthesis engine must be clean enough to swap Claude Code for a SaaS service in the future. The synthesis engine receives:
- Source agent definitions (from git repos)
- Resolved constraint graph
- Project context (existing agents, configuration, environment)

And produces:
- Synthesized agent files
- Conflict resolution report
- PR-ready diff with explanatory comments

## Consequences

- Claude Code is a development dependency for all users of the framework
- Quality gates must be robust enough to catch synthesis errors
- The PR review process must surface synthesis decisions clearly — reviewers need to understand what the AI decided and why
- Change tracking database (secondary priority) should record which model and context produced each synthesis
