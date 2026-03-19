# Hierarchical Knowledge Graph — Spec for Param

Hey Param,

You already understand the scoring lane (SAL-31 — measuring agent behavior from a stream of events). This is the next layer underneath it. Instead of scoring from events alone, we're building a graph of the codebase that lets us score from *structure* — and use that structure to flag bugs before they actually crash.

This is the problem nobody is solving. We've checked.

---

## The core idea

Right now, when an agent edits a file, we know the edit happened. We don't know what it broke. The graph changes that.

We're building a 4-layer hierarchical knowledge graph of every codebase Sploink touches:

```
Layer 1 — Intent       (what was asked, sub-goals, the original task prompt)
Layer 2 — Features     (README, PRD — what the system is supposed to do)
Layer 3 — Design       (architecture decisions, interface contracts, comments)
Layer 4 — Code graph   (files as nodes, function calls/imports as edges) ← already built
```

When a file is edited in Layer 4, staleness propagates upward. A change to `auth.py` stales `token_validator.py` (Layer 4 sibling), which stales the JWT interface contract (Layer 3), which stales the login feature spec (Layer 2). The graph tells you what's now inconsistent — not after the crash, but as the agent is still writing.

The problem you're solving: **use this graph to predict which files are about to cause a crash, before the crash happens.**

---

## What's already built

**SPO-84 — graph schema** (Done ✅)

The 4-layer node and edge schema is locked. Every node has: `id`, `layer`, `type`, `staleness_state`, `last_modified`, `confidence`. Every edge has a `type` (structural, semantic, cross-layer) and `weight`.

**SPO-60 — Layer 4 base graph** (Done ✅)

Files parsed into nodes. Import dependencies as edges. Co-edit history from session data. The skeleton of Layer 4 exists.

---

## What you're building

Three issues chain together into the full problem. Start in order.

---

### SPO-85 — Extend Layer 4: CPG + full dependency graph

Layer 4 currently has files and imports. It needs:

- **Code Property Graph (CPG)** — functions as nodes, function calls as edges. This is the granularity that matters for bug detection. A crash in `process_token()` doesn't implicate `auth.py` globally — it implicates the specific call chain leading to that function.
- **Module dependency graph** — which modules import which (beyond individual files)
- **Route dependency graph** — API routes and their handler chains
- **Co-edit history enrichment** — files edited together in the same session are implicitly coupled even if the static graph doesn't show it

All four must conform to the SPO-84 schema.

**Done when:** Given any Python repo, the system produces a fully populated Layer 4 graph in under 10 seconds.

---

### SPO-86 — Staleness propagation engine

The core primitive. When `file_edit` fires from Sploink:

```
1. Locate the edited file as a node in Layer 4
2. Mark it MODIFIED (intentional — not stale)
3. Traverse all outbound edges from that node
4. For each connected node: mark STALE, record reason
5. For each newly stale node: traverse its outbound edges
6. Continue until no new stale nodes
7. For any stale Layer 4 node: traverse layer-crossing edges upward
8. Mark connected Layer 3, 2, 1 nodes as STALE
9. Return: full stale node map with propagation path
```

Target output:

```json
{
  "modified": ["auth.py"],
  "stale": {
    "layer4": ["token_validator.py", "session_store.py", "middleware.py"],
    "layer3": ["JWT interface contract"],
    "layer2": ["login feature", "token refresh feature"],
    "layer1": []
  },
  "propagation_depth": 3,
  "cascade_root": "auth.py → token_validator interface"
}
```

**Performance target:** Full propagation on a 10k file repo in under 500ms.

**Done when:** This output is produced correctly on a real codebase within the time target.

**Blocked by:** SPO-85 (Layer 4 must be fully populated first)

---

### SPO-98 — Pre-crash bug risk scoring

This is the payoff. Once the graph is live and staleness is propagating, you have everything you need to score each file's bug risk *before* the error fires.

Input signals:

| Signal | Source | What it tells you |
|--------|--------|-------------------|
| Stale node count + depth | SPO-86 | Files downstream of a recent edit are higher risk |
| CPG in-degree | SPO-85 | Files with many callers amplify risk when modified |
| Co-edit divergence | SPO-85 | Files that always get edited together but now diverge |
| Layer boundary crossings | SPO-86 | Changes propagating into Layer 3 = architectural drift |
| Edit velocity vs. test activity | Live Sploink events | Files edited fast with no test coverage activity |

Output per file, emitted after every `file_edit` event:

```python
{
  "file": "auth/token_handler.py",
  "risk_score": 0.87,              # 0–1
  "risk_factors": [
    "stale_dependents: 4",
    "cpg_in_degree: 12",
    "no_test_activity: 18min"
  ],
  "recommended_action": "Review before next agent step"
}
```

This feeds directly into the VS Code gutter highlighting (SPO-96) — files above threshold get colored indicators in the IDE while the agent is still running.

**Success criteria:**
- Score produced within 200ms of a `file_edit` event
- In manual testing: top-3 highest-risk files include the file that eventually caused the crash in ≥60% of sessions
- False positive rate: files with risk score < 0.3 do not appear highlighted
- Queryable via `sploink.graph.risk_scan()` — returns ranked list with factors

**Blocked by:** SPO-85 + SPO-86

---

## How this connects to your scoring work

SAL-31 scores agent behavior from events: error rate trajectory, test regression depth, retry escalation. All time-series signals.

SPO-98 scores bug risk from graph structure: stale nodes, CPG topology, co-edit coupling. Structural signals.

These are orthogonal. One tells you the agent is drifting. The other tells you *where* in the codebase the drift will materialize. Together they close the loop — you know the agent is in trouble, and you know which files to look at.

The composition layer (SAL-32) eventually fuses both into one session score. That's Phase 2. For now, focus on getting the structural signal working.

---

## Where to start

1. Read SPO-84 (schema) to understand the node/edge contracts before writing any code
2. Implement SPO-85 — build out the full Layer 4 graph on a real test repo
3. Implement SPO-86 — staleness propagation on top of that graph
4. Test both together: edit a file, verify the stale map propagates correctly up through the layers
5. Then SPO-98 — the scoring layer on top

The test repo you should use is the Sploink codebase itself. It's Python, it's active, and you already know roughly what it does.

---

## Context for the RL framing

The staleness propagation is essentially a reward signal for a graph state. A high-stale, high-depth cascade after an edit is the graph equivalent of a reward cliff — the agent just made a change with consequences it can't see. The risk scorer is the policy that reads that state and outputs an intervention signal.

Your background in world models maps directly here. The graph is the world model. Staleness is the observed state. Bug risk is the predicted consequence. The question you're answering: given this graph state, what's the probability that the next N steps result in a crash?

That's the research problem. SPO-98 is the implementation of it.

---

## Linear issues

- **SPO-84** — Graph schema (Done ✅, read this first)
- **SPO-85** — Extend Layer 4: `linear.app/s-3/issue/SPO-85`
- **SPO-86** — Staleness propagation: `linear.app/s-3/issue/SPO-86`
- **SPO-98** — Pre-crash bug risk scoring: `linear.app/s-3/issue/SPO-98`

Start with SPO-85. Questions go to me directly.

— Timothy
