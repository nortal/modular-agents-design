# Human-Minutes Tracking — Deep Dive

**Date:** 2026-03-21
**Purpose:** Evaluate approaches for tracking human time spent on AI-agent-assisted tasks.

## The Requirement

Track how much human time is required per task that an agent assists with. This serves:
- ROI measurement (agent-assisted vs. manual work)
- Cost accounting / client billing
- Quality gate input (flag cost regressions)
- Productivity analysis over time

## Key Finding

**No existing AI coding tool natively measures "human-minutes vs AI-minutes" per task.** This is a recognized gap — Cursor's community forum has an open request for exactly this feature. The Agent Stacking Framework would be novel in offering it.

## Existing Time Tracking Tools

### Developer-Specific (Most Relevant)

| Tool | Approach | API | Free Tier | Relevance |
|---|---|---|---|---|
| **WakaTime** | Automatic IDE heartbeats (keystrokes, file saves) | REST API (stats, durations, heartbeats) | Yes (limited history) | **Highest** — already has Claude Code integration |
| **Wakapi** | Self-hosted WakaTime-compatible backend | REST API, Prometheus export | Fully free (self-hosted) | **High** — full data ownership, privacy-first |
| **ActivityWatch** | Automatic window/AFK tracking, all local | Python client library (`aw-client`) | Fully free, open source | **High** — extensible via custom watchers |
| **Git Time Metric (GTM)** | Editor plugins capture editing events, stored as git notes | CLI-based, git notes | Fully free, open source | **Medium** — captures actual editing, not just commits |

### General Purpose

| Tool | Approach | API | Free Tier |
|---|---|---|---|
| **Toggl Track** | Manual timers + integrations | Full REST API | Yes (limited) |
| **Clockify** | Manual timers + integrations | REST API | Yes (unlimited users) |
| **Harvest** | Manual timers, GitHub integration | V2 REST API | Trial only |
| **RescueTime** | Passive app/website tracking | Analytics API | Yes (limited) |

## Automatic Time Inference Approaches

### IDE Activity (WakaTime Model)
Heartbeats sent on keystrokes/file saves, grouped into sessions. If no heartbeat for configurable timeout (~15 min), session ends. Tracks: file path, project, language, editor, OS, branch, category (coding/building/debugging).

**Strength:** High granularity, minimal user friction.
**Weakness:** Captures "presence" not "productive effort" — thinking time looks like idle time.

### Git-Based Inference
**git-hours:** Groups commits by time gaps into sessions. Simple but misses non-commit work.
**GTM:** Captures per-file editing events via editor plugins, stores as git notes. More accurate than commit-gap analysis.

**Strength:** No additional tooling beyond git.
**Weakness:** Research, design, code review, and discussion are invisible.

### AI Interaction Timing (Most Relevant for Our Use Case)
| Measurement | What It Captures |
|---|---|
| Time from agent response to next human action | Review/comprehension time |
| Time from human prompt to agent response | AI compute time (not human time) |
| Time human spends formulating prompts | Input composition time |
| Total session duration minus AI processing time | **Human-minutes** |

### Session-Based Tracking
Langfuse and AgentOps support session-level grouping with cost/latency breakdown per session, user, and model version.

## How Existing AI Tools Track Productivity

### GitHub Copilot
- Suggestion acceptance rate, code retention rate (88%), % of code written by Copilot (46%)
- Controlled experiment: 55% faster task completion (1h11m vs 2h41m)
- PR cycle time: 75% reduction (9.6 days → 2.4 days)
- Does NOT track human-minutes directly

### Claude Code
- Tracks "active session time" (user interactions + CLI processing, excluding idle)
- Telemetry opt-in, exportable via OpenTelemetry
- WakaTime offers Claude Code time tracking integration
- Does NOT decompose into human-minutes vs AI-minutes

### Cursor
- Weekly/monthly active users, tabs accepted, AI requests made
- Does NOT track time saved
- Community forum request exists for this exact feature

### Academic Research

**METR Study (2025):** RCT with 16 experienced developers — AI tools *increased* completion time by 19% (surprising). Revised study design announced Feb 2026 due to selection bias issues.

**Anthropic Productivity Research:** Uses Claude to estimate how long tasks would take without AI, computes savings as `1 - time_with_ai / time_without_ai`. "Promising accuracy" relative to human estimates.

**Large-Scale Production Study (2024-2025):** 300 engineers over one year — cycle time reduced 33.8% (150.5h → 99.6h), review time reduced 29.8%.

## Privacy and Accuracy Considerations

### Accuracy
- Automatic tracking eliminates recall bias and omissions from manual entry
- Legal domain: automatic tracking captured 15-30% more billable time than manual
- But captures "presence" not "productive effort" — thinking looks like idle

### Self-Reported vs Automatic
- Self-reported: Subject to recall bias, rounding, social desirability
- Automatic: More objective but may over-count (idle) or under-count (offline thinking)
- UC Berkeley 2024: Self-controlled monitoring improved productivity 31% without surveillance effects
- **Best practice:** Automatic baseline with optional human corrections

### Privacy
- Key concerns: Keystroke logging, screen capture, browsing history
- Privacy-first approaches: ActivityWatch stores everything locally; Wakapi is self-hosted
- Non-intrusive tracking teams shipped features 33% faster with 45% fewer bugs vs. surveillance-style tracking
- GDPR and labor law considerations vary by jurisdiction

### Billing for AI-Assisted Work
- No established industry standard yet
- Emerging pattern: "productivity coefficient" (60-80%) applied to raw AI-saved time
- Recommendation: Track total cost of ownership at 2.5-3x subscription cost

## Open Source Libraries (Python)

| Library | Type | Key Feature |
|---|---|---|
| **ActivityWatch** | Automatic tracker | Local-only data, extensible watchers, MPL-2.0 |
| **Wakapi** | WakaTime-compatible backend | Self-hosted, uses existing WakaTime plugins |
| **TimeTagger** | Web-based tracker | Python library with interactive UI |
| **utt** | CLI tracker | Simple, no dependencies |
| **AgentOps** | Agent observability | LLM cost/latency tracking, multi-framework |
| **Langfuse** | LLM observability | Session-level tracing, cost breakdown |

## Recommended Architecture for Agent Stacking Framework

No existing tool directly measures "human-minutes per agent-assisted task." Recommended approach combines multiple signals:

### 1. Agent-Level Instrumentation (Primary)
Emit timestamped events: `session_start`, `human_prompt_sent`, `agent_response_received`, `human_action_taken`, `task_completed`. Calculate:

```
human_minutes = total_session_time - agent_processing_time
```

### 2. IDE-Level Validation (Secondary)
Use WakaTime heartbeats (or self-hosted Wakapi) to validate that the developer was actively coding during claimed human-minutes. Catches walk-away sessions.

### 3. Git-Level Correlation (Tertiary)
Use GTM or git-hours to cross-reference time estimates with commit activity. Provides independent validation signal.

### 4. Observability Backend
Use Langfuse or AgentOps for session-level tracing with cost/latency breakdown. Export to own analytics for ROI calculation.

### 5. Privacy-First Design
Store data locally by default (like ActivityWatch). Make cloud sync opt-in. Let developers control what is tracked.

## References

WakaTime. "WakaTime API Documentation." *WakaTime*, wakatime.com/developers.

WakaTime. "Claude Code Time Tracking." *WakaTime*, wakatime.com/claude-code-time-tracking.

Wakapi. "Wakapi." *GitHub*, github.com/muety/wakapi.

ActivityWatch. "ActivityWatch." *GitHub*, github.com/ActivityWatch/activitywatch.

Git Time Metric. "GTM." *GitHub*, github.com/git-time-metric/gtm.

Kimmobrunfeldt. "git-hours." *GitHub*, github.com/kimmobrunfeldt/git-hours.

AgentOps. "AgentOps." *GitHub*, github.com/AgentOps-AI/agentops.

Langfuse. "AI Agent Observability with Langfuse." *Langfuse Blog*, langfuse.com/blog/2024-07-ai-agent-observability-with-langfuse.

GitHub. "Measuring Impact of GitHub Copilot." *GitHub Resources*, resources.github.com/learn/pathways/copilot/essentials/measuring-the-impact-of-github-copilot/.

"Measuring GitHub Copilot's Impact on Productivity." *Communications of the ACM*, cacm.acm.org/research/measuring-github-copilots-impact-on-productivity/.

Cursor. "Time Tracking for AI-Assisted Development." *Cursor Forum*, forum.cursor.com/t/time-tracking-for-ai-assisted-development/36465.

Anthropic. "Claude Code Monitoring." *Anthropic Documentation*, docs.anthropic.com/en/docs/claude-code/monitoring-usage.

Anthropic. "Estimating AI Productivity Gains." *Anthropic Research*, anthropic.com/research/estimating-productivity-gains.

METR. "Early-2025 AI Developer Productivity Study." *METR Blog*, metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/.

"Intuition to Evidence: Measuring AI's True Impact on Developer Productivity." *arXiv*, arxiv.org/html/2509.19708v1.

DX. "AI ROI Calculator." *DX Blog*, getdx.com/blog/ai-roi-calculator/.
