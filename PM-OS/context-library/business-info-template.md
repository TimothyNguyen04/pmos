# Business Information

## Company Overview

### Basic Information

**Company Name:** S^3 (4-company agent lifecycle stack)

**Industry:** AI Infrastructure / Agentic Systems

**Stage:** Early-stage / Pre-product

**Founded:** 2024/2025

**Size:**
- Employees: [Founder + small team]
- Revenue: Pre-revenue
- Funding: [TBD]

**Website:** [TBD]

---

## The Four-Company Architecture

This is not four unrelated startups. It is four layers of one stack, each owning a different function in the agent lifecycle.

**The master loop:**
sense the world → create/compose agents → let agents act → measure what they did → rank/select better behavior → let stronger agents get reused → repeat

| Company | Role | One-line |
|---------|------|----------|
| **Sahalia** | Sensing | Perceives and structures external reality for agents |
| **Seoro** | Interface | Where users interact with and supervise agents |
| **Sploink** | Measurement | Turns opaque agent behavior into measurable, evolvable signals |
| **Salaxia** | Selection | Routes, ranks, and allocates agent work across the ecosystem |

---

## Company 1: Seoro

**What it is:** The user-facing agent operating environment. Where a founder or team actually lives with agents.

**Core stance:** Operator, not assistant. Assistant products wait for prompts. Operator products notice what should happen and act.

**Target user:** Founders and small teams (initial wedge). Expands to startups broadly, then teams inside larger companies.

**Core experience:**
- Chat/home — main control interface
- Agents — roster of active operators
- Artifacts — persistent output and structured knowledge
- Meetings — high-frequency workflow wedge
- Memory — persistent company context

**What Seoro ingests:** Email, Slack, calendar, docs, meetings, tasks, CRM, project systems.

**Key insight:** Agents are visible and act in the background. Users can see what agents exist, what they're doing, what they've done, what they learned, how they're improving, where intervention is needed. That visibility is what makes autonomy trustworthy.

**The wedge:** Meeting lifecycle agents. Not just transcription — the full lifecycle: schedule, prepare, contextualize, transcribe, extract decisions, create tasks, send follow-ups, store memory.

**Why meetings:** Frequent, painful, cross-functional, generative of structured company knowledge.

**Long-term:** Founder OS / Company OS — the place where human intent becomes coordinated agent activity.

**Monetization:**
- Near-term: Subscription + usage-based
- Long-term: Subscription + agent work capacity (pay for digital labor, not tokens)
- Later: Team/enterprise agent OS pricing + marketplace economics

---

## Company 2: Sploink

**What it is:** Measurement and construction substrate for agent behavior. Turns opaque agent work into measurable, evolvable behavior.

**Two levels of telemetry:**

High-level runtime:
- Latency, success/failure, tool calls, tokens, retries, workflow execution

Low-level behavioral (the differentiator):
- File churn, diff patterns, retry loops, revert/apply behavior, verification results, convergence vs. thrash, momentum, degradation, context drift

**Why it matters:** Many systems can tell you an agent ran. Very few can tell you whether it was converging intelligently or stumbling chaotically.

**Beyond telemetry:** Sploink becomes the workbench for agent building, decomposition, evolution, template generation, and operator/developer environment.

**Primary users:**
- Initially: internal team and technical operators
- Then: developers and companies building agentic systems
- Eventually: enterprises needing agent observability, debugging, construction tooling

**Why this is the deepest moat:** Behavioral telemetry compounds. More agent work → more behavioral data → better ability to distinguish effective from ineffective agents → more powerful routing and templating. Hard to replicate later.

**Monetization:**
- Developer platform
- Usage-based observability
- Enterprise agent infrastructure
- Evals/ranking services
- Agent-building environment subscriptions
- (More important: improves the entire rest of the stack)

---

## Company 3: Salaxia

**What it is:** Routing, ranking, selection, and marketplace intelligence layer. Decides which agents should do what.

**Core function:** Takes telemetry, metadata, templates, capabilities, context, and demand signals to answer:
- Which agent should run?
- Which workflow should be used?
- What should be routed where?
- What should be promoted vs. deprioritized?
- What should be recomposed into a better system?

**The key distinction:** Without Salaxia, you have an agent zoo. With Salaxia, you get an ecosystem.

**Router insight:** Routing informed by telemetry and reinforced by outcomes gives you task allocation, agent specialization, evolutionary pressure, and controlled expansion across many agents.

**Marketplace:** Not a static app store. A living market of agent templates, capabilities, sub-agents, workflow bundles — continuously re-ranked by telemetry and usage.

**Long-term:** The exchange, the scheduler, the ranking engine, the market-maker, the allocator of digital labor.

**Monetization:**
- Take rates on marketplace activity
- Ranking-as-a-service
- Routing premium plans
- Enterprise orchestration
- Large share of agent deployment economics

---

## Company 4: Sahalia

**What it is:** Sensing and semantic structuring layer for the external world. The perception system for the agent ecosystem.

**What it does:** Observes the internet (and eventually other systems), converts activity into structured meaning — ingesting signals, extracting entities and relationships, detecting changes, letting agent actions leave semantic traces.

**Key insight:** The semantic internet is not a switch. It grows gradually because agents interacting with the world leave marks. Those marks become checkpoints future agents can consult.

**The semantic checkpoint concept:**
- Agents act
- Actions leave semantic traces
- Traces become checkpoints
- Future agents hit checkpoints and consult the flywheel: what worked here before? what failed? what context exists?

**Why it sits outside the other three:** Seoro/Sploink/Salaxia form an internal closed-loop (create, act, measure, select). Sahalia is upstream and external (sense, structure, contextualize). It feeds the loop with structured world state.

**What it ultimately becomes:** A living semantic action map of the digital world. Not static knowledge representation — operational knowledge: "this kind of system, in this context, tends to produce these outcomes when agents take these actions."

**Monetization:**
- Semantic intelligence API
- Workflow/capability discovery
- Enterprise world-model layer
- Platform data infrastructure
- (Primarily an enabling layer early on — creates leverage more than immediate revenue)

---

## The Recursive Flywheel

1. Users/companies generate intent and activity in **Seoro**
2. Agents act in the world and inside company systems
3. **Sploink** captures how agents behaved and whether they converged or failed
4. **Salaxia** uses telemetry to rank and route better agent choices next time
5. **Sahalia** converts external changes and semantic traces into richer context for future decisions
6. Better agents + better context → better outcomes
7. Better outcomes → more usage → more telemetry and semantic traces
8. Whole system improves

---

## Strategy & Goals

### Current OKRs (Q1 2026)

[To be filled in]

### Product Strategy

**Strategic Themes:**
1. Wedge into Seoro via founder productivity (meeting lifecycle agents)
2. Build Sploink behavioral telemetry as compounding moat
3. Use telemetry to power Salaxia routing intelligence

**The entry point:** Small wedge (founder meeting agents) → generates context, artifacts, workflow patterns, telemetry, trust → feeds Sploink → powers Salaxia → Sahalia expands world awareness later.

**Not Doing (Explicitly):**
- ISP/packet-level surveillance approach for Sahalia
- Building one giant generalized black-box assistant
- Enterprise-first go-to-market

---

## Target Market

### Primary Customer (Seoro wedge)

**Who:** Founders and small startup teams (2-15 people)
**Company size:** Pre-seed to Series A
**Pain:** Chaos, repetitive overhead, cross-functional decisions, limited leverage
**Why they work:** Feel chaos acutely, tolerate imperfect products, want leverage immediately, make many cross-functional decisions

---

## Key Resources

**Tools:**
- Project management: [TBD]
- Design: [TBD]
- Documentation: [TBD]
- Communication: [TBD]
- Analytics: [TBD]

---

**Owner:** Timothy Nguyen
**Last Updated:** March 2026
