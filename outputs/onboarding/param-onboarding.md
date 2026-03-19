# S³ — What We're Building and Where You Fit

Hey Param,

Here's the full picture.

---

## The vision

Software is being written by AI agents now. Not assisted by them — written by them. Claude Code, Cursor, Devin, and whatever comes next will be the primary executors of engineering work within a few years. That's already true for some teams today.

But nobody can watch what an agent is actually doing. You start a session, the agent runs for 20 minutes, you come back and either it worked or it didn't. If it failed, you have no idea when it went wrong or why. If it succeeded, you have no idea how to replicate that. Agents are black boxes.

That's the problem. S³ is building the infrastructure layer that makes agents observable, measurable, and improvable over time. Not an AI product — the layer underneath AI products that makes them trustworthy enough to run at scale.

The Java 9 → 17 migration problem you described in our conversation is exactly this. An agent with access to both codebases, a knowledge graph of dependencies, and a clear convergence signal would solve that problem in hours. The missing piece isn't the agent. It's the measurement layer that tells you whether the agent is actually converging on a correct migration or silently drifting.

That missing piece is what we're building.

---

## The phases

**Phase 1 — Measurement (building now)**

We wrap the agent CLI. Every shell command, file edit, git commit, and test run gets captured as a telemetry event. From those events, we compute three signals in real time:

- *Convergence* — is the agent actually moving toward done, or just busy?
- *Thrash* — is it stuck in a loop, editing the same file over and over, running the same failing test?
- *Momentum* — is the session accelerating or decelerating?

The output is a live session score. For the first time, you can watch an agent work and know whether it's going somewhere.

**Phase 1.5 — Knowledge graph**

This is where it gets interesting for someone with your background.

We're building a 4-layer hierarchical knowledge graph of the codebase:
- Layer 1 — Intent (what was asked, sub-goals)
- Layer 2 — Features (README, PRD — what the system is supposed to do)
- Layer 3 — Design constraints (comments, architecture decisions)
- Layer 4 — Code graph (files as nodes, dependencies as edges — already built)

When a file is edited, staleness propagates upward through the layers. This tells you not just *that* the agent is thrashing, but *why* — which layer of the stack the drift originated from. A change in Layer 4 that conflicts with a constraint in Layer 3 is a different kind of thrash than a change that contradicts the original intent in Layer 1.

This is the self-adapting code problem you described — formalized into a measurable, queryable structure.

**Phase 2 — Intervention**

Once you can see what's happening, you can act on it. A real-time cockpit — convergence dropping below threshold, live file highlighting in the IDE showing where bugs are forming before they're visible in output. Not a dashboard you check after. A system that tells you where to look while the agent is still running.

**Phase 3 — Routing and reputation**

Salaxia is our routing layer. It takes a task, decomposes it into ranked intent candidates, and routes to the right agent. As execution data accumulates, agents develop measurable reputations. The marketplace surfaces this — you pick agents based on a track record you can actually see.

---

## The research problem underneath this

Here's what most people miss about what we're building.

The scoring layer is not a software engineering problem. It's a behavioral science problem with an engineering interface.

We're trying to answer: given a stream of low-level behavioral events (shell commands, file edits, test results, error types), can you accurately predict whether an agent is converging or diverging from a correct solution — before the outcome is visible?

That's an empirical question. The weights for the scoring formula can't be derived from first principles. They have to be discovered from real session data. Which means:

- What's the right weighting between error rate trajectory, test regression depth, novel error rate, and retry escalation?
- How do you distinguish intentional course correction from genuine drift?
- How do you handle cold-start (no prior session data for a new agent)?
- How do you score sub-goal completion rather than just session-level outcomes?

Your reinforcement learning background is directly applicable here. The convergence signal is essentially a reward function. The intervention layer is a policy. The session history is the environment. You've trained policies inside hydrogen atom environments — this is the same class of problem applied to agent behavior instead of quantum chemistry.

The difference is that the environment here is a real codebase, and the reward signal has to be computed from behavioral proxies rather than ground truth. That's the research contribution.

---

## Where you fit

The scoring lane is currently unowned. It's the most important open problem on the team.

Five issues compose into the full behavioral scoring system:

- **SAL-31** — Drift from error signal. Weighted scorer: error rate trajectory (0.35) + test regression depth (0.35) + novel error rate (0.20) + retry escalation (0.10). Outputs a `drift_score` between 0 and 1 with timestamps for each threshold crossing.
- **SAL-28** — Environment quality scorer. Separates "bad agent in a good environment" from "good agent in a bad environment." Scores scaffolding quality independently.
- **SAL-29** — Behavioral signal → skill matcher. Maps observed behavioral signatures to the right intervention skill.
- **SAL-26** — Intervention endpoint. Receives a behavioral signal, returns an intervention prompt.
- **SAL-32** — Unified session score. Composites all three signal streams into one number that feeds agent ranking.

This is yours if you want it. It's research-grade work with a real production system consuming the output. Not a paper that gets cited — a system that developers use to understand whether their agents are working.

---

## Honest context

We're pre-funding, pre-revenue. Three days from beta. Me and Ved have built the entire capture layer, scoring engine, and web dashboard. The scoring lane is the gap between Sploink being a dashboard and Sploink being a system that actually catches bugs before they happen.

It's equity only right now. We're applying to Y Combinator. If you're in it's because you want to be building this — not because of what's certain.

The test task if you want to move forward: implement SAL-31. The full spec is above. 48 hours. Ship the code.

That's the filter.

— Timothy
