# Friday AI First Club — Agent Stacking Framework

## What Is This?

This is a **planning and research repository** for a modular AI agent framework being developed at Nortal. It contains requirements, research, architectural decisions, and process documentation — no implementation code.

The framework ("Agent-Base") enables teams to compose reusable AI agents through object-oriented inheritance, synthesize them via LLM with human-in-the-loop PR governance, and reason about architectural constraints across an entire project stack.

## Quick Start for New Developers

1. Read this file
2. Read `docs/requirements/agent-framework-requirements.md` — the full requirements
3. Read `docs/decisions/` — architectural decisions and rationale
4. Read `docs/process/workstream-split.md` — how work is distributed
5. Read `docs/research/` — framework landscape analysis and research

## Key Concepts

- **Constraint Graph**: Selecting one agent (e.g., "Lambda") has cascading implications for other agents (restricts persistence options to AWS-native). The framework reasons about these interdependencies.
- **Multiple Inheritance + AI Conflict Resolution**: Agents can inherit from multiple parents. The LLM detects and proposes resolutions for conflicts, surfaced via PR for human review.
- **LLM Synthesis**: Agents are not hand-authored static files. Claude Code reads agent definitions from source repos and synthesizes local agents adapted to the project — governed through PR-based quality gates.
- **Platform Transparency**: A root contract rule enforced through inheritance — all agents must produce output that works on Windows, macOS, and Linux.

## Repository Structure

See `INDEX.md` for the full repository map.

## How to Contribute

- All documentation changes go through PR review
- Research must include MLA-style references
- Architectural decisions must be recorded in `docs/decisions/`
- Keep `INDEX.md` up to date after structural changes
