# Constraint Solver Libraries — Deep Dive

**Date:** 2026-03-21
**Purpose:** Evaluate open source constraint satisfaction libraries for implementing the Agent Stacking Framework's constraint graph.

## The Use Case

When a user selects agents for a project (e.g., "Springboot" + "Lambda" + "Multi-region"), the system must resolve cascading constraints: Lambda implies AWS, which excludes Azure compute, which narrows persistence options. The solver must:

1. Determine if a selection is feasible
2. **Explain WHY** it's infeasible (not just that it is)
3. **Suggest what to relax** to make it feasible
4. Present trade-offs in human-readable form

## Library Comparison

| Library | Implies/Excludes | UNSAT Core | MUS/Explain | Relaxation Suggestions | Stars | License | Maintained |
|---|---|---|---|---|---|---|---|
| python-constraint2 | Via functions | No | No | No | 510 | BSD-2 | Yes (v2.4.0, Jan 2026) |
| OR-Tools CP-SAT | Yes (native) | Yes (assumptions) | Partial | Via optimization | 13.2k | Apache 2.0 | Yes (v9.15, Jan 2026) |
| MiniZinc+Python | Yes (native) | Backend-dependent | No | No | 186 | MPL-2.0 | Yes (v0.10.0, 2025) |
| **Z3** | **Yes (native)** | **Yes (assert_and_track)** | **Via core** | **Via MaxSMT** | **12.1k** | **MIT** | **Yes (ongoing)** |
| **CPMpy** | **Yes** | **Yes** | **Yes (MUS, MARCO)** | **Yes (MCS)** | **340** | **Apache 2.0** | **Yes (v0.10.0, Jan 2026)** |
| PySAT | Yes (CNF) | Yes | Yes (MUS/MCS) | Yes (MCS) | 447 | MIT | Slower releases |
| NetworkX | N/A (graph only) | N/A | N/A | N/A | 15k+ | BSD | Yes |

## Detailed Assessments

### Z3 Theorem Prover (Microsoft) — Top Recommendation

An SMT (Satisfiability Modulo Theories) solver supporting integers, reals, booleans, arrays, strings, and quantifiers.

**Why it's the best fit:**
- `Implies(a, b)`, `And(a, Not(b))` for excludes — first-class operations
- `solver.assert_and_track(constraint, "human-readable-label")` — labeled constraints
- When infeasible, `solver.unsat_core()` returns the specific labeled constraints that conflict
- MaxSMT (optimization mode) for finding what to relax
- Millisecond performance for technology-constraint problems
- 12.1k GitHub stars, MIT license, actively maintained by Microsoft Research
- Cross-platform: Windows, macOS, Linux, ARM, WebAssembly
- Python bindings: `z3-solver` on PyPI (113k weekly downloads)

**Example for our use case:**
```python
from z3 import *

# Boolean variables for technology selections
lambda_selected = Bool('lambda')
aws = Bool('aws')
azure_compute = Bool('azure_compute')

s = Solver()
s.assert_and_track(Implies(lambda_selected, aws), "Lambda-requires-AWS")
s.assert_and_track(Implies(aws, Not(azure_compute)), "AWS-excludes-Azure-compute")
s.assert_and_track(azure_compute, "User-selected-Azure-compute")
s.assert_and_track(lambda_selected, "User-selected-Lambda")

if s.check() == unsat:
    core = s.unsat_core()
    # Returns: ["Lambda-requires-AWS", "AWS-excludes-Azure-compute",
    #           "User-selected-Azure-compute", "User-selected-Lambda"]
```

**Gap:** Z3's API is "logician-oriented" — there's a learning curve. Output is structured data, not natural language.

### CPMpy — Strong Complement to Z3

A NumPy-based constraint programming modeling library providing a solver-agnostic front-end. Supports Z3 as a backend.

**Why it's valuable:**
- Pythonic, NumPy-style constraint modeling
- Built-in MUS (Minimal Unsatisfiable Subset) extraction
- MARCO algorithm for enumerating all MUS/MSSes
- MCS (Minimal Correction Subset) computation — directly answers "what to relax"
- Won gold medals in XCSP3 competitions (2024, 2025)
- 340 GitHub stars, Apache 2.0, actively maintained

**Gap:** Smaller community than Z3 or OR-Tools.

### Google OR-Tools CP-SAT

Industrial-grade constraint solver. 13.2k stars, Apache 2.0.

**Strengths:** Native boolean logic (`AddImplication`), assumption-based infeasibility analysis, state-of-the-art performance. Handles millions of variables.

**Gaps:** Infeasibility explanation is partial — `SufficientAssumptionsForInfeasibility()` returns a subset but not guaranteed minimal. No built-in MUS extraction.

### python-constraint2

Simple, pure-Python CSP solver. Easy to learn.

**Gap:** No UNSAT core, no explanation of failures, no relaxation suggestions. Only tells you "no solution found."

### MiniZinc

Elegant declarative modeling language. Very expressive for constraint specification.

**Gaps:** No explanation of failures through the Python interface. Requires separate MiniZinc installation. Better suited as a modeling language than an embedded library.

### PySAT

Powerful SAT toolkit with MUS/MCS extraction.

**Gap:** Low-level CNF encoding — requires manual translation of your domain constraints into boolean clauses. Better as a backend (via CPMpy) than a direct API.

## Graph-Based Tools

### NetworkX
Standard Python graph library. Useful for **visualizing** the constraint graph and running graph algorithms (topological sort, cycle detection), but does **not** include a constraint solver. Complementary tool, not a replacement.

### PubGrub Algorithm
Dependency resolution algorithm (used by Dart's `pub`, Rust's `cargo`, Python's `uv`) with excellent human-readable error messages. However, it's designed for **version resolution** specifically, not general constraint satisfaction. Adapting it to technology selection would require significant reworking.

## Combining Constraint Solvers with LLMs

This is an active research area. The recommended architecture:

### Two-Layer Approach

```
Layer 1: Constraint Solver (Z3 / CPMpy)
    ├── Models implies/requires/excludes/constrains as formal logic
    ├── Detects infeasibility
    ├── Extracts UNSAT core (which constraints conflict)
    └── Computes MCS (what to relax)

Layer 2: LLM Explanation (Claude Code)
    ├── Receives UNSAT core labels and constraint graph metadata
    ├── Translates into natural language: "Lambda requires AWS, which
    │   excludes Azure compute, but you selected Azure compute"
    ├── Suggests trade-offs: "Remove Lambda to unlock Azure options,
    │   or switch to AWS compute"
    └── Considers project context for recommendation ranking
```

### Relevant Research

**MCP-Solver** (Szeider, 2025, arXiv:2501.00539): Uses MCP to connect LLMs to constraint solvers (MiniZinc, PySAT, Z3). The LLM translates natural language to solver code, solves, and interprets results back. GitHub: github.com/szeider/mcp-solver.

**Logic.py** (Meta AI, 2025, arXiv:2502.15776): Python DSL that prompts LLMs to formalize problems for SAT/SMT solving. Achieved 91.4% accuracy on logic benchmarks vs. 24.9% for LLM alone.

**"I Want It That Way"** (Lawless et al., ACM TIIS 2024): Combines LLMs with CP for interactive decision support. LLM handles preference elicitation, CP handles optimization. Directly applicable pattern.

## Recommendation

**Primary: Z3 with CPMpy as the modeling front-end.**

- CPMpy provides Pythonic constraint modeling and built-in MUS/MCS tools
- Z3 as backend provides UNSAT core extraction with labeled constraints
- Claude Code (or templates) translates solver output to natural-language trade-offs
- NetworkX optionally for constraint graph visualization

This architecture is validated by the MCP-Solver paper and the "I Want It That Way" paper, and maps directly to the Agent Stacking Framework's constraint graph requirements.

Your domain (technology selection with implies/excludes/requires) is a classic **product configuration problem** — a well-studied area in constraint programming with decades of research.

## References

Microsoft Research. "Z3 Theorem Prover." *GitHub*, github.com/Z3Prover/z3.

"Programming Z3." *Stanford Theory Group*, theory.stanford.edu/~nikolaj/programmingz3.html.

CPMpy. "CPMpy: Constraint Programming and Modeling in Python." *GitHub*, github.com/CPMpy/cpmpy.

CPMpy. "UNSAT Core Extraction." *CPMpy Documentation*, cpmpy.readthedocs.io/en/latest/unsat_core_extraction.html.

Google. "OR-Tools." *GitHub*, github.com/google/or-tools.

Google. "CP-SAT Solver." *Google Developers*, developers.google.com/optimization/cp/cp_solver.

MiniZinc. "MiniZinc Python Bindings." *GitHub*, github.com/MiniZinc/minizinc-python.

PySAT. "PySAT: SAT Technology in Python." *GitHub*, github.com/pysathq/pysat.

Szeider, Stefan. "MCP-Solver: Integrating Language Models with Constraint Programming Systems." *arXiv*, 2025, arxiv.org/abs/2501.00539.

Meta AI. "Logic.py: Bridging the Gap between LLMs and Constraint Solvers." *arXiv*, 2025, arxiv.org/abs/2502.15776.

Lawless, et al. "I Want It That Way." *ACM Transactions on Interactive Intelligent Systems*, vol. 14, no. 3, 2024, doi.org/10.1145/3685053.

"Constraint Satisfaction Approach to Resolving Product Configuration Conflicts." *ScienceDirect*, sciencedirect.com/science/article/abs/pii/S1474034612000389.
