# PRD: Salaxia MVP — Routing, Protocol, and Agent Ranking
*Aryan Pandit | Status: In Progress*

## 0) Meta

| Field | Value |
|---|---|
| **Product** | Salaxia |
| **DRI** | Timothy Nguyen |
| **Engineer** | Aryan Pandit |
| **Stage** | Team Kickoff |
| **Last Updated** | 2026-03-11 |
| **Status** | Draft |

---

## 1) Problem and Hypothesis

**Problem:**
When a user gives the system a task, there is no intelligent layer that decides which agents should handle it, in what order, and based on what evidence. Without this layer, agent selection is arbitrary — the system cannot improve routing over time, cannot detect failing agents, and cannot coordinate multi-agent workflows.

**Hypothesis:**
If we build a routing and evaluation layer that converts user context into ranked candidate intents, selects the best agents based on capability and reputation, and continuously updates agent rankings from telemetry, then routing decisions will improve with every execution — creating a compounding intelligence layer that gets smarter the more it's used.

**Strategy Fit:**
Salaxia is the brain of the stack. Seoro captures context. Sploink measures agents. Salaxia is the layer that connects them — deciding what gets done and by whom. Without it, Seoro and Sploink are disconnected. With it, the system behaves like one coherent product.

---

## 2) Scope and Non-Goals

**In Scope:**
- Candidate intent generation: LLM produces multiple scored intents from context, not just one
- Intent ranking and threshold-based routing: score intents, apply threshold, select winning agent
- Agent routing engine: capability matching, vector similarity, historical performance, reputation
- Reputation graph: agents accumulate reputation from telemetry; reputation influences routing
- Failure and bias detection: detect looping, failing, unstable, or biased agents
- Agent ranking and evaluation layer: consume Sploink telemetry → compute metrics → produce agent scores
- Schema alignment with Imran: telemetry schema must be compatible with Salaxia's ranking inputs

**Non-Goals:**
- Goal intent / orchestration layer (high-level goal decomposition) — future phase
- Multi-agent workflow orchestration — Phase 2 after single-agent routing works
- Agent marketplace — post-MVP
- User-facing UI — Ved's domain

**Tradeoffs Accepted:**
- Start with single-agent routing for MVP. Multi-agent pipelines come after the core routing loop is proven.
- Reputation graph starts simple (success rate + cost efficiency). Richer signals (user feedback, drift detection) come later.
- Scoring weights manually configured for MVP; ML-tuned weights in Phase 2.

---

## 3) System Design

### The Intent Pipeline

```
context ingestion
→ candidate intent generation (LLM produces multiple candidates with confidence scores)
→ intent ranking / scoring
→ threshold check (e.g. 0.7 → winning intent selected)
→ router (picks best agent by capability match + reputation)
→ agent execution (Sploink runtime)
→ telemetry + ranking update
```

**Why multi-candidate, not single classification:**
Context is ambiguous. "Are you free Tuesday?" could be `schedule_meeting` (0.82), `draft_email` (0.63), or `check_calendar` (0.12). Forcing one classification makes the system fragile and impossible to improve. Multi-candidate generation lets the system recover from mistakes and evaluate routing decisions over time.

**Intent format:** verb + object. Good: `schedule_meeting`, `draft_email`, `summarize_thread`. Bad: `relationship_management`, `work_task`.

### The Routing Decision

Routing uses:
1. Capability match: does the agent support the required capabilities?
2. Vector similarity: how closely does the agent's capability profile match the intent?
3. Historical performance: what's the agent's success rate on similar tasks?
4. Agent reputation: accumulated score from past telemetry

### The Ranking Loop

Salaxia consumes telemetry from Sploink and updates agent rankings after every execution:
- High success rate + low latency + low cost → reputation increases
- High retry rate + errors + instability → reputation decreases
- Rankings feed back into routing decisions → system improves over time

---

## 4) Workstreams

### Workstream 1: Routing and Intent Layer (Create Routing System)

Building on existing intent graph work (SAL-9 in progress). The missing pieces:
- Multi-candidate intent generation (not just decomposition)
- Scoring and threshold-based routing decision
- Agent routing engine with capability matching and reputation input
- Reputation graph
- Failure and bias detection

### Workstream 2: Agent Ranking and Evaluation Layer (new)

Separate from routing logic. This layer:
- Ingests telemetry events from Sploink
- Computes structured performance metrics
- Produces agent scores and rankings
- Feeds rankings back into the routing engine

---

## 5) Success Criteria

**MVP succeeds when:**
- [ ] Given a user context, system generates multiple candidate intents with confidence scores
- [ ] A threshold is applied and the highest-confidence intent routes to the correct agent
- [ ] Agent is selected based on capability match and reputation score
- [ ] After execution, telemetry from Sploink updates the agent's ranking
- [ ] Repeated executions improve routing accuracy (failing agents ranked lower over time)

**Guardrail metrics:**
- Intent generation must produce at least 2 candidates for any ambiguous input
- Routing decisions must be explainable (log: intent chosen, score, agent selected, reason)
- No agent with >20% error rate should rank in the top 3 for any task type

---

## 6) Risks

| Risk | Detection | Mitigation |
|---|---|---|
| Single intent classification | Check output: always >= 2 candidates | Enforce multi-candidate in prompt; never accept single-output classification |
| Telemetry schema mismatch with Sploink | Test ingestion with Imran's events on Day 2 | Align schemas before any ranking code is written — coordinate with Imran |
| Reputation cold start (no telemetry yet) | First routing decisions have no history | Default to capability match only; reputation weight increases as data accumulates |
| Looping agents degrade system | Monitor retry count and error rate per agent | Automatic demotion: any agent exceeding retry threshold gets flagged and deprioritized |

---

## 7) Open Questions

- [ ] What is the initial intent taxonomy? (How many intents, what are they?) — Aryan + Timothy
- [ ] How does `task_id` flow from Seoro through Salaxia into Sploink telemetry? — Timothy + Aryan + Imran
- [ ] What's the minimum reputation signal needed before reputation influences routing? (minimum N executions) — Aryan
- [ ] How does multi-agent orchestration get queued for Phase 2 without breaking MVP routing architecture? — Timothy + Aryan

---

## 8) Next Steps

| Milestone | Exit Criteria |
|---|---|
| Intent pipeline working | Multi-candidate intents generated with scores; threshold routes to agent |
| Routing engine live | Agent selected by capability match + reputation; decision logged |
| Telemetry ingestion live | Sploink events consumed and stored in Salaxia's ranking layer |
| Rankings updating | Agent scores update after each execution; routing uses updated scores |
| End-to-end test | Full loop: context → intent → route → execute → telemetry → ranking update |
