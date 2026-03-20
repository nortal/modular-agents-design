# Shared Interface Contracts

This directory will contain the interface schemas that define boundaries between development workstreams. During the development phase, each developer publishes their schemas here and consumes others'.

See `docs/process/contract-repo-design.md` for the full design.

## Structure (Planned)

```
contracts/
  agent-spec/           # Dev A: agent definitions, constraints, conflict reports
  synthesis/            # Dev B: synthesis requests, gate interfaces, PR templates
  observability/        # Dev C: metrics hooks, platform gate, CLI config
```

## Status

Schemas will be added when development begins. This directory serves as a placeholder and reference point.
