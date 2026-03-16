# Sploink — Overview for Abhigyan Patwari

*March 2026 — Confidential*

---

## The problem nobody is watching

When you run Claude Code, Cursor, or any coding agent on a real task, something happens that nobody talks about: the agent loops. It edits the same file seven times. It runs the same failing test over and over. It drifts — quietly, invisibly — away from solving the problem and toward a pattern of behavior that looks like progress but isn't.

Right now, you have no way to see this happening. You launch the agent, you wait, you check the output. If it failed, you relaunch. There's no cockpit. No signal. No way to watch what the agent is actually doing mid-session and steer it toward a better outcome.

That's what Sploink measures.

---

## What we built

Sploink wraps agent CLI sessions and captures a real-time event stream — file edits, shell commands, git state, test results. From those events, it computes whether the agent is converging on a solution or thrashing in real time.

The dashboard shows a live convergence curve. When the curve flattens or drops, you know something is wrong before the session fails. When it climbs, you know the agent is making real progress. For the first time, you can watch an agent think.

We're launching pre-beta on Tuesday.

---

## Where you come in

The most powerful feature we haven't built yet is spatial intuition for what the agent is doing.

Imagine a code repo rendered as a graph — files as nodes, dependencies and co-edit relationships as edges. Now layer a heatmap on top of it that animates in real time as the agent works. Files light up as the agent touches them. Clusters of activity become visible. Loops become visible — you can literally see the agent ping-ponging between the same three files.

Then add replay. Scrub backward through a session. Watch the exact moment the agent started thrashing. Watch it recover. Watch the convergence curve sync to the spatial view in real time.

This is the feature that makes Sploink something you show someone and they immediately get it. No explanation needed. The repo graph with the heatmap replay is the hook.

We found GitNexus while building this out. It's the closest thing we've seen to what we're trying to create. That's why we reached out.

---

## The bigger picture

Sploink is one layer of a larger infrastructure stack we're building for agentic work — the observability and measurement layer that the entire industry is missing. The way Datadog became the standard for microservice monitoring, we're building the standard for agent session monitoring.

The spatial visualization isn't just a UI feature. It's the foundation for something more ambitious: representing how an agent moves through a codebase over time as a trajectory — a behavioral fingerprint that compounds into a model of how each agent thinks. That data is the long-term moat.

The frontend isn't a skin on top of a product. It's the product.

---

## What working together looks like

This is early-stage, pre-funding, pre-beta. The team is small and moving fast. We're not looking for a contractor to execute a spec — we're looking for someone who sees the visualization problem as deeply as we see the measurement problem and wants to own it.

Compensation is flexible — equity, paid, or a mix depending on what works. We'd want to start with a scoped project to see how we work together.

Happy to share more on a call — including a live demo of what we have now and a deeper walkthrough of where the repo graph feature fits into the roadmap.

---

*Timothy Nguyen — timothy@s3.ai*
