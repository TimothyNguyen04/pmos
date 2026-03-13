# Sploink — Phase 1 Team Doc

**Last updated:** March 13, 2026
**Deadline:** Sunday, March 15, 2026
**DRI:** Timothy Nguyen

---

## What Phase 1 Is

Measurement only. No intervention, no routing, no Salaxia wiring yet.

We're building three measurement signals that flow into one canonical schema. Each person owns one signal. All three need to work together before anything else gets built on top.

```bash
sploink run -- claude "refactor the auth module"
```

One command. Sploink wraps the session and captures everything happening at the system level in real time.

---

## The Three Signals

### 1. System Behavior — Dhrit
What the agent is actually doing at the execution layer.

| Event | What it captures |
|---|---|
| Shell commands | Every command the agent issues |
| File edits | Which files, how often, how much changed |
| Git state | Diff size over time, commit frequency |
| Test runner output | Pass/fail trajectory |
| Build output | Compiling, errors, success |
| Process spawns | What tools the agent invokes |

**Output:** A stream of system events showing the raw behavioral trace of the agent session.

### 2. Drift from Error — Aryan
How far the agent has deviated from a productive path, measured through error signals.

| Signal | What it captures |
|---|---|
| Error rate trajectory | Is the error count going up, down, or flat? |
| Test regression | Were passing tests broken by recent edits? |
| Repeated failure patterns | Same error appearing across multiple iterations |
| Recovery time | How long between an error and a corrective action |

**Output:** A drift score that quantifies how lost the agent is relative to a convergent path.

### 3. Environment — Ved
The full composition of the agent: model, tools, skills, scaffolding, and task framing. This is the context that everything else is measured against.

| Signal | What it captures |
|---|---|
| Model identity | Which model is running (claude-code, cursor, etc.) |
| Tool configuration | What tools are available to the agent |
| Skills loaded | Which skills or prompts are active |
| Scaffolding | The framework or wrapper the agent runs inside |
| Task framing | How the task was described at session start |

**Output:** An environment snapshot captured at session start, attached to every event so the other two signals are always contextualized.

---

## Canonical Event Schema

All three signals write to the same schema. This is the contract everyone builds against.

```json
{
  "session_id": "uuid",
  "task_id": "uuid",
  "timestamp": "ISO-8601",
  "event_type": "file_edit | shell_command | git_commit | test_run | build | process_spawn | error | environment_snapshot",
  "source": "system | environment | drift",
  "payload": {
    "path": "/src/auth.py",
    "lines_changed": 14,
    "cumulative_diff_size": 847
  },
  "agent": "claude-code | cursor | custom",
  "metadata": {}
}
```

The `source` field tells you which signal layer the event came from. Everything is append-only and immutable.

---

## Ownership

| Signal | Owner | What they build |
|---|---|---|
| System behavior | Dhrit | CLI wrapper, file/shell/git/test/process capture |
| Drift from error | Aryan | Error rate, test regression, repeated failure, recovery time |
| Environment | Ved | Model, tools, skills, scaffolding, task framing snapshot |
| Canonical schema | Timothy + all three | Agreed before anyone writes a line of capture code |
| Event storage | Ved | Append-only store, all three signals write here |

---

## What's NOT in Phase 1 Part 1

- Intervention or routing (no Salaxia wiring yet)
- Dashboard (comes after signals are stable)
- POST /execute endpoint
- Telemetry emission to Salaxia
- Scoring or ranking

Those come after we have clean, stable measurement from all three signals.

---

## Critical Path to Sunday

1. **Lock the schema** — everyone aligns on event types, field names, and the `source` taxonomy before writing capture code. One schema doc, all three sign off.

2. **Each signal captures real events** — Dhrit, Aryan, and Ved each produce a working event stream from a live agent session against the agreed schema.

3. **Events flow into shared storage** — all three streams land in the same append-only store with correct `session_id` linking them together.

4. **One end-to-end test** — run `sploink run -- claude "write a prime checker"`, confirm all three signal streams are captured and stored correctly for the same session.

---

## Success Criteria for Sunday

- Schema is finalized and documented
- System behavior stream captures shell commands, file edits, and git state for a live Claude Code session
- Drift stream captures at least error rate and test regression for the same session
- Environment snapshot is captured at session start and attached to the session record
- All three streams share the same `session_id` and are queryable together
- One clean test session on record

---

*Questions? Ping Timothy. Issues tracked in Linear under the Sploink project.*
