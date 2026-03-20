# AI-Accelerated Orthogonal Development

**Date:** 2026-03-20
**Status:** Draft

## Problem

Traditional multi-developer projects have a blocking dependency: the spec/interface must be agreed before anyone can build against it. This creates a Phase 0 bottleneck where all developers wait for consensus.

## Solution

Each developer uses an AI agent (Claude Code) that can:

1. Read the current state of all three workstreams
2. Adapt its own workstream's code to match interface changes
3. Propose interface contracts rather than wait for them

Nobody blocks. Each developer works against their AI's best understanding of the interface, and reconciliation happens continuously.

## How It Works

### Instead of This (Blocking)

```
A defines spec → B and C wait → B and C build → integrate → fix
```

### You Get This (Orthogonal)

```
A, B, C each start Day 1 with AI generating their best-guess interfaces
    ↓
Each publishes interface proposals to a shared contract repo
    ↓
Each developer's AI reads the others' proposals and adapts
    ↓
Conflicts surface as automated PRs, not meetings
    ↓
Humans review interface PRs — that's the only sync point
```

## The Shared Contract Repo

A single repo (or directory within this repo) where all three workstreams publish interface contracts — not code, just schemas:

```
contracts/
  agent-spec/
    agent-definition.schema.yaml    ← Dev A publishes
    constraint-declaration.schema.yaml  ← Dev A publishes
    conflict-report.schema.yaml     ← Dev A publishes, Dev B consumes
  synthesis/
    synthesis-request.schema.yaml   ← Dev B publishes, Dev A consumes
    gate-interface.schema.yaml      ← Dev B publishes, Dev C consumes
    pr-template.schema.yaml         ← Dev B publishes
  observability/
    metrics-hook.schema.yaml        ← Dev C publishes, Dev B consumes
    platform-gate.schema.yaml       ← Dev C publishes, Dev B consumes
    cli-config.schema.yaml          ← Dev C publishes
```

**Rule:** When you change a contract, your AI opens a PR. The consuming developer's AI reviews it, tests compatibility, and either auto-approves or flags conflicts for human review.

## What Each Developer's AI Does

### Dev A's AI (Constraint Graph)
- Generates candidate schemas from the requirements doc and example scenarios
- Runs the Springboot + Lambda + Multi-region scenario against every schema revision
- Publishes `conflict-report.schema.yaml` — Dev B's AI immediately adapts synthesis to consume it
- When Dev B or C propose interface changes, A's AI evaluates whether the constraint model can support them

### Dev B's AI (Synthesis + Governance)
- Doesn't wait for A's final schema — generates a synthesis engine against A's latest draft
- Builds the gate runner harness with plugin slots — Dev C's gates plug in without coordination
- When A's schema changes, B's AI adapts the synthesis engine and opens a PR showing the delta
- Prototypes the PR template and workflow against GitHub API immediately

### Dev C's AI (Observability + CLI)
- Builds the metrics collection framework independently — token counting, clock timing, and human-minutes tracking don't depend on the agent spec at all
- Defines the gate plugin interface and publishes it — Dev B's AI adapts the harness
- Builds the CLI skeleton immediately — `init`, `synthesize`, `validate`, `report`
- Platform transparency gate can be built and tested against sample scripts without waiting for real agents

## What Doesn't Actually Need the Spec

| Work Item | Needs Spec? |
|---|---|
| Constraint graph algorithm | Needs schema, but AI can generate and iterate candidates |
| Schema design | IS the spec — Dev A drives this |
| Synthesis engine core (Claude Code integration) | No — LLM integration is spec-independent |
| PR creation workflow | No — GitHub API integration |
| Gate runner harness | No — plugin architecture |
| Token counting | No — wrapper around LLM API responses |
| Clock time tracking | No — stopwatch |
| Human-minutes tracking | No — UX/interaction design |
| CLI skeleton | No — command parsing, config loading |
| Platform transparency validator | No — static analysis of scripts |
| Inheritance merger | Needs schema, but prototypable against drafts |
| Conflict detection | Needs constraint declarations, but testable against synthetic examples |

Over half the work doesn't need the spec at all.

## Human Sync Points (Minimized)

| When | What | How |
|---|---|---|
| **Day 1** | 30-minute kickoff: agree on language (Python), repo structure, contract repo convention | Meeting |
| **Twice weekly** | Review interface PRs that AIs flagged as needing human decision | Async PR review |
| **End of each week** | 15-minute demo: each dev shows what their workstream produces | Meeting |
| **Integration milestone** | Wire the three components together, run the validation scenario | Pairing session |

## Timeline Impact

**Traditional:** 1 week spec → 3-4 weeks parallel build → 1 week integration = 5-6 weeks

**AI-accelerated:** All three start Day 1 → continuous AI-mediated integration → human sync twice/week → 2-3 weeks to working prototype

## Meta-Observation

This approach uses the framework's own philosophy to build the framework: AI-driven synthesis with human-in-the-loop governance, applied to the development process itself.
