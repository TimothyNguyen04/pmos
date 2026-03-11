# PRD: Sploink MVP — Agent Intelligence Engine

## 0) Meta

| Field | Value |
|---|---|
| **Product** | Sploink |
| **DRI** | Timothy Nguyen |
| **Stage** | Team Kickoff |
| **Last Updated** | 2026-03-11 |
| **Status** | Draft |
| **Owners** | Imran Ullah (supply layer), Ved Sharma (product + infrastructure) |

---

## 1) Problem and Hypothesis

**Problem:**
AI agents are black boxes. Teams deploy them without any way to measure whether they're working, compare one architecture against another, or know why they degrade over time. There is no infrastructure layer that quantifies agent behavior.

**Hypothesis:**
If we build a closed-loop system that executes agents, captures telemetry, computes performance metrics, and ranks agents by score, then developers will be able to make data-driven decisions about which agents to deploy — replacing guesswork with measurement.

**Strategy Fit:**
This is the moat. The telemetry layer is what turns Sploink from a tool into infrastructure. Every agent execution makes the ranking smarter, which makes routing (Salaxia) better, which drives more usage, which produces more telemetry. The compounding loop only starts if Sploink ships first.

---

## 2) Scope and Non-Goals

**In Scope (MVP):**
- Agent runtime supporting LangChain, LangGraph, CrewAI, and custom agents
- Runtime telemetry: structured events on every execution (latency, tokens, errors, retries, tool calls)
- Metrics engine: compute success rate, token efficiency, retry rate, tool dependency depth
- Benchmark engine: standardized tasks all agents must run (email drafting, document retrieval, multi-step reasoning, code generation)
- Agent scoring: weighted composite score (accuracy, latency, cost, stability)
- Agent submission system: upload code, prompt templates, tool schemas
- Dashboard: upload agents, run benchmarks, view scores, compare agents
- Backend infrastructure: API, database, telemetry storage, job queue, containerized execution

**Non-Goals:**
- Workspace/low-level telemetry (file edits, git signals, IDE events) — Phase 2
- Agent marketplace or public distribution — post-MVP
- Salaxia routing integration — separate PRD, separate team

**Tradeoffs Accepted:**
- Support one agent framework first (LangChain) on Day 1. Expand to LangGraph, CrewAI, custom on Day 2+.
- Dashboard is functional, not polished. Ranking and score visibility > visual design for MVP.
- Scoring weights will be manually configured for MVP; ML-tuned weights come later.

---

## 3) AI Behavior Contract

| Dimension | Specification |
|---|---|
| **Primary Tasks** | classify (intent), extract (telemetry signals), evaluate (agent outputs vs benchmark) |
| **Inputs** | agent execution traces, benchmark task definitions, telemetry events |
| **Latency Budget** | Benchmark run P50: <30s per task / Scoring P95: <5s |
| **Constraints** | Agent execution in sandboxed environments; no PII echoed in telemetry logs |
| **Disallowed** | Agents with no sandboxing, unrestricted filesystem access, or unlimited runtime |

**Benchmark Behavior:**

| Scenario | Input | Expected Output |
|---|---|---|
| Clean agent run | Agent executes email draft task | Telemetry emitted, score computed, result displayed |
| Agent fails mid-task | Tool call throws exception | Error captured in telemetry, retry count incremented, failure score assigned |
| Agent loops infinitely | Retry count exceeds limit | Execution terminated, thrash risk flagged in score output |

---

## 4) Product Flow

**User Flow (MVP):**
1. User uploads an agent (code, prompt template, or tool schema)
2. Platform queues the agent for benchmark execution
3. Agent runs all benchmark tasks inside sandboxed environment
4. Telemetry is captured on every execution event
5. Metrics engine computes performance signals from telemetry
6. Agent receives a composite score with breakdown
7. User sees score on dashboard and can compare against other agents

**Core Loop:**
```
Upload → Benchmark → Telemetry → Metrics → Score → Dashboard
```

**Key Interactions:**
- **Agent submission:** Accepts code uploads, prompt templates, and tool schemas. Auto-triggers benchmarking on submission.
- **Benchmark execution:** All agents run the same standardized tasks. Results are deterministic and reproducible.
- **Score breakdown:** Users see overall score plus component scores (capability, latency, stability, cost).
- **Agent comparison:** Side-by-side view of two or more agents on the same benchmark.

---

## 5) Success Metrics

**Primary Metric:**
- MVP succeeds when a user can upload an agent, receive a score, and compare it against another agent in a single session.

**Quantitative Targets (4-Day Sprint):**
- At least 3 different agent architectures benchmarked end-to-end
- Telemetry emitted on 100% of agent executions (no silent failures)
- Score computed in <5s after benchmark run completes
- Dashboard displays score breakdown without manual intervention

**Guardrail Metrics:**
- No agent execution escapes the sandbox
- No telemetry stored with user PII
- Benchmark tasks produce consistent scores on repeated runs (variance <5%)

**Kill Criteria:**
If telemetry is missing on >10% of executions or scores are non-reproducible across identical runs, halt and fix the pipeline before continuing.

---

## 6) 4-Day Execution Plan

### Day 1 — Contracts + Skeleton

| Owner | Deliverable |
|---|---|
| Timothy | Telemetry schema, agent card schema, agent taxonomy, evaluation metric definitions, benchmark task definitions |
| Imran | Agent runtime skeleton: execute one LangChain agent, capture execution info, emit structured telemetry |
| Ved | Backend skeleton: API structure, database schema, telemetry storage table, job execution framework |

**Exit criteria:** One agent runs a task. Telemetry is emitted and stored.

### Day 2 — Telemetry Pipeline

| Owner | Deliverable |
|---|---|
| Imran | Full telemetry instrumentation: every execution records agent_id, task_type, tokens, latency, tool_calls, errors, retries |
| Ved | Telemetry DB + API: ingestion endpoint, structured storage, query API for metrics engine consumption |
| Timothy | Benchmark task design: define 4 evaluation tasks, each with expected output criteria and scoring rubric |

**Exit criteria:** Every execution produces queryable, structured telemetry. Benchmark tasks defined and ready.

### Day 3 — Metrics + Ranking

| Owner | Deliverable |
|---|---|
| Imran | Metrics engine: compute success rate, token efficiency, latency per task, retry rate, tool dependency depth |
| Ved | Dashboard skeleton: upload agents, view benchmark results, see rankings and score comparisons |
| Timothy | Agent scoring formula: define weights for accuracy, latency, cost efficiency, stability |

**Exit criteria:** Telemetry converts into scores. Scores display on dashboard.

### Day 4 — End-to-End Loop

| Owner | Deliverable |
|---|---|
| Imran | Benchmark execution: all agents run through all tasks with telemetry recorded throughout |
| Ved | Final UI: complete user flow — upload → benchmark → view telemetry → see metrics → compare rankings |
| Timothy | Insight layer: architecture recommendations output (thrash risk, latency inefficiencies, cost problems) |

**Exit criteria:** A user can upload an agent, run benchmarks, receive a score, and compare with another agent without manual intervention.

---

## 7) Risks

| Risk | Detection | Fallback |
|---|---|---|
| Agent sandbox escape | Monitor system calls and filesystem access during execution | Kill process immediately, block agent, log incident |
| Telemetry silent failure | Check telemetry count == expected execution count after every run | Alert + halt pipeline; do not compute scores on incomplete data |
| Benchmark inconsistency | Run same agent 3x, compare scores; flag if variance >5% | Investigate prompt sensitivity and LLM temperature settings |
| Infrastructure overload | Monitor job queue depth and execution latency | Rate-limit submissions; queue overflow alert |

---

## 8) Owners and Next Steps

**DRI:** Timothy Nguyen
**Eng:** Imran Ullah (supply layer), Ved Sharma (product + infra)

**Open Questions:**
- [ ] What LLM provider powers the benchmark evaluation tasks? (OpenAI vs Anthropic vs local model) -- Imran
- [ ] What cloud provider for containerized execution? (GCP, AWS, or local for MVP) -- Ved
- [ ] How do we handle agents that require API keys or secrets during execution? -- Timothy + Imran
- [ ] What's the canonical benchmark task format? (JSON schema for task definition) -- Timothy

**Next Milestones:**

| Target | Milestone | Exit Criteria |
|---|---|---|
| Day 1 end | Contracts locked, runtime skeleton running | Telemetry emitted from one agent execution |
| Day 2 end | Telemetry pipeline live | Every execution produces queryable structured data |
| Day 3 end | Scores computing | Dashboard shows agent score with breakdown |
| Day 4 end | Full loop working | Upload → benchmark → score → compare in one session |

---

## 9) Appendix

### Telemetry Schema (Runtime)

```json
{
  "agent_id": "string",
  "task_id": "string",
  "task_type": "email_draft | retrieval | reasoning | code_gen",
  "latency_ms": "number",
  "tokens_used": "number",
  "tool_calls": ["string"],
  "errors": ["string"],
  "retry_count": "number",
  "success": "boolean",
  "timestamp": "ISO-8601"
}
```

### Agent Card Schema

```json
{
  "agent_id": "string",
  "agent_name": "string",
  "framework": "langchain | langgraph | crewai | custom",
  "capabilities": ["summarization", "retrieval", "coding", "email_generation"],
  "tools": ["string"],
  "version": "string",
  "submitted_by": "string",
  "submitted_at": "ISO-8601"
}
```

### Scoring Formula (v1)

```
composite_score = (
  accuracy_weight * accuracy_score +
  latency_weight * latency_score +
  cost_weight * cost_score +
  stability_weight * stability_score
)

Default weights (v1):
  accuracy:  0.40
  latency:   0.25
  cost:      0.20
  stability: 0.15
```

### Benchmark Tasks (v1)

| Task | Description | Evaluation Signal |
|---|---|---|
| Email draft | Given thread context, draft a professional reply | Accuracy vs rubric, latency, token count |
| Document retrieval | Given a query, retrieve and summarize the most relevant doc | Recall accuracy, latency |
| Multi-step reasoning | Given a scenario, produce a step-by-step decision | Logical consistency, completion rate |
| Code generation | Given a spec, generate a working function with tests | Pass rate on test suite, token efficiency |

### Phase 2 Scope (Not MVP)

- Low-level workspace telemetry (file edits, git signals, IDE events via CLI wrapper)
- Thrash detection and convergence scoring (requires workspace telemetry)
- Agent momentum signals
- IDE extension for deeper telemetry capture
- Salaxia routing integration
- Agent marketplace

### Changelog

| Date | Change | Who |
|---|---|---|
| 2026-03-11 | Initial draft | Timothy |
