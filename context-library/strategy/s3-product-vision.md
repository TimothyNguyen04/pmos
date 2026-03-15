# S³ Product Vision and Architecture Philosophy
*Last updated: 2026-03-11*

---

## Unified Product Experience (Most Important)

The user should never feel:
- "now I'm in Seoro"
- "now I'm in Salaxia"
- "now I'm in Sploink"

They should only feel: **I gave the system context, it understood what I needed, it chose or built the right agents, and it got the work done.**

From the outside, it is one product. Seoro, Salaxia, and Sploink are three connected backend layers, not three separate products.

**The clean one-line framing:**
- Seoro is the interface.
- Salaxia is the brain that routes work.
- Sploink is the engine that creates and improves the workers.

---

## End-to-End User Flow

A user opens the product through the Seoro interface (dashboard, chat, workspace, browser extension, or assistant view). The system gathers context: calendar, email, CRM, messages, documents, notes, tasks.

User says: "Draft a follow-up to yesterday's investor call and remind me if they don't reply."

**What the backend does invisibly:**

```
Seoro: captures intent and context
  → Who is "they"? What happened on the call? What tone is right? What data exists?

Salaxia: routes and orchestrates execution
  → Breaks into parts: summarize call / retrieve CRM context / draft email / schedule reminder
  → Selects best agents for each part based on ranking data

Sploink: powers agent intelligence in the background
  → If agents exist: provides telemetry, performance history, ranking signals to Salaxia
  → If agents don't exist or underperform: generates or adapts a better agent
  → Continuously measures agent behavior, detects thrash, flags architectural problems
```

**User sees:** polished emails and reminders. One ask. Done.

---

## The Platform Loop (Multi-User)

When multiple users are on the system, it becomes a shared intelligence layer:

- Founder asks for investor update → Seoro captures context → Salaxia routes → Sploink measures
- Sales lead asks for similar follow-up workflow → Salaxia routes using improved agents from the first user's session

**One user generates context and task patterns. The backend learns from execution and telemetry. Another user benefits from better routing, better agents, and better defaults.**

This is how it becomes a platform. No private data is exposed directly — only the performance signals from agent executions inform the shared ranking layer.

---

## The Business-to-Agent Compiler

The most important architectural insight: the system is not just an assistant. It is a **business-to-agent compiler**.

Founders reveal high-level pain through behavior:
- They constantly lose track of investor follow-ups
- Customer conversations create repeated ad hoc work
- Engineering time is wasted on repetitive internal tasks

The system detects these patterns and translates them into what needs to be built:
- A CRM enrichment agent
- An investor follow-up drafting agent
- A meeting summarization and tagging agent
- A code generation workflow for internal dashboards

```
Founder behavior and context
↓
Latent workflow detection
↓
Agent opportunities identified
↓
Technical implementation / coding
↓
Telemetry and improvement
```

**Without this bridge:** you have either a nice assistant that understands the founder but doesn't build real machinery, or a low-level agent builder that can create things but doesn't know what matters for the business.

The bridge between high-level context and what's being coded is the actual moat.

---

## How Templates Emerge (Not from Marketplaces)

Templates do not come from agent marketplaces. They come from what employees choose to share.

**The organic template flywheel:**
1. Each individual has their own context
2. Context is shared within their business → builds templated agents
3. Businesses with these agent templates band together
4. An industry of agents forms, working together

This means:
- Templates are validated by real use, not curated by a platform
- Trust is peer-to-peer, not top-down from a marketplace
- The supply of agents grows organically as employees share workflows

---

## Long-Term Architecture (7 Layers)

```
Layer A — Perception (Seoro)
Raw context: Gmail, Slack, Calendar, meeting transcripts, ambient screen/UI signals, docs/OCR, user actions

Layer B — Routing Intent (Salaxia)
Small, deterministic, verb+object task labels for routing
Examples: draft_email, schedule_meeting, summarize_thread, update_crm, prepare_brief

Layer C — Goal Intent (Timothy's layer — future)
Higher-level inferred outcomes that decompose into routing intents
Example: plan_meeting → schedule_meeting + draft_email
Goal intent decomposes into routing intents, NOT directly to agents

Layer D — Supply (Sploink + community)
Agents, templates, workflows, sub-agent bundles, eventually marketplace components

Layer E — Execution + Telemetry (Sploink)
Run, measure, rank, improve

Layer F — Artifacts + Memory (Seoro)
Persist useful outputs: meeting briefs, summaries, outreach drafts, action plans, CRM updates

Layer G — Human-in-the-Loop (across all layers)
For low-confidence cases: show 2-3 ranked action paths, let user approve/reject/modify
Generates correction telemetry → improves future routing
```

**MVP implements: A, B, D, E only.**
Goal Intent (C) comes later. Architecture must leave room for it.

---

## The Correct Intent Pipeline

```
context ingestion
→ candidate intent generation (LLM produces multiple scored candidates — NOT just one)
→ intent ranking / scoring (confidence scores per candidate)
→ threshold + router (e.g. threshold 0.7 → winning intent selected → best agent picked)
→ agent execution
→ telemetry + artifacts
```

Why candidate intents matter: context is ambiguous. "Are you free Tuesday?" could be schedule_meeting (0.82), draft_email (0.63), or summarize_thread (0.12). Forcing a single classification makes the system fragile and hard to improve.

Telemetry must log: context, predicted intent, chosen agent, result, user correction. This is how routing improves over time.

---

## What This System Ultimately Is

Agents should not be black boxes. They should be measurable, comparable, and continuously improving infrastructure.

The system compounds because:
- More usage → more telemetry
- More telemetry → better routing and ranking
- Better routing → better agent performance
- Better agent performance → more usage

Each user makes the system better for every other user. That's the platform moat.
