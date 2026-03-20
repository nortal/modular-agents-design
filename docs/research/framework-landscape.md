# Open Source Agent Framework Landscape Analysis

**Date:** 2026-03-20
**Purpose:** Evaluate existing open source agent frameworks against the Agent Stacking Framework requirements to identify gaps and opportunities.

## Comparison Matrix

| Feature | **Agent Stacking** | CrewAI | AutoGen | LangGraph | Semantic Kernel | Claude Agent SDK | OpenAI Agents SDK | DSPy | Haystack | Google ADK | MS Agent Framework |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **OO Inheritance** | Multiple | No | Yes (single) | No | Yes (single) | No | No | Single | No | No | No |
| **Multiple Inheritance** | Yes + AI conflict resolution | No | Python MRO only | No | No | No | No | No | No | No | No |
| **Constraint Graph** | Yes | No | No | Partial (edges) | No | No | No | No | No | No | No |
| **AI-Driven Synthesis** | Yes + PR review | No | No | No | No | No | No | Auto-optimize (prompts only) | No | No | No |
| **Token Tracking** | Per-agent | Via integrations | Built-in | Via LangSmith | Via OpenTelemetry | Built-in | Built-in | No | No | Via Vertex | Via Azure |
| **Clock Time** | Per-agent | Via integrations | Via AgentOps | Via LangSmith | Via OpenTelemetry | Via OTEL | Via tracing | No | No | No | Via telemetry |
| **Human-Minutes** | Yes | No | No | No | No | No | No | No | No | No | No |
| **Cross-Platform OS** | Win/Mac/Linux | Yes | Yes | Yes | Yes (C#/Py/Java) | Yes (Py/TS) | Python | Python | Python | Python | Py + .NET |
| **Git Per-Agent Repos** | Yes | No | No | No | No | No | No | No | No | No | No |
| **Human-in-the-Loop** | PR-based governance | No | No | Execution pauses | No | AskUser tool | No | No | No | No | No |
| **LLM Agnosticism** | TBD | Strong (200+) | Strong (protocol) | Strong | Strong | Anthropic + Bedrock/Vertex | OpenAI only | Strong (LiteLLM) | Strong | Google-focused | Azure-focused |

## Key Finding

None of the existing frameworks implement the core differentiators of the Agent Stacking Framework:

1. **OO multiple inheritance with AI conflict resolution** — No framework does this. AutoGen and Semantic Kernel have single inheritance. Nobody resolves conflicts via LLM.

2. **Constraint graph** — Nobody models agent interdependencies where selecting one agent restricts valid options for others. LangGraph's directed graph is control flow, not constraint satisfaction.

3. **LLM-based synthesis with PR governance** — Nobody synthesizes agents from specs via LLM and governs them through PR review. DSPy's auto-optimization is the closest analog but operates on prompts, not agent definitions.

4. **Human-minutes tracking** — Unique to this design.

5. **Per-agent git distribution** — Nobody does this. Everything is single-package distribution via PyPI/npm.

## Individual Framework Assessments

### CrewAI

Open-source Python framework (MIT License) for orchestrating autonomous, role-playing AI agents. Uses composition over inheritance — agents are configured instances of a single `Agent` class, not subclassed. Supports sequential, parallel, and conditional task execution. Strong LLM agnosticism via LiteLLM (200+ models). No constraint-aware composition. Observability via third-party integrations (AgentOps, Langtrace, SigNoz).

- **GitHub:** 46.7k stars
- **Relevance:** Role-based metaphor is intuitive; LiteLLM integration pattern worth studying for LLM agnosticism

### AutoGen (Microsoft)

Layered architecture with Core API (event-driven), AgentChat API (rapid prototyping), and Extensions. Full OO class hierarchy with subclassing — `BaseChatAgent` inherits from `ChatAgent`, `ABC`, and `ComponentBase` (multiple inheritance via Python MRO). Built-in token and cost tracking per agent. Being merged into Microsoft Agent Framework (GA Q1 2026).

- **GitHub:** github.com/microsoft/autogen
- **Relevance:** Closest to OO inheritance model; protocol-based LLM abstraction is well-designed

### LangGraph (LangChain)

Low-level orchestration framework modeling agent workflows as directed graphs. Nodes are functions, edges determine flow. Favors composition over inheritance deliberately. State reducers handle merging concurrent outputs. Strong LLM agnosticism via LangChain integrations.

- **GitHub:** ~27k stars, v1.1.3
- **Relevance:** State reducer pattern could inform constraint graph merge logic; durable execution model is strong

### Semantic Kernel (Microsoft)

Lightweight SDK with full OO class hierarchy (Agent → KernelAgent → ChatHistoryKernelAgent → ChatCompletionAgent). Supports declarative YAML agent specs (Prompty format) — git-friendly. Being merged into Microsoft Agent Framework. Available in C#, Python, and Java.

- **GitHub:** github.com/microsoft/semantic-kernel
- **Relevance:** Declarative YAML specs align with "agents as data" model; multi-language support is notable

### Claude Agent SDK (Anthropic)

Provides the same agent loop that powers Claude Code. Supports subagents with tool allowlists. Lifecycle hook system (PreToolUse, PostToolUse, Stop, SessionStart). Built-in token and cost tracking per message. Deep MCP integration.

- **Relevance:** Lifecycle hooks could inform quality gate integration points; MCP integration is important for tooling

### OpenAI Agents SDK

Successor to Swarm. Flat composition via handoffs and routines. Built-in tracing for token usage and latency. Guardrails for input/output validation. OpenAI-only.

- **Relevance:** Guardrails pattern could inform quality gate design

### DSPy (Stanford)

Declarative framework for programming (not prompting) language models. Modules inherit from `dspy.Module`. Key differentiator: automatic optimization — given examples and a metric, optimizers compile programs into effective prompts or fine-tuned weights.

- **Relevance:** Auto-optimization paradigm could inform how synthesis improves over time via feedback

### Haystack (deepset)

Pipeline-based orchestration. Components are composed in pipelines, not inherited. `SearchableToolset` reduces context window usage by letting agents search for relevant tools.

- **Relevance:** SearchableToolset concept could inform how agents discover available tools in a large stack

### Google Agent Development Kit (ADK)

Hierarchical agent tree with root agent delegating to sub-agents. Standout feature: native A2A (Agent-to-Agent) protocol support for cross-framework agent communication.

- **Relevance:** A2A protocol is the leading cross-framework interop standard; should be supported

### Microsoft Agent Framework (AutoGen + Semantic Kernel)

Public preview Oct 2025, GA target Q1 2026. Merges AutoGen's multi-agent orchestration with Semantic Kernel's enterprise features. Supports A2A and MCP protocols.

- **Relevance:** Convergence of two major frameworks; strongest enterprise integration story

## Strengths Worth Adopting

| Framework | Pattern to Learn From |
|---|---|
| **Semantic Kernel** | Declarative YAML agent specs (git-friendly) |
| **AutoGen** | Protocol-based LLM abstraction (`ChatCompletionClient`) |
| **LangGraph** | State reducers for merging concurrent outputs; durable execution |
| **CrewAI** | LiteLLM integration for 200+ model support |
| **Claude Agent SDK** | Lifecycle hooks for intercepting agent behavior; MCP integration |
| **Google ADK** | A2A protocol for cross-framework agent interop |
| **DSPy** | Auto-optimization paradigm for improving synthesis over time |

## References

CrewAI Inc. "CrewAI: Framework for Orchestrating Role-Playing, Autonomous AI Agents." *GitHub*, github.com/crewAIInc/crewAI.

CrewAI Inc. "Introduction." *CrewAI Documentation*, docs.crewai.com/en/introduction.

CrewAI Inc. "Agents." *CrewAI Documentation*, docs.crewai.com/en/concepts/agents.

CrewAI Inc. "LLMs." *CrewAI Documentation*, docs.crewai.com/en/concepts/llms.

CrewAI Inc. "CrewAI Tracing." *CrewAI Documentation*, docs.crewai.com/en/observability/tracing.

CrewAI Inc. "Installation." *CrewAI Documentation*, docs.crewai.com/en/installation.

Microsoft. "AutoGen." *GitHub*, github.com/microsoft/autogen.

Microsoft. "AutoGen AgentChat Agents Reference." *AutoGen Documentation*, microsoft.github.io/autogen/stable/reference/python/autogen_agentchat.agents.html.

Microsoft. "Model Clients." *AutoGen Documentation*, microsoft.github.io/autogen/stable/user-guide/core-user-guide/components/model-clients.html.

Microsoft. "Semantic Kernel Agent Architecture." *Microsoft Learn*, learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/agent-architecture.

Microsoft. "ADR 0032: Agents." *GitHub*, github.com/microsoft/semantic-kernel/blob/main/docs/decisions/0032-agents.md.

Microsoft. "ADR 0070: Declarative Agent Schema." *GitHub*, github.com/microsoft/semantic-kernel/blob/main/docs/decisions/0070-declarative-agent-schema.md.

Microsoft. "Introducing Microsoft Agent Framework." *Azure Blog*, azure.microsoft.com/en-us/blog/introducing-microsoft-agent-framework/.

Microsoft. "Track Your Token Usage and Costs with Semantic Kernel." *Microsoft Developer Blog*, devblogs.microsoft.com/semantic-kernel/track-your-token-usage-and-costs-with-semantic-kernel/.

LangChain. "LangGraph Overview." *LangChain Documentation*, docs.langchain.com/oss/python/langgraph/overview.

LangChain. "LangChain and LangGraph Agent Frameworks Reach v1.0 Milestones." *LangChain Blog*, blog.langchain.com/langchain-langgraph-1dot0/.

LangChain. "Cost Tracking." *LangSmith Documentation*, docs.langchain.com/langsmith/cost-tracking.

Langfuse. "Trace and Evaluate LangGraph Agents." *Langfuse Guides*, langfuse.com/guides/cookbook/example_langgraph_agents.

OpenAI. "OpenAI Agents SDK." *OpenAI Documentation*, openai.github.io/openai-agents-python/.

OpenAI. "Swarm." *GitHub*, github.com/openai/swarm.

Anthropic. "Agent SDK Overview." *Claude API Documentation*, platform.claude.com/docs/en/agent-sdk/overview.

Anthropic. "Building Agents with the Claude Agent SDK." *Anthropic Engineering Blog*, anthropic.com/engineering/building-agents-with-the-claude-agent-sdk.

Stanford NLP. "DSPy." *GitHub*, github.com/stanfordnlp/dspy.

Deepset. "Agents." *Haystack Documentation*, docs.haystack.deepset.ai/docs/agents.

Google. "Agent Development Kit Documentation." *Google ADK*, google.github.io/adk-docs/.

Google. "ADK with A2A Protocol." *Google ADK Documentation*, google.github.io/adk-docs/a2a/.
