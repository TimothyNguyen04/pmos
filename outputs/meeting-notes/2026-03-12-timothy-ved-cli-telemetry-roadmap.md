# Meeting Notes: CLI Wrapper + Agent Telemetry Roadmap
**Date:** March 12, 2026
**Attendees:** Timothy Nguyen, Ved Sharma
**Duration:** ~30 min

---

## Decisions Made

- **CLI first, IDE second, OS third.** Confirmed sequencing. CLI is the entry point.
- **MVP scope locked:** CLI wrapper + dashboard. 4-day timeline starting Monday.
- **Dashboard is web-based.** Needs a frontend/UI person — Timothy to recruit.
- **Dashboard should be non-intrusive** — side panel style (like Granola), no tab switching.
- **Not building (yet):** agent marketplace, IDE extension, multi-agent orchestration, billing, user accounts.
- **B2B pivot is happening** regardless of Ben's response. Vision has diverged.

---

## Key Discussion Points

### Why CLI over IDE
The IDE couldn't read prompts, so it couldn't attribute agent actions to specific sub-goals. The CLI can. That distinction is load-bearing — it's what makes goal attribution possible.

### Product framing confirmed
Low-level is the wedge. LangSmith/Langfuse/AgentOps own high-level (token usage, call tracing). The gap is file-level behavioral telemetry: drift, convergence, divergence. Nobody is measuring this.

AgentOps flagged as the one competitor to watch — they *could* add low-level telemetry. The defensible moat is the specific formula for convergence and thrash, plus the data that accumulates.

### Dashboard visualizations discussed
1. **Codebase heat map** — graph of the codebase showing which files the agent touched and where its "context" is concentrated
2. **Token usage per component** — bar chart showing how many tokens were spent on which part of the task
3. **Time saved metric** — user-facing value prop
4. **Convergence/thrash/momentum curve** — the core behavioral signal

---

## New Ideas (Game-Changing)

### 1. Goal Attribution via CLI Prompt Reading
Because the CLI can read the agent's prompts, you can attribute actions to specific sub-goals. Example: "Create a LinkedIn clone" → sub-goals: create frontend, create backend, wire them together. The CLI can map which file edits, commands, and events belong to which sub-goal.

This means you can rank not just agents, but the **individual skills and prompts that comprise an agent**. Massive signal expansion.

### 2. Skills Marketplace (not Agent Marketplace)
The cold start problem for an agent marketplace is real — you need agents before you have data. But skills are more atomic and easier to bootstrap.

Idea: instead of routing demand (user intent) to supply (full agents), route demand (abstract sub-tasks) to supply (**skills** — components of agents). A generalized agent picks and combines skills at runtime. The Salaxia router applies to skills, not just agents.

This solves cold start because you don't need complete agents. You need skills. And skills can be extracted from existing agent runs via the telemetry data.

**Implication:** Salaxia scores skills, not just agents. Same infrastructure, bigger surface.

### 3. Dynamic Parameters for Thrash/Convergence Formulas
Thrash and convergence are not static thresholds. What counts as "editing the same file too many times" depends on the project, the person, and the context. A refactor task looks different from a greenfield build.

The convergence/thrash formula needs parameters that adapt over time and per context. This is SPL-18. The insight: the formula itself is a model, not a rule. It needs to learn from the data it sees.

### 4. B2B → B2B → B2C Evolution Path
Long-term GTM arc:
1. **B2B (now):** CLI for power users (developers running agents). Build telemetry data.
2. **B2B (next):** Agent/skills marketplace. Businesses find and use proven agents.
3. **B2C (future):** Use marketplace agents to help anyone build agents. The data from usage trains the next cycle.

The "n-to-n" marketplace dynamic kicks in at step 2. Step 3 is the consumer surface.

### 5. Ved's Workflow Automation Idea (Phase 3)
Ved proposed: deploy as a workflow automation tool where users add a prompt/task, system captures events (meetings, emails, etc.), Salaxia router maps the required workflow, ranks and connects skills together. Not now — depends on time and resources — but worth keeping in the roadmap.

---

## Action Items

| Owner | Action | Timeline |
|---|---|---|
| Timothy | Find frontend/UI person for dashboard | This week |
| Timothy | Onboard Aryan on new B2B direction and skills routing idea | This week |
| Timothy | Create specs for CLI wrapper + dashboard for team | After this meeting |
| Timothy | Continue thinking on B2B → B2C arc | Ongoing |
| Ved | Look into "the option" (eBPF or skills router — to confirm) and get back | TBD |
| Ved | Review docs Timothy sends and add comments | This week |

---

## Open Questions

- What exactly is "the option" Ved is looking into? (eBPF probes? The skills routing idea?) Confirm async.
- Dashboard: what's the tech stack? Who owns it? Frontend hire or existing team?
- How does goal attribution work in practice — does the CLI parse the initial prompt, or does the agent emit sub-goal markers?
- Skills taxonomy: how do we define and categorize skills so the router can match them?

---

## Context Notes

- Ben situation: Timothy confirmed moving forward with B2B pivot regardless of Ben's response. Vision has diverged and Ben has been difficult on equity. Pivot proceeds.
- Timothy accidentally called Ved "Ben" at the end of the call — no issue, just a note.
- 4-day MVP timeline starts Monday March 16.
