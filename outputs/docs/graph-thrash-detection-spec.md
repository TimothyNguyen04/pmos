# Hierarchical Knowledge Graph — Early Phase Spec
### Thrash Detection + Error Replay
*March 17, 2026 — Internal*

---

## What We're Building

A 4-layer hierarchical knowledge graph that sits alongside Sploink's event stream. Each layer produces raw signals. Those signals feed a thrash scorer per layer. The composite score tells us what type of thrash is happening and at what abstraction level — which determines the intervention.

The second deliverable is replay: when thrash is detected or errors compound, the graph lets us reconstruct exactly what state the codebase was in at any point in the session and replay the cascade forward.

**This phase is not about interventions. Not about the full marketplace. Just:**
1. Build the graph
2. Detect thrash at each layer
3. Replay when things go wrong

---

## What We Already Have

| Component | Status |
|-----------|--------|
| Layer 4 event capture (file edits, shell commands, git, tests) | Done — Sploink CLI wrapper |
| Layer 4 code graph ingestion (files as nodes, dependencies as edges) | Done — SPO-60 |
| Layer 1 intent parsing (goal attribution, sub-goal tagging) | Done — SPO-29 |
| Flight recorder (append-only raw event store) | Done — SPO-24 |
| Session replay infrastructure (static) | In progress — SPO-79 |
| Layer 4 thrash scoring (convergence, thrash flag, momentum) | Done — metrics.py |

**What we don't have:** Layers 2 and 3. Staleness propagation. Per-layer thrash scorers. Animated replay tied to graph state.

---

## The 4-Layer Graph

```
Layer 1 — Intent
  What was asked. The original goal and sub-goals.
  Source: initial prompt parsed by goal attribution (SPO-29)
  Nodes: parent_goal, sub_goals
  Edges: sub_goal → parent_goal

Layer 2 — Features
  What the code is supposed to do. PRD-level functionality.
  Source: parsed from PRD, README, function docstrings, file structure
  Nodes: features, user-facing behaviors
  Edges: feature → files that implement it

Layer 3 — Design
  Why it was built this way. Constraints and decisions.
  Source: architecture docs, inline comments, commit messages, docstrings
  Nodes: design decisions, interface contracts, architectural constraints
  Edges: design_decision → files it governs

Layer 4 — Code
  The actual structure. Already built.
  Source: static analysis + Sploink live events
  Nodes: files, functions, classes, modules
  Edges: imports, function calls, inheritance, shared state
```

---

## Step-by-Step Build Plan

---

### Step 1 — Define the Graph Schema
**Owner:** Sarvesh
**Dependency:** None — this is day one
**Output:** A locked data model before any code is written

Define the node and edge schema for all 4 layers. Every node needs:
```json
{
  "id": "unique identifier",
  "layer": 1 | 2 | 3 | 4,
  "type": "goal | feature | design_decision | file | function",
  "label": "human readable name",
  "stale": false,
  "stale_reason": null,
  "last_modified": "timestamp",
  "connected_to": ["node_ids"]
}
```

Every edge needs:
```json
{
  "from": "node_id",
  "to": "node_id",
  "relationship": "implements | governs | depends_on | sub_goal_of",
  "layer_crossing": true | false
}
```

Layer-crossing edges are the critical ones. They connect Layer 4 nodes upward to Layer 3, 2, and 1. This is what makes staleness propagate all the way up.

**Done when:** Schema is documented and reviewed. No ambiguity about what each node type represents.

---

### Step 2 — Extend Layer 4 Code Graph
**Owner:** Sarvesh
**Dependency:** Step 1 schema locked
**Output:** Layer 4 fully populated from a real codebase

SPO-60 already parses files and dependencies. Extend it to capture:
- **CPG (Code Property Graph)** — functions as nodes, calls as edges
- **Module dependency graph** — which modules import which
- **Route dependency graph** — API routes and their handler chains
- **Co-edit history** — which files have been edited together in the same session (from flight recorder)

The co-edit history is important. It's a behavioral signal on top of the static structure — files that always get edited together are implicitly coupled even if the static graph doesn't show it.

**Done when:** Given any repo, the system produces a fully populated Layer 4 graph in under 10 seconds.

---

### Step 3 — Build Staleness Propagation Engine
**Owner:** Sarvesh
**Dependency:** Step 2 complete
**Output:** Working proof of concept — edit a file, get back the full stale dependency map

This is the core primitive. When a `file_edit` event fires from Sploink:

```
1. Locate the edited file as a node in Layer 4
2. Mark it modified (not stale — it was intentionally changed)
3. Traverse all outbound edges from that node
4. For each connected node: mark STALE, record reason ("depends on [modified file]")
5. For each newly stale node: traverse its outbound edges
6. Continue until no new stale nodes are found
7. For any stale Layer 4 node: traverse layer-crossing edges upward
8. Mark connected Layer 3, 2, 1 nodes as STALE
9. Return: full stale node map with propagation path
```

Performance target: full propagation on a 10k file repo in under 500ms.

**The MVP demo:**
```
Input:  file_edit → auth.py
Output: {
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

**Done when:** This output is produced correctly and fast on a real codebase.

---

### Step 4 — Populate Layer 3 (Design Constraints)
**Owner:** Sarvesh
**Dependency:** Step 1 schema
**Output:** Layer 3 nodes extracted from an existing codebase

Start rule-based. Don't use LLM yet — keep it fast and deterministic.

Extract design nodes from:
- Comments containing keywords: `# decision:`, `# constraint:`, `# why:`, `# NOTE:`, `# IMPORTANT:`
- Docstrings that describe architectural choices
- README sections titled "Architecture", "Design", "Decisions"
- Interface definitions (abstract classes, Pydantic models, TypeScript interfaces) — these are implicit contracts

Each extracted design node gets linked to the files it governs (Layer 4 nodes).

**A design constraint is violated when:**
- A file it governs is edited in a way that changes its interface signature
- A function it governs changes its return type or parameters
- A file it governs is deleted or renamed

**Done when:** Given a real codebase, the system extracts at least 10 meaningful design nodes and correctly links them to the files they govern.

---

### Step 5 — Populate Layer 2 (Features)
**Owner:** Sarvesh
**Dependency:** Step 1 schema, Step 2 complete
**Output:** Feature nodes mapped to implementing files

Two sources:

**Source A — PRD/docs parsing:**
Parse `README.md`, any `docs/` folder, `FEATURES.md` if it exists. Extract feature descriptions. Map them to files based on naming patterns and keyword matching.

**Source B — Function/file clustering:**
Group files by co-edit history (from flight recorder) and import relationships. Files that always change together and import each other likely implement the same feature. Name the cluster based on the dominant directory or function names.

Start with Source A. Source B is the smarter version for later.

**Done when:** For a real codebase with a README, at least 5 feature nodes are extracted and linked to implementing files.

---

### Step 6 — Wire Sploink Events into Graph in Real Time
**Owner:** Sarvesh + Ved
**Dependency:** Steps 2-5 complete
**Output:** Graph updates live as the agent works

When Sploink emits any event, it also calls the graph update API:

```
file_edit event →
  graph.update(file="auth.py", change_type="edit", lines_changed=12)
  → triggers staleness propagation
  → returns stale map
  → stale map appended to session event stream

shell_command event (exit_code=1) →
  graph.flag_error(command="python test.py", file_context=["auth.py"])
  → links error to files currently being edited
  → marks test nodes as failing
```

The graph state at any timestamp = the cumulative result of all events up to that point. This is what makes replay possible.

**Done when:** Running a real Sploink session populates and updates the graph in real time without adding more than 100ms latency to event processing.

---

### Step 7 — Per-Layer Thrash Scorers
**Owner:** Scoring lane hire (or Raj if cleared)
**Dependency:** Steps 3-6 complete — graph must be live before scorers can consume it
**Output:** 4 thrash scores, one per layer, each between 0 and 1

**Layer 4 thrash score** (already partially exists in metrics.py — extend it):
```
signals:
  - edit_frequency[file] > threshold → file-level loop
  - stale_node_count growing → cascade expanding
  - same shell command failing N times → command loop
  - diff_size volatile (growing then shrinking) → churn

formula:
  L4 = 0.35 * edit_loop + 0.35 * cascade_growth + 0.20 * command_loop + 0.10 * churn
```

**Layer 3 thrash score:**
```
signals:
  - design_constraints_violated: count of Layer 3 nodes marked stale
  - interface_contracts_broken: functions that changed signature
  - architecture_decisions_overridden: Layer 3 nodes whose governing files changed

formula:
  L3 = 0.50 * constraints_violated + 0.30 * contracts_broken + 0.20 * decisions_overridden
```

**Layer 2 thrash score:**
```
signals:
  - features_affected: how many feature nodes are stale
  - feature_conflict: two features sharing a dependency both stale simultaneously
  - scope_expansion: features affected growing beyond original sub-goals

formula:
  L2 = 0.40 * features_affected_rate + 0.40 * feature_conflict + 0.20 * scope_expansion
```

**Layer 1 thrash score:**
```
signals:
  - intent_distance: how far current execution path has drifted from original sub-goals
  - sub_goal_completion_rate: are sub-goals being completed or abandoned
  - scope_drift: agent working on files outside original intent scope

formula:
  L1 = 0.50 * intent_distance + 0.30 * scope_drift + 0.20 * sub_goal_abandonment
```

---

### Step 8 — Composite Thrash Score + Dominant Layer Detection
**Owner:** Scoring lane hire
**Dependency:** Step 7 complete
**Output:** One composite score + one dominant layer label

```python
thrash_score = (
    0.40 * L4 +   # weight highest early — most observable
    0.25 * L3 +   # design violations are serious
    0.20 * L2 +   # feature conflicts matter
    0.15 * L1     # intent drift is real but hardest to measure precisely
)
thrash_score = clamp(thrash_score, 0, 1)

dominant_layer = argmax(L4, L3, L2, L1)
```

**Dominant layer maps directly to intervention type:**
```
L4 dominant → "implementation loop — break the local cycle"
L3 dominant → "design violation — restore the interface contract"
L2 dominant → "feature conflict — resolve at the feature boundary"
L1 dominant → "scope drift — reset to original intent"
```

**Done when:** A drifting test session produces a composite score > 0.7 with a correct dominant layer identified.

---

### Step 9 — Error Replay
**Owner:** Ved
**Dependency:** Step 6 complete (graph state timestamped), SPO-79 (session replay)
**Output:** Scrub through a session and see exact graph state at any moment

The flight recorder (SPO-24) stores every raw event with a timestamp. The graph update log stores every state change with a timestamp. Together they let you reconstruct:

- What files were stale at t=180s
- What the thrash score was at t=180s
- Which layer was dominant at t=180s
- What the agent did next

**The replay UI shows:**
```
Timeline scrubber → drag to any point in session

At each timestamp:
  - Convergence curve position
  - Codebase heatmap (which files were hot)
  - Stale node map (which nodes were stale, highlighted in red)
  - Per-layer thrash scores (sparkline for each layer)
  - Dominant layer label
  - The event that triggered the last staleness propagation
```

**The error replay use case specifically:**
When a test fails or a build crashes, replay backwards to find the exact edit that triggered the cascade. The graph shows the propagation path. The developer sees: "this test failed because `session_store.py` went stale when `auth.py`'s interface changed at t=150s."

**Done when:** Given a recorded session with a known failure, the replay correctly identifies the root cause event and shows the full propagation path.

---

## What This Phase Does NOT Include

- Automated intervention firing (that's Phase 2)
- Skill matcher or intervention skills registry
- Marketplace routing
- LLM-powered design node extraction (rule-based only for now)
- Multi-agent session support
- Layer 2 Source B (behavioral clustering) — rule-based feature mapping only

---

## Definition of Done for This Phase

1. Edit a file → get back the full stale dependency map across all 4 layers in < 500ms
2. Run a Sploink session → graph updates live with no more than 100ms added latency
3. Composite thrash score computed from 4 per-layer scores with dominant layer identified
4. Replay scrubber shows exact graph state + thrash scores at any session timestamp
5. Error replay correctly identifies root cause of a known test failure from graph propagation path

---

## Owner Summary

| Step | What | Owner |
|------|------|-------|
| 1 | 4-layer graph schema | Sarvesh |
| 2 | Extend Layer 4 code graph | Sarvesh |
| 3 | Staleness propagation engine | Sarvesh |
| 4 | Layer 3 population (design) | Sarvesh |
| 5 | Layer 2 population (features) | Sarvesh |
| 6 | Wire Sploink events into graph | Sarvesh + Ved |
| 7 | Per-layer thrash scorers | Scoring lane hire |
| 8 | Composite score + dominant layer | Scoring lane hire |
| 9 | Error replay | Ved |
