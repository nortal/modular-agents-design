# gitagent — Deep Dive Analysis

**Date:** 2026-03-20
**Purpose:** Evaluate gitagent as a potential foundation or reference for the Agent Stacking Framework.

## Overview

gitagent is an open standard (MIT License) that defines AI agents as files in a git repository. Created by Shreyas Kapale / Lyzr AI (San Francisco). Released 2026-02-24. "Clone a repo, get an agent."

- **GitHub:** github.com/open-gitagent/gitagent (702 stars, 60 forks, 5 contributors)
- **Spec Version:** v0.1.0
- **Language:** TypeScript (Node.js >= 18)
- **License:** MIT

## Agent File Structure

An agent is a directory with structured files (only `agent.yaml` and `SOUL.md` are required):

```
my-agent/
  agent.yaml          # Manifest (name, version, model, extends, dependencies)
  SOUL.md             # Identity and personality (free-form markdown)
  RULES.md            # Hard constraints / safety boundaries
  DUTIES.md           # Segregation of duties policy
  AGENTS.md           # Framework-agnostic fallback (for Cursor/Copilot)
  skills/             # Reusable capability modules (Agent Skills open standard)
  tools/              # MCP-compatible tool definitions (YAML schemas)
  workflows/          # Multi-step deterministic procedures (YAML with depends_on)
  knowledge/          # Reference documents with index.yaml
  memory/             # Persistent cross-session state (MEMORY.md, archive/)
  hooks/              # Lifecycle handlers (on_session_start, pre_tool_use, etc.)
  agents/             # Sub-agent definitions (recursive)
  compliance/         # Regulatory artifacts (regulatory-map.yaml, audit schemas)
  config/             # Environment-specific overrides (default.yaml, production.yaml)
  examples/           # Calibration I/O examples
  .gitagent/          # Runtime state (gitignored)
```

## Mapping to Agent Stacking Requirements

| Requirement | gitagent | Gap? |
|---|---|---|
| **OO inheritance** | Single inheritance via `extends: <git-url>` | Only single, not multiple |
| **Multiple inheritance + AI conflict resolution** | Not supported | **Major gap** |
| **Constraint graph** | RULES.md (union-merge), DUTIES.md (conflict matrices), workflow DAGs | Partial — behavioral constraints, not architectural interdependence |
| **Agent per git repo** | Core design principle | Perfect alignment |
| **Swappability** | Adapter system exports to Claude, OpenAI, CrewAI, LangChain, AutoGen, Google ADK | Strong |
| **Cross-platform** | Node.js CLI runs on Win/Mac/Linux | CLI is cross-platform, but agent scripts/hooks may not be |
| **PR-based governance** | Git-native — PRs, diffs, blame all work naturally | Structural support, but no built-in enforcement |
| **Token/time tracking** | Not built in | **Gap** |
| **Human-minutes** | Not built in | **Gap** |
| **Synthesis via LLM** | `gitagent init` scaffolds from templates, no LLM synthesis | **Major gap** |
| **Safe upgrades** | Semver versioning, git tags, dependency pinning | Good |
| **Init command** | `gitagent run <repo> --adapter claude` | Runs one agent, doesn't compose a stack |
| **MCP/A2A interop** | Both supported | Strong |
| **Platform transparency** | Not enforced — hooks use JSON stdin/stdout protocol but scripts inside agents can be OS-specific | **Gap** |

## Platform Transparency Concern

gitagent's CLI is cross-platform (Node.js), but agents themselves can contain:
- `hooks/` — lifecycle scripts that may use OS-specific commands
- `workflows/` — procedures that may reference OS-specific tooling
- `skills/` — capability modules with OS-specific implementations

The spec says hooks use a "JSON stdin/stdout protocol so implementations can be in any language" — but that's a convention, not enforcement. Nothing prevents an agent author from writing a bash hook that fails on Windows.

**This is a fundamental design difference from the Agent Stacking Framework**, where platform transparency is a root contract rule enforced through inheritance and quality gates.

## Inheritance Model

gitagent supports single inheritance via `extends` in `agent.yaml`:

```yaml
extends: https://github.com/org/base-agent.git
```

Resolution rules:
- `agent.yaml`: deep merge, child overrides parent
- `SOUL.md`: child replaces parent entirely
- `RULES.md`: child rules **append** to parent (union — cannot remove parent rules)
- `skills/`, `tools/`: union, child shadows parent on name collision
- `memory/`: isolated, not inherited
- `compliance/`: inherited, can override

**No multiple inheritance.** The `extends` field is a string, not an array.

## Composition Mechanisms

1. **Sub-agents** (`agents/` directory): Recursive — each sub-agent is a full gitagent definition
2. **Dependencies** (`dependencies` in `agent.yaml`): Mount external agents from git URLs with version pinning
3. **Delegation**: Configurable strategy (`auto`, `explicit`, or `router` mode)

## Framework Integrations (Adapters)

- Claude Code, OpenAI Agents SDK, CrewAI, Lyzr Studio, LangChain, AutoGen, Google ADK
- GitHub Actions
- MCP and A2A protocol support

## Known Criticisms (Hacker News)

1. **Secret management**: `.env` + `.gitignore` approach called "hope-based security"
2. **Discovery problem**: Defining agents is not the bottleneck — runtime tool discovery is
3. **Prompt injection**: SOUL.md and SKILL.md become attack surfaces in a forking ecosystem
4. **Runtime safety**: Version control doesn't prevent runtime destructive actions

## Strategic Assessment

### Decision: Do Not Extend gitagent (see ADR 001)

**Reasons:**
- < 1 month old, spec v0.1.0, expect breaking changes
- 5 contributors mostly from one company (Lyzr AI) — bus factor risk
- Core differentiators (constraint graph, multiple inheritance + AI conflict resolution, LLM synthesis) don't exist and would require such heavy modification that the result would be a fork
- Static-file philosophy conflicts with synthesis-based approach
- Platform transparency is not enforced
- TypeScript/Node dependency adds runtime requirement

**What to learn from:**
- File structure conventions (agent.yaml, SOUL.md, RULES.md) are good naming patterns
- RULES.md union-merge with parent is a sound inheritance rule for constraints
- Adapter pattern for framework portability is worth studying
- DUTIES.md conflict matrix concept could inform constraint declarations

**Future option:** If gitagent matures, adapter compatibility could be added as an export format.

## References

Kapale, Shreyas. "GitAgent." *GitHub*, open-gitagent, github.com/open-gitagent/gitagent.

Kapale, Shreyas. "GitAgent: 14 Patterns All AI Agents Should Follow." *Medium*, Mar. 2026.

Kapale, Shreyas. "Show HN: GitAgent — An Open Standard That Turns Any Git Repo into an AI Agent." *Hacker News*, news.ycombinator.com/item?id=47376584.

"GitAgent." *gitagent.sh*, www.gitagent.sh/.

"GitAgent Registry." *registry.gitagent.sh*, registry.gitagent.sh/.
