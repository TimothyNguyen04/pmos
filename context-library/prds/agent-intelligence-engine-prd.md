# Agent Intelligence Engine — Salaxia + Sploink MVP Spec
*Last updated: 2026-03-11 | Source: ChatGPT spec, reviewed and adopted by Timothy*

---

## Product Goal

Build an **Agent Intelligence Engine** that measures, ranks, and improves AI agents before they are deployed at scale.

The system analyzes an agent's structure, runtime behavior, and architectural risks. It produces an overall agent score, risk analysis, cost profile, and architecture optimization recommendations.

This engine becomes the foundation infrastructure layer for agent ecosystems, enabling routing, orchestration, and agent evolution.

---

## Core Product Loop

```
Agent execution
→ Telemetry capture
→ Agent measurement
→ Agent ranking
→ Agent improvement
```

If this loop works, the system becomes self-improving infrastructure for agents.

---

## System Layers

| Layer | Owner | Focus |
|---|---|---|
| Supply Layer | Imran | Agent execution and measurement |
| Protocol Layer | Aryan | Routing, orchestration, graph logic |
| Product Layer | Ved | User interface and infrastructure |
| Architecture / Product Direction | Timothy | System design, evaluation design, narrative |

---

## Team Responsibilities

### Imran Ullah — Agent Intelligence Engine (Supply Layer)

**Agent Runtime**
- Support execution of LangChain, LangGraph, CrewAI, and custom agent architectures
- Allow agents to run tasks, call tools, produce outputs while capturing execution traces

**Telemetry Instrumentation**
Every agent execution must generate structured telemetry:
- agent_id, task_type, tokens_used, latency, tool_calls, errors, retry_loops

**Agent Metrics Engine**
Core metrics: success rate, token efficiency, latency, retry rate, hallucination detection, tool dependency depth
Advanced signals: thrash detection, momentum signals, convergence detection, architecture stability

**Benchmark Engine**
Standardized evaluation tasks all agents must run:
- Email drafting, API tool usage, document retrieval, multi-step reasoning, code generation
- Scores: capability score, latency score, stability score, cost score

**Hybrid Ranking Model**
Combines: capability match, benchmark performance, execution feedback, historical success rates

**Security and Sandboxing**
Agents run inside secure sandboxed environments with isolation and runtime limits

**Low-Level Workspace Telemetry**
Captures fine-grained workspace events to measure whether agents are converging or thrashing:

Event Schema (each event):
- event_id, timestamp, session_id, task_id, agent_id, workspace_id, event_type, source, target_path, payload, sequence_index
- Optional: correlation_id, parent_event_id, environment_type, git_commit_hash, branch_name

Event Types:
- File: file_opened, file_created, file_edited, file_saved, file_deleted, file_renamed
- Diff: diff_generated, diff_applied, diff_reverted
- Command: command_started, command_finished, command_failed
- Build/Test: build_started/finished/failed, test_started/finished/failed
- Verification: lint_started/finished, verification_failed/passed
- Git: git_status_checked, git_diff_checked, commit_created, checkout_changed
- Error/Recovery: error_observed, retry_triggered, rollback_triggered

Capture strategy: Start with CLI wrapper (wrap commands, capture git state, diffs, build/test outputs). Schema must be extensible to IDE extension later.

Metrics this enables: thrash detection (repeated edits to same file), convergence (shrinking errors, fewer touched files), agent momentum (rate of positive progress), editing churn (total edits per file without validation success)

---

### Aryan Pandit — Routing and Protocol Layer

**Intent Graph**
- User tasks → structured intent nodes representing capabilities required
- Example: "draft follow-up email" → intent node: {summarization, email_generation, contact_retrieval}

**Candidate Intent Generation (IMPORTANT)**
- LLM generates multiple candidate intents from context, not just one
- Example: "Are you free Tuesday?" → {schedule_meeting: 0.82, draft_email: 0.63, summarize_thread: 0.12}

**Intent Ranking / Scoring**
- Score from: LLM confidence, rules, user history, context signals, telemetry, recency, frequency
- Apply threshold (e.g. 0.7) to select winning intent → router picks agent

**Agent Routing Engine**
- Routing uses: capability matching, vector similarity, historical performance, agent reputation
- Most reliable agents selected automatically

**Multi-Agent Workflow System**
- Pipelines: e.g. summarize meeting → extract contacts → draft email → schedule reminder

**Reputation Graph**
- Agents accumulate reputation: success rate, cost efficiency, stability, user feedback
- Reputation influences routing

**Failure and Bias Detection**
- Detect: failing agents, looping agents, unstable behavior, biased outputs

**Agent Ranking and Evaluation Layer**
- Consumes telemetry events → computes metrics → produces agent scores
- Outputs: performance scores, rankings per task, benchmark comparisons
- Feeds into routing and marketplace

---

### Ved Sharma — Product and Infrastructure Layer

**Dashboard UI**
- Run agents, upload agents, view rankings, compare performance
- Accessible to technical and non-technical users

**Agent Submission System**
- Upload: agent code, prompt templates, tool schemas, configuration files
- Platform auto-benchmarks and evaluates on submission

**Agent Template Registry**
- Templates are reusable architectural patterns (model config, prompt structure, tool integrations, memory, orchestration logic)
- Registry: create, version, store, query templates
- Templates are deployable blueprints → execute through runtime → collect telemetry

**Backend Infrastructure**
- API services, authentication, data storage, job queues, telemetry storage, containerized execution environments

**Deployment and Scaling**
- Container orchestration, cloud infra, autoscaling workers

---

### Timothy Nguyen — System Architecture and Product Direction

**Day 1 Contracts (must define first):**
- Telemetry Schema (canonical fields for all telemetry events)
- Agent Card Schema (how agents are described)
- Agent Categories / Taxonomy: communication, analysis, coding, retrieval agents
- Evaluation Metric Definitions (what metrics matter, their weights)
- Benchmark Task Definitions (email drafting, retrieval, multi-step reasoning, code gen)

**Agent Scoring Formula**
- Weighted aggregation: accuracy weight, latency weight, cost efficiency weight, stability weight

**Insight Layer**
- Architecture recommendations: thrash risk, latency inefficiencies, tool dependency problems, cost inefficiencies

**Product Narrative**
- Why agents must be measurable
- Why ranking agents matters
- Why enterprises will trust a system that quantifies agent performance

---

## 4-Day MVP Execution Plan

**Goal: Prove the closed loop works**
`Agent Execution → Telemetry → Measurement → Ranking → Insight`

### Day 1 — Contracts + Runtime Skeleton
| Who | What |
|---|---|
| Timothy | Define telemetry schema, agent card schema, agent categories, metric definitions, benchmark tasks |
| Imran | Agent runtime skeleton — execute one agent (LangChain), capture execution info, produce telemetry |
| Aryan | Agent registry — store/discover agents by ID, define agent identity structure |
| Ved | Backend skeleton — API structure, database, telemetry storage, job execution framework |

### Day 2 — Telemetry Pipeline
| Who | What |
|---|---|
| Imran | Expand telemetry — every execution records: agent_id, task_type, tokens, latency, tool_calls, errors, retries |
| Aryan | Capability mapping — agents described by capabilities (summarization, retrieval, coding, email) |
| Ved | Telemetry DB + API — ingestion endpoint, database storage, query API |
| Timothy | Benchmark design — define evaluation tasks measuring accuracy, latency, token usage, reliability |

### Day 3 — Metrics + Ranking Engine
| Who | What |
|---|---|
| Imran | Metrics engine — compute success rate, token efficiency, latency, retry rate, tool dependency depth |
| Aryan | Ranking engine — combine benchmark perf + capability match + execution feedback → single agent score |
| Ved | Dashboard skeleton — upload agents, view benchmark results, see ranking comparisons |
| Timothy | Agent scoring formula — weights for accuracy, latency, cost, stability |

### Day 4 — End-to-End Product Loop
| Who | What |
|---|---|
| Imran | Benchmark execution — run all agents through benchmarks, record telemetry |
| Aryan | Reputation update — use benchmark results to update agent reputation scores |
| Ved | Final UI — full user flow: upload → run benchmarks → view telemetry → see metrics → compare rankings |
| Timothy | Insight layer — produce architecture recommendations (thrash risk, latency issues, tool dependency problems, cost inefficiencies) |

---

## MVP Success Criteria

The MVP succeeds if a user can:
1. Upload an agent
2. Run benchmark tests
3. Receive an agent score
4. Compare agents by performance

Example output:
- Overall agent score
- Latency profile
- Cost estimate
- Thrash risk
- Architecture improvement suggestions

---

## Long-Term Architecture (7 Layers)

```
Layer A — Perception
Raw context from integrations, ambient data, OCR, meetings, UI activity

Layer B — Routing Intent
Small deterministic task labels for routing
(draft_email, schedule_meeting, summarize_thread, update_crm, prepare_brief, create_note, remind_later)

Layer C — Goal Intent
Higher-level inferred outcomes that decompose into routing intents
(plan_meeting → schedule_meeting + draft_email)

Layer D — Supply
Agents, templates, workflows, marketplace components

Layer E — Execution + Telemetry
Run, measure, rank, improve

Layer F — Artifacts + Memory
Persist useful outputs and learned context

Layer G — Human-in-the-Loop
Confirm, approve, correct at low-confidence points
```

**MVP implements: A, B, D, E only. C (Goal Intent) comes later.**

**Key principle:** High-level goals decompose into routing intents, not directly to agents.

```
context → goal intent → routing intents → agents/workflows → telemetry → artifacts/memory
```

---

## Correct Intent Pipeline

```
context ingestion
→ candidate intent generation (LLM produces multiple candidates)
→ intent ranking / scoring (confidence scores per candidate)
→ threshold + router (select winning intent, pick best agent)
→ agent execution
→ telemetry + artifacts
```

Telemetry must log: context, predicted intent, chosen agent, result, user correction — this is how routing improves over time.

---

## What This MVP Proves

If this loop works, the foundation exists for:
- Agent marketplaces
- Agent routing systems
- Autonomous agent orchestration
- Enterprise AI infrastructure

**Core idea:** Agents should not be black boxes. They should be measurable, comparable, and continuously improving infrastructure.
