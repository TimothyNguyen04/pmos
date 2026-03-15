# PRD: Sploink — Low-Level Agent Telemetry + Dashboard
**Version:** 2.0
**DRI:** Timothy Nguyen
**Engineers:** Ved Sharma (Sploink), Aryan Pandit (Salaxia), Navneet (CLI capture)
**Status:** Active — 4-day sprint starting March 16, 2026
**Last Updated:** March 12, 2026

---

## 1. Problem + Hypothesis

**Problem:**
Every observability tool in the market traces LLM calls. LangSmith, Langfuse, AgentOps — they tell you what prompts went in and what came out. That's not enough.

An agent can make 50 LLM calls with clean responses and still be completely stuck — touching the same files in a loop, producing no commits, making no real progress. You can't see that with call tracing. You can only see it with system-level behavioral telemetry.

**Hypothesis:**
If we capture low-level execution behavior from agent sessions (file edits, shell commands, git state, build/test output) and compute convergence, thrash, and momentum signals from that data, then developers will have the visibility they need to intervene, improve, and ultimately build better agents.

**Why we win:**
AgentOps is the closest competitor and could add low-level telemetry. Our defensible moat is not just the capture layer — it's the specific formula for convergence and thrash detection, the dynamic parameters that adapt those formulas per context, and the proprietary execution trace data that accumulates with every session. Data compounds. Formula quality compounds. First mover matters here.

**Positioning:**
Sploink is "Datadog for agents" — not at the API call level, but at the execution behavior level.

---

## 2. Who It's For

**Primary:** Power users running coding agents in the terminal. Claude Code, Cursor, GPT-4o via CLI, open-source frameworks. People who already live in the terminal and have felt the pain of an agent that "looks busy" but isn't making real progress.

**Why power users first:**
They suffer from thrashing the most. They run complex, multi-step agent sessions. They have the technical context to understand convergence signals. They're the right first audience to validate the product before expanding.

---

## 3. MVP Scope

**In scope (4-day sprint):**
- CLI wrapper — wraps any agent session, captures low-level telemetry
- Canonical event schema — the data contract everything downstream depends on
- POST /execute endpoint — Sploink's HTTP API
- Telemetry emission to Salaxia — closes the feedback loop
- Salaxia scoring — convergence, thrash, momentum (Aryan, in progress)
- Dashboard v1 — terminal UI with real-time visualizations

**Not in scope:**
- Agent marketplace
- IDE extension
- Multi-agent orchestration
- Billing or user accounts
- Web dashboard (v2)

---

## 4. The Product

### Entry Point
```bash
sploink run -- claude "refactor the auth module"
```
Drop-in. Zero behavior change. Wraps any agent command.

### What It Captures
| Signal | What it tells you |
|---|---|
| Shell commands issued | What the agent is actually doing step by step |
| File edit frequency | Is it touching the same files repeatedly? |
| Git state (diff size, commits) | Is real work accumulating? |
| Test runner output | Pass/fail trajectory over time |
| Build output | Is the codebase getting closer to working? |
| Process spawns | What tools is the agent invoking? |

### Dashboard — What It Shows

**Visualization 1: Codebase Heat Map**
A graph of the codebase showing which files the agent has touched and how frequently. Hot files = high edit frequency. The heat map shows where the agent's context is concentrated. If the same 3 files are glowing red with no commits, that's a thrash signal visible at a glance.

**Visualization 2: Token + Time Attribution**
Bar graph showing how many tokens were spent on which component of the task, and how much time each component took. Built on goal attribution — because the CLI reads the initial prompt, we can map actions to sub-goals (e.g. "create the frontend" vs "wire to the backend"). This lets us rank not just agents but the individual skills and prompts that comprise an agent.

Metrics displayed:
- Tokens saved vs. baseline
- Time saved vs. baseline
- Tokens per sub-goal
- Time per sub-goal

**Visualization 3: Convergence / Thrash / Momentum Curve**
Real-time curve showing whether the session is making progress. A healthy session ramps up. A stuck session flatlines or oscillates.
- Convergence score (0-100): git diff growing, test pass rate trending up, new functions created vs. overwritten
- Thrash flag: same files edited 5+ times without commits, circular shell command patterns, no meaningful diff growth over N minutes
- Momentum score: derivative of convergence over time — velocity of real work

**Visualization 4: Agent Path Replay**
A scrub-able timeline of every action the agent took in sequence. Like git blame but for the full session. Scroll back to see exactly when thrashing started and what triggered it. Essential for debugging bad sessions and understanding what went wrong.

**Visualization 5: Sub-Goal Progress Tracker**
If the initial prompt contains multiple sub-goals, show a progress bar per sub-goal. Example: "Create frontend: 80% converged. Wire to backend: thrashing. Create tests: not started." Power users see immediately where to intervene without reading logs.

**Visualization 6: Comparative Session View**
Side-by-side convergence curves for the same task run with different agents or prompts. This is how you rank agents in practice — not synthetic benchmarks, but real sessions on real codebases. Available once a user has 2+ sessions on the same task type.

**Visualization 7: Intervention Prompt**
When thrash is detected, surface a suggested intervention inline. Example: "Agent has edited auth.py 8 times without committing. Suggested: interrupt and re-prompt with narrower scope." This closes the detection → intervention loop. The dashboard doesn't just observe — it acts.

**Visualization 8: Live Cost Meter**
Running total of tokens spent and estimated dollar cost, updating in real time. When an agent is thrashing, it's burning money on circular LLM calls. Power users need to see this immediately. Displayed prominently — not buried in a settings panel.

**Display format:** Side panel (Granola-style) — appears alongside the agent session, no tab switching required. Real-time updates. V1 ships visualizations 1-3 + cost meter. Visualizations 4-7 are v1.5 once the core pipeline is stable.

---

## 5. Build Spec

### Layer 1: CLI Wrapper (Navneet — SPL-24)
- `sploink run -- <any agent command>`
- Hooks into: shell history, filesystem events, git, test runners, build scripts
- Streams raw events to canonical schema in real time
- Framework-agnostic — works with Claude Code, Cursor, raw Python, anything
- Reads the initial prompt to enable goal attribution downstream

### Layer 2: Canonical Event Schema (Ved — SPL-22, SPL-23)
Everything downstream depends on this. Must be finalized first.

```json
{
  "event_id": "uuid",
  "session_id": "uuid",
  "task_id": "uuid",
  "timestamp": "ISO-8601",
  "sequence_index": "number",
  "event_type": "file_edit | shell_command | git_commit | test_run | build | process_spawn",
  "source": "cli_wrapper",
  "agent": "claude-code | cursor | custom",
  "target_path": "/src/auth.py",
  "payload": {
    "lines_changed": 14,
    "cumulative_diff_size": 847,
    "command": "pytest tests/",
    "exit_code": 0
  },
  "goal_context": {
    "parent_goal": "refactor_auth_module",
    "sub_goal": "add_token_refresh",
    "goal_depth": 2
  },
  "metadata": {}
}
```

Events are append-only and immutable. Schema is the API contract between Sploink and Salaxia. Must support future computation of thrash, convergence, momentum, and churn without schema changes.

**Event taxonomy:**
- File: `file_opened`, `file_created`, `file_edited`, `file_saved`, `file_deleted`
- Diff: `diff_generated`, `diff_applied`, `diff_reverted`
- Command: `command_started`, `command_finished`, `command_failed`
- Build/Test: `build_started`, `build_finished`, `build_failed`, `test_started`, `test_finished`, `test_failed`
- Git: `git_status_checked`, `git_diff_checked`, `commit_created`, `checkout_changed`
- Error: `error_observed`, `retry_triggered`, `rollback_triggered`

### Layer 3: POST /execute Endpoint (Ved — SPL-28)
```
POST /execute
{
  "task_id": "uuid",
  "intent": "refactor_auth_module",
  "agent_id": "claude-code-v1",
  "context": { ... }
}

Response:
{
  "session_id": "uuid",
  "status": "started",
  "telemetry_stream": "/sessions/{session_id}/events"
}
```

### Layer 4: Telemetry Emission to Salaxia (Ved — SPL-29)
After each execution, Sploink emits the canonical event stream to Salaxia's `/telemetry-ingest` endpoint. Closes the feedback loop.

```
POST /telemetry-ingest
{
  "session_id": "uuid",
  "task_id": "uuid",
  "agent_id": "uuid",
  "events": [ ...canonical event stream... ]
}
```

### Layer 5: Salaxia Scoring (Aryan — SEO-13, SEO-14)
Already in progress. Ingests event stream and computes:

**Convergence score** (weighted composite):
- Git state: 40% (diff size growth, commit frequency)
- Test trajectory: 35% (pass rate trend, failure reduction)
- File churn: 25% (edit frequency per file, revert rate)

**Thrash flag:**
- Boolean + confidence score (0-1)
- Triggered by: same file edited N+ times without commit, circular command patterns, zero diff growth over T minutes

**Momentum score:**
- Derivative of convergence over time
- Positive = accelerating toward goal
- Zero = stuck
- Negative = regressing

### Layer 6: Dynamic Parameters (Ved — SPL-18)
Thrash and convergence thresholds are not static. What constitutes "editing the same file too many times" depends on project type, task complexity, developer context, and historical patterns.

Dynamic parameters adapt the formula per context:
- Per project: a refactor task has different thrash signatures than a greenfield build
- Per person: some developers naturally iterate faster
- Over time: as more sessions accumulate, the formula learns what normal looks like for a given context

Implementation: parameters are a learned model, not hardcoded rules. Start with sensible defaults, adjust based on session history. This is what separates Sploink's formula from anything AgentOps could copy quickly.

### Layer 7: Goal Attribution (CLI → Dashboard)
Because the CLI reads the initial prompt, we can parse the user's intent and map sub-goals to agent actions.

Example:
- Initial prompt: "Create a LinkedIn clone"
- Sub-goals parsed: create frontend, create backend, wire frontend to backend
- Every file edit, command, and build event is tagged with the sub-goal it belongs to

This enables:
- Ranking not just agents but the skills and prompts that comprise an agent
- Per-sub-goal token and time attribution in the dashboard
- Identifying which parts of complex tasks agents consistently struggle with

### Layer 8: Dashboard (Fullstack hire — unassigned)
Terminal UI for v1 using Rich or Textual (Python). Ships in days, fits the power user audience. No tab switching — side panel display.

Web dashboard is v2. Comes when you need to share sessions across users or demo to investors.

**Timothy to recruit:** fullstack developer with Python + React experience. Backend-first profile preferred.

---

## 6. Skills Scoring (Salaxia Extension)

Beyond scoring agents, Salaxia scores skills — the individual components that make up an agent (specific prompts, tool chains, reasoning patterns).

This is enabled by goal attribution. Once you can tag events to sub-goals, you can score how well each skill performs on each type of sub-task.

Result: a skills marketplace where demand (abstract task types like "create backend") routes to supply (proven skills), assembled into a generalized agent at runtime. Solves the cold start problem — you don't need complete agents, you need skills.

---

## 7. Success Metrics

**MVP succeeds when:**
- One complete agent session flows through CLI → telemetry → Salaxia → convergence score visible in dashboard
- Thrash detection fires correctly on at least one real thrashing session
- One power user runs a real session and says "I can see what my agent is doing"

**4-day exit criteria:**
| Day | Milestone |
|---|---|
| Day 1 (Mar 16) | Event schema finalized. CLI wrapper capturing basic events. POST /execute live. |
| Day 2 (Mar 17) | Full telemetry pipeline flowing. Salaxia receiving and scoring events. |
| Day 3 (Mar 18) | Dashboard showing convergence curve + thrash flag in real time. |
| Day 4 (Mar 19) | End-to-end: one agent session, complete event stream, scored, visualized. |

**Guardrails:**
- Telemetry must be captured on 100% of agent executions — no silent failures
- Schema must not require breaking changes to support thrash/convergence/momentum computation
- Dashboard must update in under 2 seconds from event emission

---

## 8. Risks

| Risk | Detection | Mitigation |
|---|---|---|
| Schema too narrow for future metrics | Validate against all future metric requirements before finalizing | Review schema against thrash/convergence/skills signals before Day 1 ends |
| CLI wrapper misses events in some agent frameworks | Test with Claude Code, Cursor, and raw Python on Day 1 | Log uncaptured events as `unknown_event` with raw payload |
| Dynamic parameters too naive for edge cases | Sessions with unusual thrash patterns score incorrectly | Start with conservative defaults, flag edge cases for manual review |
| Dashboard hire doesn't land in time | No visual surface for demo | Fall back to CLI output (plain text scores) for Day 4 demo |
| AgentOps adds low-level capture | Monitor their changelog weekly | Differentiation is the formula + dynamic parameters + data accumulation — not just the capture layer |

---

## 9. Long-Term Roadmap

**Phase 2 — IDE Extension + Multi-Session (Q2 2026)**
VS Code extension, editor-level events, multi-session comparison dashboard, programmatic API access for developers.

**Phase 3 — eBPF Kernel-Level Visibility (Q3-Q4 2026)**
Memory trajectory, syscall entropy, I/O loop detection, CPU distribution, process tree shape. Binary-level visibility into agent behavior. No call-level tool can get here.

**Phase 4 — Agent + Skills Marketplace (2027)**
Reputation graph live. Agents and skills ranked by real execution performance. Cold start solved by Phase 3 data volume. Agent templates from behavioral fingerprints.

**Phase 5 — Agent-Native OS (2027-2028)**
Agents as kernel-level primitives. Custom scheduler, capability-based security, marketplace-native IPC. S³ becomes the operating system for the agentic era.

---

## 10. Open Questions

- [ ] Dashboard hire: who and when? Timothy recruiting this week.
- [ ] First power user: who runs a real session on Day 4 for the demo?
- [ ] Dynamic parameters: what are the Day 1 defaults for thrash threshold (N file edits, T minutes)?
- [ ] Goal attribution: how does the CLI parse sub-goals — rule-based or LLM call?
- [ ] Navneet: is he the right person for SPL-24? Reassess by end of Day 1.
