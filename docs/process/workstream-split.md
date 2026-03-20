# Development Workstream Plan

**Date:** 2026-03-20
**Team Size:** 3 developers
**Status:** Draft

## Overview

Work is split into three parallel workstreams designed to be **orthogonal** — each developer can work independently with minimal blocking dependencies. AI acceleration (see `ai-acceleration.md`) eliminates the traditional Phase 0 spec bottleneck.

## Workstream Assignments

### Dev A: Constraint Graph + Spec

**Owns:** The constraint resolution engine, inheritance resolution (including multiple inheritance), conflict detection.

**Core work:**
- Parse agent declarations → build dependency graph → detect conflicts → surface trade-offs
- Define the agent definition schema (the spec all workstreams build against)
- Implement inheritance merger with multiple parent support

**Builds:**
- Constraint solver
- Inheritance merger
- Conflict report generator

**Tests against:**
- Unit tests with mock agent specs that have known conflicts
- Diamond inheritance scenarios
- Cascading constraints (Springboot + Lambda + Multi-region example)

**Skills needed:** Graph algorithms, schema design, architectural reasoning

---

### Dev B: Synthesis + Governance

**Owns:** Claude Code integration for synthesis, PR workflow, quality gate framework.

**Core work:**
- Read agent specs from source repos → invoke Claude Code → generate synthesized agent → create PR
- Build the gate runner harness with plugin slots
- Design PR template that surfaces constraint resolutions and conflict decisions

**Builds:**
- Synthesis engine interface (SaaS-swappable)
- PR template generator
- Gate runner harness

**Tests against:**
- End-to-end: source spec in → synthesized agent out → PR created → gates pass
- Multiple synthesis runs to validate non-determinism is handled by gates

**Skills needed:** LLM integration, git/GitHub API, CI/CD pipeline design

---

### Dev C: Observability + CLI

**Owns:** Token/clock/human-minutes tracking, init command, platform transparency gate.

**Core work:**
- Instrument agent calls → collect metrics → report
- Build `init` command that wires project to selected agents
- Build platform transparency validator

**Builds:**
- Metrics collector (token, clock, human-minutes)
- CLI entry point (`init`, `synthesize`, `validate`, `report`)
- Platform transparency quality gate

**Tests against:**
- Instrumented test agents
- Init against a test agent set
- Scripts validated on Windows, macOS, and Linux

**Skills needed:** Metrics/telemetry, CLI design, cross-platform testing

## Integration Points

| Touchpoint | Between | What to Agree On |
|---|---|---|
| Agent spec schema | A ↔ B, A ↔ C | The parsed representation of an agent that all three consume |
| Conflict report format | A → B | How constraint conflicts are represented so synthesis can reason about them |
| Gate interface | B ↔ C | How gates are defined, invoked, and report pass/fail |
| Metrics hook points | C ↔ B | Where in the synthesis/execution flow metrics get collected |

## Integration Phase

All three developers come together to:
- Wire components into a working end-to-end flow
- Run the Terraform/Azure validation scenario
- Fix integration gaps
- Write first real agents as proof of concept

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Spec debates drag on | AI-accelerated approach eliminates Phase 0 bottleneck (see `ai-acceleration.md`) |
| Dev A's constraint graph is too academic | Ground it in real examples — Springboot + Lambda + Multi-region is the constant test case |
| Dev B's synthesis produces unpredictable output | Quality gates exist for this reason. Start with simple agents. |
| Cross-platform testing gaps | Dev C needs CI matrix with Windows/Mac/Linux from day one |
| Three devs diverge on assumptions | Shared contract repo + twice-weekly interface PR review |
