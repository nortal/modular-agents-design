# Model Context Protocol (MCP) — Deep Dive

**Date:** 2026-03-21
**Purpose:** Evaluate MCP for tool integration in the Agent Stacking Framework.

## Overview

MCP is an open protocol created by Anthropic (November 2024) that standardizes how LLM applications integrate with external data sources, tools, and services. Often described as "USB-C for AI" — it reduces M models x N tools integrations to M + N implementations.

- **Spec:** modelcontextprotocol.io/specification/2025-11-25
- **Governance:** Agentic AI Foundation (AAIF) under Linux Foundation (Dec 2025)
- **Ecosystem:** 10,000-20,000+ servers, 97M+ monthly SDK downloads

## Technical Architecture

### Wire Format
JSON-RPC 2.0 for all messages.

### Three-Tier Architecture
- **Hosts**: LLM applications that initiate connections (Claude Desktop, IDEs)
- **Clients**: Connectors within hosts maintaining 1:1 server connections
- **Servers**: Services providing context and capabilities

Inspired by the Language Server Protocol (LSP) used in IDEs.

### Transport
| Transport | Use Case |
|---|---|
| **stdio** | Local integrations, CLI tools, subprocess communication |
| **Streamable HTTP** | Remote/SaaS integrations, network services (replaced SSE in 2025-11-25 spec) |

### Connection Lifecycle
1. Client sends `initialize` with protocol version and capabilities
2. Server responds with its capabilities
3. Client sends `initialized` notification
4. Normal message exchange
5. Either side can terminate

## Core Primitives

### Server-Side
| Primitive | Purpose |
|---|---|
| **Tools** | Functions the AI model can execute (e.g., `search_files`, `run_query`) |
| **Resources** | Contextual data for reading via URIs (e.g., `file:///path`, `postgres://db/table`) |
| **Prompts** | Templated messages/workflows for users (appear as slash commands in UIs) |

### Client-Side
| Primitive | Purpose |
|---|---|
| **Sampling** | Servers request LLM completions through the client (enables agentic loops) |
| **Roots** | Servers query filesystem/URI boundaries they should operate within |
| **Elicitation** | Servers request information directly from user through client UI |

## Framework Support

### AI Platforms
Claude Desktop, Claude Code, Claude.ai, OpenAI ChatGPT (desktop), Google Gemini, Microsoft Copilot

### IDEs
VS Code (native), Cursor (first-class), Windsurf/Codeium, Zed, JetBrains

### Agent Frameworks
LangChain (`langchain-mcp-adapters`), Strands Agents (AWS, built around MCP), CrewAI (via bridge)

### SDKs
Official TypeScript and Python SDKs. Community SDKs in Rust, Go, Java, C#, and others. FastMCP is a popular community framework for rapid server development.

## Server Ecosystem

10,000-20,000+ MCP servers including:
- **Developer tools:** GitHub, Docker, Sentry, Postman
- **Databases:** PostgreSQL, SQLite
- **Productivity:** Slack, Notion, Atlassian (Jira/Confluence)
- **Cloud:** AWS, Cloudflare, Vercel
- **Design:** Figma
- **Payments:** Stripe
- **Browser automation:** Playwright, Puppeteer
- **Search:** Brave Search
- **Filesystem:** Local file read/write

### Discovery
- **Official registry:** registry.modelcontextprotocol.io (preview, Sept 2025) — acts as a metaregistry
- **Community registries:** mcp.so (16,000+ servers), PulseMCP (5,500+), Smithery, Glama

## Authentication & Security

### Authorization Framework (2025-11-25 spec)
- OAuth 2.1 for remote servers
- PKCE mandatory for all clients
- Resource Indicators (RFC 8707) required
- Client ID Metadata Documents for self-describing clients

### Local Servers (stdio)
No OAuth needed — inherits user's local permissions.

### Security Concerns
| Issue | Detail |
|---|---|
| **66% of scanned servers had security findings** | AgentSeal study of ~1,800 servers |
| **Tool poisoning** | Malicious tool descriptions can manipulate LLM behavior |
| **Cross-server tool shadowing** | One server's tool can impersonate another's |
| **No message signing** | No built-in tamper detection |
| **Quality variance** | With 10K+ community servers, quality varies wildly |

## Governance

- Created by Anthropic, November 2024
- Donated to Agentic AI Foundation (AAIF) under Linux Foundation, December 2025
- Co-founders: Anthropic, Block, OpenAI
- Platinum members: AWS, Anthropic, Block, Bloomberg, Cloudflare, Google, Microsoft, OpenAI
- Technical direction via SEP (Specification Enhancement Proposal) process

## MCP vs A2A

| Dimension | MCP | A2A |
|---|---|---|
| **Purpose** | Agent-to-tool (vertical) | Agent-to-agent (horizontal) |
| **Model** | Client-server | Peer-to-peer |
| **Analogy** | "Gives the mechanic tools" | "Lets mechanics talk to each other" |

**Complementary, not competing.** Most production multi-agent systems in 2026 use both.

## How Agent Stacking Framework Would Use MCP

### As an MCP Client (consuming tools)
- Synthesized agents connect to existing MCP servers for capabilities (database access, API calls, file operations, web search)
- Each agent can have different MCP server configurations based on its role

### As an MCP Server (exposing capabilities)
- The framework could expose its agents as MCP servers, allowing Claude Desktop, VS Code, and Cursor to invoke them

### Combined with A2A
```
[Orchestrator Agent]
    |-- MCP Client --> [GitHub MCP Server]
    |-- MCP Client --> [Database MCP Server]
    |-- MCP Client --> [Custom Domain MCP Server]
    |-- A2A --> [Specialist Agent 1 (with its own MCP connections)]
    |-- A2A --> [Specialist Agent 2 (with its own MCP connections)]
```

## Relevance to Agent Stacking Framework

**Should support MCP for tool integration.** Synthesized agents should be able to:
1. Consume existing MCP servers as tools (massive ecosystem leverage)
2. Optionally expose themselves as MCP servers (interop with IDEs/Claude)
3. Declare MCP server dependencies in their constraint declarations

## References

Anthropic. "Introducing the Model Context Protocol." *Anthropic News*, anthropic.com/news/model-context-protocol.

"MCP Specification 2025-11-25." *Model Context Protocol*, modelcontextprotocol.io/specification/2025-11-25.

Anthropic. "Donating MCP and Establishing the Agentic AI Foundation." *Anthropic News*, anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation.

"One Year of MCP." *MCP Blog*, blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/.

"Introducing the MCP Registry." *MCP Blog*, blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/.

"MCP Adoption Statistics." *MCP Manager*, mcpmanager.ai/blog/mcp-adoption-statistics/.

"MCP Server Security Findings." *AgentSeal*, agentseal.org/blog/mcp-server-security-findings.

Red Hat. "Model Context Protocol (MCP): Understanding Security Risks and Controls." *Red Hat Blog*, redhat.com/en/blog/model-context-protocol-mcp-understanding-security-risks-and-controls.

Auth0. "MCP vs A2A." *Auth0 Blog*, auth0.com/blog/mcp-vs-a2a/.

InfoQ. "MCP: Universal Connector for Modular AI Agents." *InfoQ Articles*, infoq.com/articles/mcp-connector-for-building-smarter-modular-ai-agents/.
