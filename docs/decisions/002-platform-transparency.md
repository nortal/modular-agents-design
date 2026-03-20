# ADR 002: Platform Transparency as Root Contract Rule

**Date:** 2026-03-20
**Status:** Accepted
**Decision Makers:** Keith Stegbauer

## Context

The framework must work on Windows, macOS, and Linux. We needed to decide how to achieve cross-platform compatibility.

Two approaches were considered:

1. **Synthesis-time adaptation:** The synthesis engine detects the target OS and generates OS-specific output.
2. **Root contract rule:** All agents must produce platform-transparent output regardless of target OS. Enforced through inheritance and quality gates.

## Decision

Platform transparency is a root contract rule in the Friday AI First Club Agent base contract. All inheriting agents must produce output that works on all platforms. This is enforced through quality gates, not synthesis-time adaptation.

## Rationale

- **Simpler and more reliable** than maintaining OS-specific synthesis paths
- **Prevents OS-specific drift** — agents don't slowly accumulate platform assumptions
- **Testable** — a quality gate can statically analyze scripts for OS-specific commands
- **Inherited** — every agent gets this rule automatically through the root contract; child agents cannot weaken it
- **Team discipline** — developers learn to write portable code from the start

## Specific Rules

- Scripts must use cross-platform languages (Python, Node, etc.) — no bare bash, PowerShell, or cmd
- Path handling must use platform-agnostic abstractions (`os.path.join`, `pathlib`, `path.resolve`)
- No OS-specific shell commands (`grep`, `sed`, `rm -rf`, `Get-ChildItem`)
- Environment variable access must account for platform differences
- Line endings handled explicitly
- File permissions must degrade gracefully (Windows lacks `chmod`)
- No assumptions about shell availability

## Consequences

- A quality gate ("platform transparency") must be built to validate this
- Some common patterns (shell scripts for automation) are prohibited — teams must use Python/Node equivalents
- This is more restrictive than gitagent's approach, which hopes authors write portable code but doesn't enforce it
