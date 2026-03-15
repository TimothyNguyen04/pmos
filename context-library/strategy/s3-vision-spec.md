# S³ — Vision Spec

*March 2026*

---

## The shift nobody has infrastructure for

Software is no longer written by humans alone. AI agents — Claude Code, Cursor, Devin, and everything coming after — are becoming the primary executors of engineering work. This is already true for early adopters and will be true for most teams within two years.

But the infrastructure underneath agents hasn't changed. Agents run on operating systems built for human processes. They're monitored with tools built for human developers. They're evaluated by humans who check the output after the fact. Nobody watches what an agent actually does during a session. Nobody measures whether it's converging or drifting. Nobody routes tasks to the agent most likely to succeed on that specific class of work.

This is the gap. S³ is building the infrastructure layer that fills it.

---

## The thesis

Agents are processes, not applications. The right abstraction isn't "an AI tool you use" — it's "a process you spawn, monitor, and route work to." Once you accept that, the infrastructure requirements become obvious: you need observability, you need scheduling, you need routing, you need a reputation system. None of these exist for agents today.

S³ is building the operating system for agentic work. Not an agent. The layer underneath agents that makes them observable, rankable, and orchestratable at scale.

---

## The stack

```
Sahalia        OS layer — agents as kernel primitives (2027-2028)
    ↓
Sanora         Perception layer — interface, context, BCI (future)
    ↓
Sploink        Measurement layer — telemetry, convergence, thrash detection (now)
    ↓
Salaxia        Routing layer — intent decomposition, agent selection, ranking (now)
    ↓
Marketplace    Reputation graph — agents ranked by execution track record (next)
```

Each layer is a distinct product with its own moat. Together they form a closed feedback loop: agents execute work, Sploink measures the execution, Salaxia uses the measurements to route better next time, the marketplace surfaces agents with the best track records, better agents attract more tasks, more tasks generate more measurement data.

---

## What Sploink does

Sploink wraps agent CLI sessions and captures every event: shell commands, file edits, git operations, test runs, build outputs. From those events, it computes three signals in real time:

**Convergence (0.0–1.0):** Is the agent moving toward done? Weighted composite of git progress, test pass rate, and file churn. A session trending toward 1.0 is healthy. Flat or declining means something is wrong.

**Thrash (boolean + confidence):** Is the agent stuck in a loop? Triggered by repeated edits to the same file without commits, the same command failing multiple times, or zero git delta over the last window of events. Thrash is the primary failure mode of coding agents today and nobody detects it in real time.

**Momentum (float + trend):** Is the session accelerating or decelerating? Compares the current event window against the prior window. Tells you not just where you are but where you're going.

Phase 1 is CLI-only. Phase 1.2 adds an IDE extension that captures diagnostic state in parallel — error counts, lint warnings, type errors. The two streams fuse into a gap metric: divergence between what the agent *did* and what the code *actually is*. "Agent said it fixed the bug. Error count went up." Nobody measures this today. That gap data compounds over time into execution fingerprints unique to each agent — behavioral signatures that can't be replicated or faked.

Phase 3 goes to eBPF probes: kernel-level signals without OS modification. Memory trajectory, syscall entropy, I/O loop detection. At this layer, thrash becomes visible at the binary level before it surfaces in git or file events. This is the moat that takes years to build and that no application-layer competitor can replicate.

---

## What Salaxia does

Salaxia is the routing brain. It takes a task description, decomposes it into multiple ranked intent candidates (never a single classification — ambiguity is preserved), and routes to the agent best suited for that class of work.

In Phase 1, routing is capability-match only. Cold start problem: no execution history means you default to declared capability. As Sploink data accumulates, routing becomes empirical. An agent that converges reliably on summarization tasks ranks higher for summarization. An agent that thrashes on TypeScript migrations moves down. The ranking is based on measured execution behavior, not marketing claims.

This is the reputation graph. It's the marketplace primitive that makes agent selection trustworthy at scale.

---

## The human-on-the-loop paradigm

The current paradigm for working with agents is: launch, wait, check output, re-launch if wrong. This works for small tasks. It breaks down for long-running sessions, parallel agent fleets, and complex multi-step work.

The new paradigm is human-on-the-loop: a cockpit that shows you what every agent is doing in real time. Convergence dropping below threshold surfaces as an alert. One tap to pause, redirect, or swap the agent out. You don't re-launch from scratch — you intervene mid-session with full context preserved.

This is what the Sploink dashboard and iOS app surface. Not a dashboard you check after. A cockpit you watch during. The mobile interface specifically unlocks a new behavior: managing a fleet of agents while you're moving through the world. Agents work continuously; humans check in on demand.

Phase 2 extends this with predictive intervention — forecasting convergence failure before it happens. The cockpit tells you what's about to go wrong, not just what's going wrong now.

---

## Why now, why us

Three conditions are true simultaneously for the first time:

1. Agents are capable enough that real engineering work gets delegated to them
2. Sessions are long enough and expensive enough that mid-session failure is costly
3. The tooling layer is still completely empty — nobody has built observability for agents

Every competitor in this space owns one layer. Datadog owns observability for microservices. LangChain owns agent orchestration primitives. OpenAI owns the model. Nobody owns the full stack from telemetry through routing through marketplace. S³ is the only product being built with all three layers connected from the start.

The proprietary execution trace data is the compounding advantage. Every session run through Sploink generates measurement data that trains better routing in Salaxia. The marketplace creates a flywheel: better routing attracts more agents, more agents generate more data, more data improves routing. This loop doesn't exist if you only build one layer.

---

## Where this goes

Near term: the standard telemetry layer for serious teams running AI coding agents. The way Datadog became the standard for microservice monitoring.

Medium term: the routing layer that enterprise teams use to manage fleets of specialized agents across codebases. Agents become interchangeable — the routing intelligence and execution history are what's permanent and valuable.

Long term: the operating system for agentic work. Sahalia — a custom OS where agents are first-class kernel primitives with native scheduling, capability-based security, and marketplace-native IPC. eBPF is the bridge. Nobody is building toward this yet because nobody has the telemetry foundation to justify it. We will.

The bet: whoever owns the measurement layer owns the trust layer. And whoever owns trust owns the marketplace. The infrastructure play always wins over the application play when the infrastructure doesn't exist yet.

---

## The paradigm nobody has built

Nearly every multi-agent system in production today is horizontal:

```
Supervisor → Coding Agent → Testing Agent → Review Agent
```

Sequential pipeline. Output of one feeds the next. This is CrewAI, AutoGen, LangChain — all of them. It compounds errors. If Agent 1 makes a wrong assumption, every downstream agent inherits it. It's also slow — each agent waits for the previous one to finish.

The missing paradigm is vertical:

```
Agent A ─┐
Agent B ─┼─→ trajectory mixer → best unified output
Agent C ─┘
```

Multiple agents attack the same step simultaneously. Different approaches, different contexts, different models. You measure all trajectories in real time, pick the winner, or merge the best paths forward. This is parallel search over the solution space — the same principle that makes AlphaGo work. Not 2x faster. A different class of intelligence.

Nobody has built this in production. The reason: you can't do vertical agents without the trajectory measurement layer. You need real-time signal on which agent is converging and which is drifting. Without that you have no basis for selecting or merging trajectories. You're flying blind.

Sploink is building that measurement layer. It's the prerequisite the entire industry is missing — which is why the entire industry is stuck building horizontal pipelines.

The measurement layer unlocks vertical agents. Vertical agents unlock a new paradigm of how software gets written. That's what S³ is actually building toward.

---

*S³ Inc — Delaware C-Corp (in progress)*
*Team: Timothy Nguyen, Ved Sharma*
