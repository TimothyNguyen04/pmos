# Sploink: Low-Level Agent Telemetry + Dashboard
**Meeting Prep — Timothy + Ved | March 12, 2026**

---

## Meeting Agenda (30 min)

1. Align on the product (10 min) — what are we actually building and who's it for
2. Lock the MVP scope (10 min) — what ships first, what gets cut
3. Assign the dashboard (5 min) — who builds it, what does v1 show
4. Set the week's critical path (5 min) — what blocks Sunday and what doesn't

---

## The Insight

Every observability tool in the market traces LLM *calls*. LangSmith, Langfuse, AgentOps — they all tell you what prompts went in and what came out.

Nobody measures what the agent *actually did* at the execution layer.

That's the gap. An agent can make 50 LLM calls with clean responses and still be completely stuck — touching the same files in a loop, producing no commits, making no real progress. You can't see that with call tracing. You can only see it with system-level behavioral telemetry.

**Sploink is "Datadog for agents" — not at the API call level, but at the execution behavior level.**

---

## The Product (Phase 1 MVP)

### What it is
A CLI wrapper + dashboard for power users who run agents in their terminal. Drop-in, zero behavior change required.

```bash
sploink run -- claude "refactor the auth module"
```

That's it. Sploink wraps the session, captures everything happening at the system level, and surfaces a live dashboard showing whether the agent is making progress or spinning.

### Who it's for
Power users running coding agents (Claude Code, Cursor, GPT-4o via CLI, open-source frameworks). People who already live in the terminal and have felt the pain of an agent that "looks busy" but isn't making real progress.

### What it captures
| Signal | What it tells you |
|---|---|
| Shell commands issued | What the agent is actually doing step-by-step |
| File edit frequency | Is it touching the same files repeatedly? |
| Git state (diff size, commits) | Is real work accumulating? |
| Test runner output | Pass/fail trajectory over time |
| Build output | Is the codebase getting closer to working? |
| Process spawns | What tools is the agent invoking? |

### What the dashboard shows
Three core signals, visible in real time:

**1. Convergence score** — is the agent getting closer to the goal? Measured by: git diff growing meaningfully, test pass rate trending up, new files/functions being created vs. overwritten.

**2. Thrash detection** — is the agent stuck in a loop? Triggered by: same files edited 5+ times without commits, circular shell command patterns, no meaningful diff growth over N minutes.

**3. Momentum curve** — velocity of real work over time. A healthy session looks like a ramp. A stuck session flatlines or oscillates.

---

## Build Spec

### What needs to exist for MVP

**Layer 1: CLI Wrapper (Navneet — SPL-24)**
- Wraps any shell command: `sploink run -- <agent command>`
- Hooks into shell history, file system events, git, test runners
- Streams raw events to the canonical schema
- Must be framework-agnostic. Works with Claude Code, raw Python scripts, anything.

**Layer 2: Canonical Event Schema (Ved — SPL-22, SPL-23)**
This is the most important piece. Everything downstream depends on it.

```json
{
  "session_id": "uuid",
  "task_id": "uuid",
  "timestamp": "ISO-8601",
  "event_type": "file_edit | shell_command | git_commit | test_run | build",
  "payload": {
    "path": "/src/auth.py",
    "lines_changed": 14,
    "cumulative_diff_size": 847
  },
  "agent": "claude-code | cursor | custom",
  "metadata": {}
}
```

Events must be append-only and immutable. The schema is the API contract between Sploink and Salaxia.

**Layer 3: POST /execute Endpoint (Ved — SPL-28)**
Sploink's HTTP API. Receives task context from Salaxia, returns execution result + telemetry stream reference.

```
POST /execute
{
  "task_id": "uuid",
  "intent": "refactor_auth_module",
  "agent_id": "claude-code-v1",
  "context": { ... }
}
```

**Layer 4: Telemetry Emission to Salaxia (Ved — SPL-29)**
After each execution, Sploink emits the canonical event stream to Salaxia's `/telemetry-ingest` endpoint. This closes the feedback loop.

**Layer 5: Salaxia Scoring (Aryan — SEO-13, SEO-14)**
Already in progress. Ingests the event stream and computes:
- Convergence score (weighted: git state 40%, test trajectory 35%, file churn 25%)
- Thrash flag (boolean + confidence)
- Momentum score (derivative of convergence over time)

**Layer 6: Dashboard (unassigned — needs decision)**
The user-facing surface. Two options:

*Option A: Terminal UI (faster, fits the audience)*
Built with something like Rich or Textual (Python). Runs in-terminal alongside the agent session. Shows live convergence curve, thrash alerts, momentum score. Ships in days not weeks.

*Option B: Web dashboard (more shareable, better for demos)*
React frontend, hosted. Takes longer but gives you a URL to share with early users and investors. Better for the eventual marketplace surface.

**Recommendation: ship terminal UI for v1, web for v2.** Power users prefer terminal. Web comes when you have multiple sessions to compare.

---

## What We're NOT Building in Phase 1

- Agent marketplace (no data yet, cold start unsolvable)
- IDE extension (Phase 2 — after CLI is proven)
- Multi-agent orchestration UI
- Billing/payments
- User accounts (sessions are local or keyed by API token)

---

## Long-Term Roadmap

### Phase 2: IDE Extension + Multi-Session Intelligence + Data Fusion (Q2 2026)
Once the CLI has traction, extend capture deeper:
- VS Code extension for editor-level events (cursor position, symbol search, file saves)
- Multi-session dashboard: compare agent runs across tasks, see which agents perform best on which task types
- Alerting: "your agent has been thrashing for 8 minutes — consider interrupting"
- API access: developers can query their own session data programmatically
- **OTel ingest endpoint:** accept high-level traces from LangSmith, Arize, and Langfuse. Correlate against Sploink's system-level stream to detect lie/no-op/loop conditions that neither layer can see alone. Positions Sploink as the correlation layer, not a replacement for existing tools.

### Phase 3: eBPF Probes — Kernel-Level Visibility (Q3-Q4 2026)
The step-change in signal quality. No OS modification required.

New metrics that become available:
- Memory trajectory (heap growth = thrash signal before it's visible at file level)
- Syscall entropy (narrowing = converging, expanding = exploring/lost)
- I/O loop detection (same file read 200x = retrieval loop)
- CPU time distribution (% inference vs. I/O vs. wait)
- Process tree shape (zombie processes = architectural quality signal)

This is where the data moat gets real. No call-level tracing tool can get here. This is binary-level visibility into what agents actually do.

### Phase 4: Agent Marketplace (2027)
By this point you have execution histories on hundreds of agents across thousands of sessions. The reputation graph is live. Now the marketplace is defensible.

**What the marketplace enables:**
- Agents are ranked by real behavioral performance, not self-reported benchmarks
- Buyers search for "best agent for refactoring Python codebases >50k LOC" — ranked by convergence scores on similar tasks
- Agents with proven track records get more routing, earn more. Cold start solved by Phase 3 data accumulation.
- Agent templates: behavioral fingerprints from top-performing sessions become starting points for new agents

**The flywheel:**
More sessions → better reputation data → better routing → better agent outcomes → more sessions.

### Phase 5: Agent-Native OS (2027-2028)
Ved's insight from March 11. Agents as kernel-level primitives, not applications. Custom scheduler with agent-aware resource allocation, capability-based security, marketplace-native IPC.

This is the full thesis: S³ becomes the operating system layer for the agentic era. Every agent runs on S³ infrastructure. Every execution generates telemetry. The marketplace is the app store.

Nobody is building toward this. Everyone treats agents as applications. We treat them as processes.

---

## The Competitive Moat (Why This Is Hard to Copy)

1. **Data moat**: execution trace data accumulates with every session. LangSmith can add system-level capture tomorrow — they can't add two years of behavioral training data.

2. **Schema moat**: whoever defines the canonical event schema for agent telemetry becomes the standard. First mover matters here. OpenTelemetry for agents.

3. **Routing moat**: Salaxia's reputation graph gets smarter with every execution. A new entrant starts with cold-start reputation. We start with history.

4. **eBPF moat**: kernel-level visibility requires deep systems expertise. This isn't a product sprint, it's a 12-month engineering investment. High barrier.

---

## Open Questions for Today's Meeting

1. **Dashboard:** terminal UI or web for v1? Who owns it?
2. **Event schema:** can Ved finalize SPL-22/23 by tomorrow so Navneet and Aryan can build against it?
3. **First user:** do we have one power user in mind to test the CLI wrapper on a real session this week?
4. **Demo for Sunday:** what's the exact scenario we show? One agent, one task, one session visible in the dashboard?
5. **Sploink website:** does it need to be updated to reflect the B2B pivot before Sunday, or is it internal only?
