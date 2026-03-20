# Constraint Graph — Design Rationale

**Date:** 2026-03-20
**Status:** Draft

## Core Concept

Agents are interdependent. Selecting one agent has cascading implications across the stack. The constraint graph is the data structure and resolution engine that models these relationships.

## Why Not Simple Dependency Resolution?

Traditional dependency managers (npm, pip, Maven) resolve version compatibility — "A requires B >= 2.0." This is necessary but insufficient for agent composition because:

1. **Architectural implications aren't version-based.** Choosing "Lambda" doesn't require a specific version of a persistence agent — it eliminates entire categories of persistence agents (anything non-AWS).

2. **Trade-offs are contextual.** Whether "Lambda + DynamoDB" is better than "Kubernetes + CockroachDB" depends on the project's latency requirements, team expertise, compliance constraints, and budget. A dependency resolver can't reason about this.

3. **Cross-domain effects are non-obvious.** A "Multi-region" requirement affects compute, storage, networking, and CI/CD simultaneously. These interactions require holistic reasoning, not pairwise dependency checks.

## Agent Declarations

Each agent declares its constraints using four relationship types:

### Implies

What this agent introduces as a new requirement:

```yaml
implies:
  - jvm-runtime        # Springboot implies a JVM
  - persistence-layer   # Springboot implies need for a database
```

### Requires

Hard dependencies on other agents or capabilities:

```yaml
requires:
  - aws-account         # Lambda requires AWS
  - iam-permissions      # Lambda requires IAM setup
```

### Excludes

What this agent is incompatible with:

```yaml
excludes:
  - azure-compute        # Lambda excludes Azure compute options
  - gcp-compute          # Lambda excludes GCP compute options
  - on-premises-compute  # Lambda excludes on-prem
```

### Constrains

Restrictions placed on other agents' options:

```yaml
constrains:
  persistence:
    must-support: multi-region   # Multi-region constrains persistence options
  compute:
    must-support: multi-region   # Multi-region constrains compute options
```

## Resolution Process

```
1. User declares requirements: ["Springboot", "Multi-region", "Lambda"]

2. Framework collects all declarations:
   - Springboot implies: [jvm-runtime, persistence-layer]
   - Multi-region constrains: [persistence.must-support: multi-region, compute.must-support: multi-region]
   - Lambda requires: [aws-account, iam-permissions]
   - Lambda excludes: [azure-compute, gcp-compute, on-premises-compute]
   - Lambda implies: [aws-native]

3. Framework resolves:
   - persistence-layer + multi-region + aws-native → [DynamoDB Global Tables, Aurora Global Database]
   - Eliminated: Cosmos DB (excluded by Lambda/AWS), CockroachDB (not AWS-native), single-region RDS

4. Framework surfaces trade-offs:
   - "Your choices narrow persistence to DynamoDB Global Tables or Aurora Global Database."
   - "If you drop Lambda, Kubernetes + CockroachDB becomes available across any cloud."
   - "DynamoDB Global Tables offers lower latency; Aurora Global offers SQL compatibility."

5. Human decides. Framework records the decision.
```

## AI's Role in Resolution

The constraint graph has two layers:

1. **Declarative layer** — the implies/requires/excludes/constrains declarations. These are machine-parseable and deterministic.

2. **Reasoning layer** — the AI examines the resolved graph and adds contextual analysis:
   - Emerging best practices ("Aurora Global added write forwarding in 2025, making it more viable for multi-region write workloads")
   - Non-obvious interactions ("Lambda cold starts may conflict with your latency SLAs for this use case")
   - Project-specific context ("Your team has more DynamoDB experience based on the existing codebase")

The reasoning layer always produces recommendations, never decisions. Decisions are human.

## Relationship to Multiple Inheritance

When an agent inherits from multiple parents, both parents' constraint declarations merge. If Parent A excludes something that Parent B requires, the AI detects this conflict and proposes a resolution — surfaced in the PR for human review.

## Open Questions

1. What is the schema format for constraint declarations? (YAML proposed above, but needs validation)
2. How deep does the implies chain resolve? (A implies B, B implies C — is there a depth limit?)
3. How are "soft" constraints handled vs. "hard" constraints? (e.g., "prefers AWS" vs. "requires AWS")
4. Should the constraint graph be visualizable? (Graph rendering for PR reviews)
