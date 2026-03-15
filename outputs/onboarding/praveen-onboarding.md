# S³ — What We're Building and Where You Fit

Hey Praveen,

Here's the full picture.

---

## The vision

Software is being written by AI agents now. Not assisted by them — written by them. Claude Code, Cursor, Devin, and whatever comes next will be the primary executors of engineering work within a few years. That's already true for some teams today.

But nobody can watch what an agent is actually doing. You start a session, the agent runs for 20 minutes, you come back and either it worked or it didn't. If it failed, you have no idea when it went wrong or why. If it succeeded, you have no idea how to replicate that. Agents are black boxes.

That's the problem. S³ is building the infrastructure layer that makes agents observable, measurable, and orchestratable. Think of it as the operating system for agentic work — not an AI product, but the layer underneath AI products that makes them trustworthy enough to actually run at scale.

---

## The phases

**Phase 1 — Measurement (building now)**

We wrap the agent CLI. Every shell command, file edit, git check, and test run gets captured as a telemetry event. From those events, we compute three signals in real time:

- *Convergence* — is the agent actually moving toward done, or just busy?
- *Thrash* — is it stuck in a loop, editing the same file over and over, running the same failing command?
- *Momentum* — is the session accelerating or decelerating over the last few minutes?

The output is a live session score. For the first time, you can watch an agent work and know whether it's going somewhere.

**Phase 1.2 — Data fusion**

The CLI tells you what the agent *did*. An IDE extension tells you what the code *actually is* — error counts, diagnostics, lint state. We run both in parallel and measure the gap between them.

"Agent said it fixed the bug. Error count went up." Nobody captures that divergence today. That gap metric is data nobody else has, and it compounds into execution fingerprints unique to each agent over time.

**Phase 2 — Intervention**

Once you can see what's happening, you can act on it. A real-time cockpit — convergence dropping below threshold, one tap to pause the agent, redirect, or swap it out. Not a dashboard you check after. A cockpit you watch during.

The mobile vision is this: you're in a meeting, three agents are running tasks in parallel, your phone tells you one of them is thrashing. You intervene from your phone. You don't need to be at a desk to manage a fleet.

**Phase 3 — Routing and reputation**

Salaxia is our routing layer. It takes a task, decomposes it into ranked intent candidates, and routes to the right agent. As execution data accumulates across sessions, agents develop measurable reputations. An agent that converges reliably on a class of task moves up the ranking. One that thrashes moves down. The marketplace surfaces this — you pick agents the way you pick contractors, based on a track record you can actually see.

**Phase 4 — OS layer**

This is the long game. eBPF probes that capture kernel-level signals — memory trajectory, syscall entropy, I/O loops — without modifying the OS. Eventually: a custom OS where agents are first-class primitives, not applications running on top of infrastructure that wasn't built for them. This is 2027-2028.

---

## Where you might fit

We're three people right now. Me and Ved are building the CLI layer and backend. We need someone who can build the surfaces people actually see — and who thinks at the systems level, not just the component level.

The dashboard and iOS app are the cockpit. That's your Swift background applied to something that doesn't exist yet. But honestly, what stood out in our conversation wasn't the Swift — it was that you mapped the routing/scoring problem to your fraud detection work before I framed it. The question of how you rank agents with cold-start data, how you distinguish real drift from intentional course correction, how you decide when to intervene — those are open problems and I want that thinking at the table.

We're pre-funding, pre-revenue. Pre-beta is Sunday. If you're in, it's because you want to be building this, not because of what's certain.

— Timothy
