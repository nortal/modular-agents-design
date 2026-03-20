# Agent Stacking Framework — Requirements Document

## Project Overview

**Project Name:** Friday AI First Club Agent Framework (Agent-Base)

**Organization:** Nortal

**Purpose:** A modular, cross-platform framework for defining, synthesizing, and governing reusable AI agents across projects. Agents are composed through an object-oriented inheritance model with constraint-aware synthesis, governed through PR-based human review with quality gates. The framework is domain-agnostic — it provides the composition and governance machinery, not the domain agents themselves.

**Core Principles:**
- Agents are **proposed**, not deployed. Every change goes through human-in-the-loop review.
- Agents are **interdependent**. Selecting one agent has cascading implications across the stack. The framework reasons about these constraints holistically.
- The framework is **domain-agnostic**. It could compose IaC agents, application architecture agents, CI/CD agents, or any future domain.

## 1. Agent Definition Model

### 1.1 Object-Oriented Inheritance

Agent definitions follow an inheritance hierarchy rooted in a base contract:

```
Friday AI First Club Agent  (root contract)
└── [Domain Agent]  (e.g., persistence, compute, pipeline — implements root contract)
    └── [Implementation Agent]  (e.g., DynamoDB Global Tables, Aurora Global — implements domain contract)
```

- All agents implement the **Friday AI First Club Agent** root contract
- Specialized agents extend parent agents with domain-specific knowledge
- Agents must be **swappable** — any implementation can be replaced by another that satisfies the same parent interface without breaking the project
- Agent definitions must declare their parent(s) and the interface they fulfill

#### 1.1.1 Platform Transparency (Root Contract Rule)

All agents inheriting from the Friday AI First Club Agent root contract must produce platform-transparent output. This is a non-negotiable, inherited rule — child agents cannot override or weaken it.

**Requirements:**

- Scripts must use cross-platform languages (Python, Node, etc.) — no bare bash, PowerShell, or cmd scripts
- Path handling must use forward slashes or platform-agnostic abstractions (e.g., `os.path.join`, `pathlib`, `path.resolve`)
- No OS-specific shell commands (`grep`, `sed`, `rm -rf`, `Get-ChildItem`, etc.) — use language-level equivalents
- Environment variable access must account for platform differences
- Line endings must be handled explicitly (no assumption of LF or CRLF)
- File permission operations must degrade gracefully on platforms that don't support them (e.g., Windows lacks `chmod`)
- No assumptions about shell availability (`/bin/sh`, `cmd.exe`)

**Enforcement:**

- The rule is inherited by all agents through the root contract — it cannot be removed by child agents
- The quality gate suite (Section 7) includes a **platform transparency check** that validates synthesized agent output against these requirements
- Violations fail the gate — no exceptions without explicit human override documented in the PR

### 1.2 Multiple Inheritance with AI Conflict Resolution

Agents may inherit from multiple parents (e.g., an agent that spans compute and networking concerns).

- The synthesis step **detects** conflicts between parent definitions
- Claude Code **proposes** a resolution based on project context
- Conflict resolutions are **explicitly surfaced** in the PR — never silently resolved
- Human reviewers approve, modify, or reject proposed resolutions
- Quality gates specifically validate inherited behaviors from all parent chains

### 1.3 Swappability

If the direction of a project changes, any agent can be swapped for an alternative implementation of the same interface. The framework maintains a clean boundary between:

- **What an agent does** (interface/contract — inherited from parent)
- **How it does it** (implementation — specific to the agent)

Swapping an agent triggers re-evaluation of the constraint graph (see Section 2).

## 2. Constraint Graph

### 2.1 Agent Interdependence

Agents do not exist in isolation. Selecting an agent has cascading implications for the rest of the stack. Each agent declares:

| Declaration | Purpose |
|---|---|
| **Implies** | What this agent introduces as a requirement (e.g., "Springboot" implies a JVM runtime and a persistence layer) |
| **Requires** | Hard dependencies on other agents or capabilities |
| **Excludes** | What this agent is incompatible with (e.g., "Lambda" excludes non-AWS compute) |
| **Constrains** | Restrictions this agent places on other agents' options (e.g., "Multi-region" narrows available persistence agents) |

### 2.2 Constraint Resolution

When a user declares project needs, the framework must:

1. **Resolve the constraint graph** — determine what agents and options remain valid given all declared requirements
2. **Surface conflicts early** — identify when declared requirements create contradictions or eliminate all options in a domain
3. **Propose a coherent agent set** — not individual agents selected in isolation, but a compatible composition across the full stack
4. **Present trade-offs** — when constraints conflict, show what options open up if specific requirements are relaxed

### 2.3 AI-Driven Constraint Reasoning

Static dependency resolution handles simple cases (A requires B, A excludes C). But architectural trade-offs across domains — compute, storage, networking, CI/CD, region strategy, compliance — require contextual reasoning. The synthesis engine (Claude Code) reasons about:

- Cross-domain implications that aren't captured in simple dependency declarations
- Emerging best practices and evolving platform capabilities
- Project-specific context that affects which trade-offs are acceptable

This reasoning is always surfaced for human review, never applied silently.

### 2.4 Illustrative Example

```
User declares: "Springboot" + "Multi-region" + "Lambda"

Framework resolves:
  "Springboot"
      → implies Java/Kotlin runtime
      → implies persistence layer needed
  "Multi-region"
      → constrains persistence to multi-region-capable options
        (e.g., DynamoDB Global Tables, Cosmos DB, CockroachDB, Spanner)
      → constrains compute to multi-region-capable options
  "Lambda"
      → implies AWS
      → excludes non-AWS compute (Kubernetes on Azure, Cloud Run, etc.)
      → further constrains persistence to AWS-native multi-region options
        (e.g., DynamoDB Global Tables, Aurora Global Database)

Framework surfaces:
  "Lambda + Multi-region limits persistence to DynamoDB Global Tables
   or Aurora Global Database. If you drop the Lambda requirement,
   Kubernetes + CockroachDB and other options become available.
   Here are the trade-offs: ..."
```

This example is illustrative. The framework itself is domain-agnostic — it provides the constraint resolution machinery, not the domain knowledge. Domain knowledge lives in the agents.

## 3. Synthesis & Governance

### 3.1 Synthesis Engine

The framework reads agent definitions from source git repositories and synthesizes local agents adapted to the target project, environment, and resolved constraint graph.

**Current engine:** Claude Code (local)
**Future path:** SaaS-based synthesis service

The interface between the framework and the synthesis engine must be clean enough to support this transition without restructuring agent definitions.

### 3.2 Synthesis Lifecycle

```
User declares project requirements
    → Framework resolves constraint graph
    → Surfaces conflicts and trade-offs for human decision
    → Synthesizes coherent agent set from source repos
    → PR created with proposed agents
    → Quality gates run automatically
    → Human review
    → Merge
    → Agents become active
```

### 3.3 Non-Determinism by Design

Synthesis is intentionally non-deterministic. As models improve, re-synthesis of the same source definitions may produce improved agents. This is a feature — the quality gates and human review ensure correctness regardless of variation.

### 3.4 Initialization

A single command initializes a project:

- User declares desired capabilities and constraints
- Framework resolves the constraint graph and presents options/trade-offs
- User confirms choices
- Framework synthesizes agents from source repos
- Updates `CLAUDE.md` and helper files to reflect the active agent stack

### 3.5 Re-evaluation

Adding, removing, or swapping any agent triggers re-evaluation of the full constraint graph. The framework identifies what changed, what new conflicts or options emerged, and proposes adjustments — all through the PR/review process.

## 4. Observability

### 4.1 Per-Agent-Call Metrics

Every agent invocation must track:

| Metric | Description |
|---|---|
| **Token usage** | Total tokens consumed (input + output) per call |
| **Clock seconds** | Wall-clock time elapsed per call |
| **Human-minutes** | Human time spent on the task the agent assisted with |

These metrics serve dual purposes: operational observability and input to quality gates (e.g., flagging cost regressions).

### 4.2 Change Tracking Database

A lightweight record of agent evolution:

- PR links for each agent change
- Source git repo and version that triggered the synthesis
- Which model/context produced the synthesis
- Constraint graph state at time of synthesis

**Priority:** Secondary. The system must support this but it is not a primary feature for v1. Design should ensure we can come back to it.

## 5. Platform Requirements

### 5.1 Cross-Platform Compatibility

The framework must work transparently on:

- Windows
- macOS
- Linux

The platform transparency rule (Section 1.1.1) is enforced at the root contract level and validated through quality gates. All agents must produce output that works on all three platforms without modification.

### 5.2 Caching

The framework should use tensor/embedding caching effectively to minimize redundant LLM API calls during synthesis and agent operation.

## 6. Git Architecture

### 6.1 Repository Structure

- **Agent-base** lives in a core repository containing the root contract, synthesis engine interface, quality gate definitions, constraint resolution machinery, and initialization tooling
- **Each agent** lives in its own git repository
- **Consuming projects** reference agents by repo + version

### 6.2 Safe Upgrades

Agent upgrades follow the same synthesis → PR → gates → review → merge lifecycle. No agent is upgraded without human approval. The PR history provides a complete audit trail of every agent's evolution.

## 7. Quality Gates

Synthesized agents must pass the following gates before human review:

| Gate | Description |
|---|---|
| **Schema validation** | Agent conforms to its declared interface contract |
| **Constraint compliance** | Agent's declarations (implies, requires, excludes, constrains) are consistent with the resolved graph |
| **Capability test** | Agent performs its declared functions against a test scenario |
| **Regression check** | Agent handles all cases the previous version handled |
| **Cost/token budget** | Agent stays within acceptable token usage bounds |
| **Security review** | Agent requests only the permissions it should have |
| **Inheritance validation** | Behaviors from all parent chains are correctly implemented |
| **Platform transparency** | All scripts, paths, and OS interactions conform to the root contract's platform transparency rule |
| **Peer review** | Human evaluates the synthesis output |

## 8. Validation Use Case

**Scenario:** Terraform development on Azure with a safe dev/test/prod methodology

| Concern | Choice |
|---|---|
| **Infrastructure-as-Code** | Terraform on Azure |
| **Environments** | dev / test / prod |
| **Compute** | Kubernetes |
| **App CI/CD** | GitHub Actions |
| **Infra CI/CD** | Azure DevOps Pipelines |
| **Pipeline gates** | Lint, SonarQube, SAST, DAST |

This use case exercises multiple agent types across two CI/CD platforms and multiple environments. It is a **validation scenario for the framework**, not a framework feature. If the framework can compose a coherent, governed agent stack for this scenario, it generalizes to other domains.

## Open Questions for Future Sessions

1. What schema/format should agent definitions use? (YAML, Markdown, Python classes, etc.)
2. How are constraint declarations structured? What's the schema for implies/requires/excludes/constrains?
3. How is human-minutes tracked? (Timer, self-reported, inferred from interaction patterns?)
4. What does the project configuration file look like for init?
5. How does feedback from rejected PRs flow back to improve future synthesis?
6. What is the versioning strategy for agent repos? (Semver tags, branch-based, etc.)
