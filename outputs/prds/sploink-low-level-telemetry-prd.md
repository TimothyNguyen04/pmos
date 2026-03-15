# PRD: Sploink — Low-Level Workspace Telemetry
*Imran Ullah | Ved complete | Status: In Progress*

## 0) Meta

| Field | Value |
|---|---|
| **Feature** | Low-Level Workspace Telemetry |
| **DRI** | Timothy Nguyen |
| **Engineer** | Imran Ullah |
| **Stage** | Team Kickoff |
| **Last Updated** | 2026-03-11 |
| **Status** | Draft |

---

## 1) Problem and Hypothesis

**Problem:**
Runtime telemetry tells us what an agent did and whether it succeeded. It does not tell us whether the agent is actually making progress. An agent can succeed on paper while thrashing internally — repeatedly editing the same files, looping on failures, oscillating between states. That behavior is invisible today.

**Hypothesis:**
If we capture fine-grained workspace events from agent execution (file edits, commands, diffs, build/test results), we can compute whether an agent is converging or thrashing — giving us the signal layer needed to power meaningful agent ranking, routing improvement, and architecture recommendations.

**Why now:**
Ved's infrastructure is complete. The backend can ingest and store events. The runtime layer exists. Low-level telemetry is the missing input that makes the metrics engine actually intelligent.

---

## 2) Scope and Non-Goals

**In Scope:**
- Canonical workspace telemetry event schema
- Event taxonomy: file, diff, command, build/test, verification, git, error/recovery
- CLI wrapper as the initial capture surface (wraps commands, captures git state, diffs, build/test outputs)
- Append-only event storage with per-session, per-task, per-agent grouping
- Metrics-readiness layer: schema must support future computation of thrash, convergence, momentum, churn

**Non-Goals:**
- IDE extension — Phase 2
- Computing thrash/convergence scores — Phase 2 (schema must support it, not implement it)
- Salaxia routing integration — separate PRD
- Polished dashboards — Ved's domain, already handled

**Tradeoffs Accepted:**
- CLI wrapper only for MVP. IDE extension adds richer signals but is harder to standardize. Ship the simpler surface first, keep schema compatible.
- Schema designed for future metric computation, not optimized for current query patterns.

---

## 3) What Imran Is Building

A telemetry system that captures how an agent interacts with files, commands, diffs, and verification steps — so the system can later determine whether the agent is converging, thrashing, or making real progress.

### Event Schema

Every workspace event must include:

```json
{
  "event_id": "string (uuid)",
  "timestamp": "ISO-8601",
  "session_id": "string",
  "task_id": "string",
  "agent_id": "string",
  "workspace_id": "string",
  "event_type": "string (see taxonomy below)",
  "source": "cli_wrapper | ide_extension | local_observer",
  "target_path": "string (file path or entity)",
  "payload": "object (event-specific data)",
  "sequence_index": "number"
}
```

Optional fields (include where available):
```json
{
  "correlation_id": "string",
  "parent_event_id": "string",
  "environment_type": "string",
  "git_commit_hash": "string",
  "branch_name": "string"
}
```

### Event Taxonomy

**File Events**
`file_opened`, `file_created`, `file_edited`, `file_saved`, `file_deleted`, `file_renamed`

**Diff Events**
`diff_generated`, `diff_applied`, `diff_reverted`

**Command Events**
`command_started`, `command_finished`, `command_failed`

**Build/Test Events**
`build_started`, `build_finished`, `build_failed`, `test_started`, `test_finished`, `test_failed`

**Verification Events**
`lint_started`, `lint_finished`, `verification_failed`, `verification_passed`

**Navigation Events**
`file_switched`, `symbol_searched`, `search_executed`

**Git Events**
`git_status_checked`, `git_diff_checked`, `commit_created`, `checkout_changed`

**Error/Recovery Events**
`error_observed`, `retry_triggered`, `rollback_triggered`

### Capture Layer — CLI Wrapper

Wrap agent execution commands and instrument:
- Commands: `git commit`, `git push`, test runners, `npm install`, build scripts
- Build/test output capture
- Diffs: before/after state on file modifications
- Git state: current branch, commit hash, dirty files

**Implementation principle:** source-agnostic design. The schema should not assume CLI forever — just start there.

### Storage Requirements

- Append-only event stream (never mutate past events)
- Queryable by: session_id, task_id, agent_id
- Must support reconstructing: event timeline, workspace trajectory, task progression
- Ved's backend already handles storage infrastructure — Imran needs to emit events in the correct schema and confirm ingestion works end-to-end

---

## 4) What the Data Must Eventually Answer

The schema Imran designs must support future computation of:

| Future Metric | Signals Required |
|---|---|
| **Thrash detection** | Repeated edits to same file in short window, revert/apply loops, repeated build failures with similar diffs |
| **Convergence** | Shrinking error count, fewer touched files over time, increasing verification pass rate |
| **Agent momentum** | Rate of successful progress events, reduction in blockers, positive build/test trajectory |
| **Editing churn** | Total edits per file, repeated edits without validation success, high edit volume with low net progress |

These do not need to be computed in this phase. The schema just needs to capture the right signals.

---

## 5) Success Criteria

Imran's work is complete when:

- [ ] A single agent session produces a structured, ordered workspace event stream
- [ ] All events are tied to `task_id` and `agent_id`
- [ ] The event stream can be queried in sequence order
- [ ] A future metrics engine could compute thrash/convergence from stored events without schema changes
- [ ] Design is extensible to IDE extension without breaking existing event consumers
- [ ] CLI wrapper captures at minimum: file edits, command execution, build/test results, retry counts, diff apply/revert

---

## 6) Risks

| Risk | Detection | Mitigation |
|---|---|---|
| Schema too narrow for future metrics | Review against thrash/convergence signal requirements before finalizing | Validate schema against each future metric in section 4 before shipping |
| CLI wrapper misses events in some agent frameworks | Test with LangChain, LangGraph, and a custom agent in Day 1 | Log all uncaptured events as `unknown_event` with raw payload for debugging |
| Events arrive out of order | Check sequence_index continuity on ingestion | Use sequence_index + timestamp as dual sort keys |
| Storage coupling — schema too tightly tied to Ved's current DB | Keep event format as pure JSON, decouple from DB schema | Imran emits JSON; Ved handles storage adapter |

---

## 7) Open Questions

- [ ] What is the minimum set of commands the CLI wrapper must intercept on Day 1? — Imran
- [ ] How does the CLI wrapper attach `task_id` and `agent_id` to events? Who sets these IDs? — Timothy + Imran
- [ ] Should `payload` be schema-typed per event_type, or freeform JSON for now? — Imran
- [ ] What's the ingestion endpoint format on Ved's side? — Imran + Ved

---

## 8) Next Steps

| Milestone | Exit Criteria |
|---|---|
| Schema finalized | All fields defined, event taxonomy complete, validated against future metric requirements |
| CLI wrapper emitting events | At minimum: file edits, commands, build/test results captured for one agent session |
| Events stored and queryable | Can retrieve ordered event stream by session_id and task_id |
| End-to-end test | One full agent session produces a complete, queryable workspace event stream |
