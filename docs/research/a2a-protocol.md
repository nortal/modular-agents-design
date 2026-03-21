# Google Agent-to-Agent (A2A) Protocol — Deep Dive

**Date:** 2026-03-21
**Purpose:** Evaluate A2A protocol for inter-agent communication in the Agent Stacking Framework.

## Overview

A2A is an open communication protocol (Apache 2.0) introduced by Google on April 9, 2025, enabling AI agents built on diverse frameworks to communicate as peers. Donated to the Linux Foundation in June 2025. Reached v1.0.0 on March 12, 2026.

- **GitHub:** github.com/a2aproject/A2A (22.7k stars)
- **Spec:** a2a-protocol.org/latest/specification/
- **Governance:** Linux Foundation, vendor-neutral

## What Problem It Solves

Agents built with different frameworks (LangChain, CrewAI, Google ADK, AutoGen, etc.) cannot natively communicate. A2A provides a common protocol so agents can discover each other, delegate tasks, exchange context, and collaborate — analogous to what HTTP did for the web.

## Technical Architecture

### Transport
- HTTP(S) mandatory
- Three protocol bindings: JSON-RPC 2.0, gRPC (added v0.3), HTTP/REST
- Streaming via Server-Sent Events (SSE)
- Push notifications via webhooks

### Core Operations
1. Send Message / Send Streaming Message
2. Get Task / List Tasks / Cancel Task
3. Subscribe to Task
4. Push Notification Config (create/get/list/delete)
5. Get Extended Agent Card

### Authentication
Six security scheme types: API Key, HTTP Auth (Basic/Bearer), OAuth2, OpenID Connect, Mutual TLS, Custom extensions.

## Agent Cards

A JSON document serving as a "digital business card" for an A2A-compatible agent, declaring:
- Identity (name, description, provider)
- Service endpoint URL
- Capabilities (streaming, push notifications)
- Authentication requirements
- Skills (with id, name, description, input/output modes, examples)
- Protocol version

## Discovery Mechanisms

| Method | Description |
|---|---|
| **Well-Known URI** | `https://{domain}/.well-known/agent-card.json` (primary) |
| **Curated Registries** | Intermediary services maintaining Agent Card collections (no standard API prescribed) |
| **Direct Configuration** | Hardcoded endpoints, config files, environment variables |

## Task Lifecycle

| State | Meaning |
|---|---|
| `WORKING` | Processing in progress |
| `COMPLETED` | Successfully finished (terminal) |
| `FAILED` | Processing failed (terminal) |
| `CANCELED` | User-initiated cancellation (terminal) |
| `REJECTED` | Agent rejected the request (terminal) |
| `INPUT_REQUIRED` | Awaiting client input |
| `AUTH_REQUIRED` | Awaiting client authentication |

Multi-turn conversations use `contextId` (groups related tasks) and `taskId` (specific work unit). `referenceTaskIds` link related tasks for delegation chains.

## Framework Support

| Framework | Status |
|---|---|
| Google ADK | Native (v1.0.0 stable) |
| Amazon Bedrock AgentCore | Native |
| LangChain / LangGraph | Via LangSmith Agent Server |
| CrewAI | Native |
| Microsoft Azure | Listed supporter |
| SAP Joule | Listed supporter |

Official SDKs: Python, Go, JavaScript, Java, .NET.

## A2A vs MCP

| Dimension | A2A | MCP |
|---|---|---|
| **Purpose** | Agent-to-agent (horizontal) | Agent-to-tool (vertical) |
| **Model** | Peer-to-peer | Client-server |
| **Transport** | HTTP(S), gRPC, SSE | stdio, HTTP+SSE |
| **Originated by** | Google (Apr 2025) | Anthropic (Nov 2024) |
| **Governance** | Linux Foundation | AAIF / Linux Foundation |

**These are complementary, not competing.** MCP = how agents use tools. A2A = how agents talk to each other.

## Security Concerns

| Issue | Detail |
|---|---|
| **Unsigned Agent Cards** | Spoofing risk — no enforcement of card signing |
| **OAuth weaknesses** | No mandatory token expiration, coarse-grained scopes |
| **Stream hijacking** | Concurrent streams without mandatory termination |
| **No prompt injection defense** | Attacks can spread between agents |
| **No RBAC** | No built-in role-based access control |

## Known Limitations

1. No machine-readable skill input/output schemas — semantic ambiguity
2. N-squared scaling problem for peer-to-peer connections
3. No standard registry API prescribed
4. Orchestration patterns still immature
5. Competing standards (AAIF, ACP) could fragment ecosystem
6. Few production deployments live today

## Adopters (150+ organizations)

Google Cloud, AWS, Microsoft Azure, Salesforce, SAP, ServiceNow, Cisco, PayPal, Zoom, Atlassian, MongoDB, LangChain, Cohere, Renault Group, and many more.

## Relevance to Agent Stacking Framework

**Should support A2A for inter-agent communication.** When synthesized agents need to coordinate with each other or with external agents from other frameworks, A2A is the emerging standard. However:

- Security gaps require supplementary controls
- The framework should not depend on A2A for internal agent composition — that's the constraint graph's job
- A2A is best suited for the external interface, not the internal architecture

## References

Google. "Announcing the Agent2Agent Protocol (A2A)." *Google Developers Blog*, developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/.

"A2A Protocol Specification." *A2A Protocol*, a2a-protocol.org/latest/specification/.

"Agent Discovery." *A2A Protocol*, a2a-protocol.org/latest/topics/agent-discovery/.

Google. "Google Cloud Donates A2A to Linux Foundation." *Google Developers Blog*, developers.googleblog.com/en/google-cloud-donates-a2a-to-linux-foundation/.

Linux Foundation. "Linux Foundation Launches the Agent2Agent Protocol Project." *Linux Foundation*, linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents.

IBM. "What Is Agent2Agent (A2A) Protocol?" *IBM Think*, ibm.com/think/topics/agent2agent-protocol.

Semgrep. "A Security Engineer's Guide to the A2A Protocol." *Semgrep Blog*, semgrep.dev/blog/2025/a-security-engineers-guide-to-the-a2a-protocol/.

Cloud Security Alliance. "Threat Modeling Google's A2A Protocol with the Maestro Framework." *CSA Blog*, cloudsecurityalliance.org/blog/2025/04/30/threat-modeling-google-s-a2a-protocol-with-the-maestro-framework.

"A2A and MCP." *A2A Protocol*, a2a-protocol.org/latest/topics/a2a-and-mcp/.

AWS. "Introducing Agent to Agent Protocol Support in Amazon Bedrock AgentCore Runtime." *AWS Machine Learning Blog*, aws.amazon.com/blogs/machine-learning/introducing-agent-to-agent-protocol-support-in-amazon-bedrock-agentcore-runtime/.
