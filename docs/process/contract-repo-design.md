# Shared Interface Contract Design

**Date:** 2026-03-20
**Status:** Draft

## Purpose

The `contracts/` directory contains interface schemas that define the boundaries between workstreams. These are the only files that require cross-team agreement — everything else can be developed independently.

## Structure

```
contracts/
  agent-spec/
    agent-definition.schema.yaml       # What an agent definition looks like
    constraint-declaration.schema.yaml  # How agents declare implies/requires/excludes/constrains
    conflict-report.schema.yaml        # How conflicts are reported (consumed by synthesis engine)
  synthesis/
    synthesis-request.schema.yaml      # What the synthesis engine receives as input
    gate-interface.schema.yaml         # How quality gates are defined and invoked
    pr-template.schema.yaml            # Structure of synthesized agent PRs
  observability/
    metrics-hook.schema.yaml           # How metrics are collected during agent execution
    platform-gate.schema.yaml          # Platform transparency validation interface
    cli-config.schema.yaml             # CLI configuration structure
```

## Ownership

| Schema | Published By | Consumed By |
|---|---|---|
| `agent-definition.schema.yaml` | Dev A | Dev B, Dev C |
| `constraint-declaration.schema.yaml` | Dev A | Dev B |
| `conflict-report.schema.yaml` | Dev A | Dev B |
| `synthesis-request.schema.yaml` | Dev B | Dev A |
| `gate-interface.schema.yaml` | Dev B | Dev C |
| `pr-template.schema.yaml` | Dev B | — |
| `metrics-hook.schema.yaml` | Dev C | Dev B |
| `platform-gate.schema.yaml` | Dev C | Dev B |
| `cli-config.schema.yaml` | Dev C | — |

## Change Protocol

1. Developer modifies a schema they own
2. Their AI opens a PR with the change
3. Consuming developers' AIs review for compatibility
4. If compatible: auto-approve
5. If breaking: flag for human review with proposed adaptation
6. Human approves or discusses

## Schema Versioning

Each schema file includes a `version` field. Breaking changes increment the major version. Consuming code must specify which version it targets.
