# S³ — What We're Building

*For the person who said "agents brute force debugging"*

---

You identified the problem better than most people I've talked to.

When an agent hits a bug, it tries option A, option B, option C. It doesn't know *where* the problem is. It doesn't know *how bad* it is. It just keeps trying. That's not intelligence — that's brute force with a language model on top.

The reason agents do this: they're blind to their own execution. No agent today knows whether it's converging on a solution or drifting further away. There's no measurement layer. Nobody built it.

That's what we're building.

---

## The core idea

We wrap agent sessions and capture everything in real time — file edits, shell commands, test runs, git operations. From those events we compute three signals continuously:

**Convergence** — is the agent moving toward done, or drifting?
**Thrash** — is it stuck in a loop, repeating the same failed approaches?
**Momentum** — is the session accelerating or slowing down?

With these signals, the debugging problem you described becomes solvable:

- Telemetry narrows down to the right *file* (highest edit churn, most error density)
- Agent analysis narrows down to the right *line*
- Specialist agent gets injected mid-session with exact context to fix that specific error

You don't restart the session. You don't lose context. The original agent keeps running — the specialist handles the blocker and hands back.

That's agent injection. It's one of three intervention primitives we're building.

---

## The stack

```
Sploink      Measurement layer — captures sessions, computes convergence/thrash/momentum
    ↓
Salaxia      Routing layer — routes tasks to the right agent, ranks by execution history
    ↓
Marketplace  Reputation graph — agents ranked by measured track record, not marketing claims
```

Each layer is a product with its own moat. Together they form a feedback loop: agents execute, Sploink measures, Salaxia routes better next time.

The long-term destination is vertical agents — multiple agents attacking the same problem simultaneously with split context, trajectories measured in real time, best paths merged into one output. Nobody has built this because nobody has the measurement layer. We're building that layer now.

---

## Where we are

Working telemetry pipeline in under two weeks. CLI wrapper capturing full event streams, scoring engine computing live convergence, IDE extension capturing diagnostics and editor state, telemetry emitting to Salaxia after each session. 26 tests on the emitter alone.

Two engineers. One founder. Pre-seed.

---

## Where you'd fit

You have the RL background and the multi-agent intuition. The problems we're working on next — trajectory mixing, intervention routing, agent scoring from execution data — are exactly the intersection of systems engineering and RL thinking.

The question you raised about how a master agent chooses between hundreds of agents at scale? That's Salaxia's core problem. We're rebuilding it in Python and designing the routing to be empirical — based on measured execution behavior, not declared capability.

If that's interesting, let's talk more specifically about what you'd own.

---

*S³ Inc — Timothy Nguyen (timothy@s3.ai)*
