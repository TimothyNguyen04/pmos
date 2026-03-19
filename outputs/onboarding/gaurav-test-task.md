# Test Task — Gaurav

Hey Gaurav,

Here's your task. It's research-grade, not a CRUD exercise. Take your time but move fast -- 48-72 hours.

---

## Background

We're building a 4-layer hierarchical knowledge graph of every codebase Sploink monitors:

```
Layer 1 — Intent        (task prompt, sub-goals — what was asked)
Layer 2 — Features      (README, PRD — what the system is supposed to do)
Layer 3 — Design        (architecture decisions, interface contracts, inline comments)
Layer 4 — Code graph    (files, functions, dependencies — already built)
```

Layer 4 is done. The problem is Layers 1-3. They live in natural language -- prompts, READMEs, PRDs, comments. Turning that unstructured text into a structured graph that connects meaningfully to Layer 4 is the hard part. That's what you're solving.

Every node in the graph has this shape:

```json
{
  "id": "unique_identifier",
  "layer": 1,
  "type": "goal | feature | design_decision | file | function",
  "label": "human readable name",
  "stale": false,
  "stale_reason": null,
  "last_modified": "timestamp",
  "connected_to": ["node_ids"]
}
```

Every edge:

```json
{
  "from": "node_id",
  "to": "node_id",
  "relationship": "implements | governs | depends_on | sub_goal_of",
  "layer_crossing": true
}
```

Layer-crossing edges are what make the graph useful. A `design_decision` node in Layer 3 that `governs` a `file` node in Layer 4 means: if that file changes, the design decision is now potentially stale. That's the propagation path.

---

## The Task

Given the input below (a real feature spec from our system), do three things:

### Part 1 — Entity extraction

Extract all meaningful entities from the text. For each entity, identify:
- What layer it belongs to (1, 2, or 3)
- What type it is (`goal`, `feature`, or `design_decision`)
- A clean label

Don't over-extract. A good graph has the right nodes, not all possible nodes. Show your reasoning for what you included and what you left out.

### Part 2 — Graph construction

Build the graph. Output it as JSON conforming to the schema above. Include:
- All extracted nodes with correct fields
- All edges between them, including layer-crossing edges where applicable
- Relationship types on every edge (`implements`, `governs`, `depends_on`, `sub_goal_of`)

### Part 3 — Temporal extension (bonus, but important)

You mentioned temporal knowledge graphs in our call. This graph is live -- it updates as an agent edits files in real time.

Answer this question in 200-300 words: how would you extend this graph to be temporal? Specifically -- when a node's `stale` field flips to `true`, how should the graph represent the history of that change? What data structure would you use, and why?

---

## Input: Feature spec

```
Feature: Session convergence scoring

Goal: Determine in real time whether a coding agent is converging toward
a correct solution or diverging.

The system wraps the agent CLI. Every shell command, file edit, git commit,
and test run is captured as a telemetry event. From these events, three
signals are computed continuously:

  - Convergence: is the agent moving toward done, or just busy?
  - Thrash: is it stuck in a loop, editing the same files repeatedly?
  - Momentum: is the session accelerating or decelerating?

Architecture decision: signals are computed as independent modules and
composed into a single session score. This allows each signal to be
calibrated independently without affecting the others.

Architecture decision: the scorer is stateless per event. It receives
an event stream and outputs a score. No session state is stored inside
the scorer itself -- state lives in the flight recorder.

The flight recorder (sploink/recorder.py) captures all events to disk.
The scorer (sploink/metrics.py) reads from the recorder. The convergence
engine (sploink/convergence.py) composes the three signals.

Success: given a live agent session, the system outputs a convergence
score between 0 and 1 that updates within 100ms of each new event.
```

---

## What we're looking for

- **Precision in extraction:** Did you get the right entities? Too many nodes is as bad as too few.
- **Edge quality:** Are the relationships correct? Are layer-crossing edges where they should be?
- **Schema compliance:** Does your JSON match the schema exactly?
- **Temporal thinking:** Does your Part 3 answer show you understand the tradeoffs, not just the mechanics?

Ship it as a GitHub repo or a Gist. Drop the link in WhatsApp when done.

— Timothy
