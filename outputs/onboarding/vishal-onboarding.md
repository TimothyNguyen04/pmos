# S³ — What We're Building and Where You Fit

Hey Vishal,

Here's the full picture.

---

## The vision

Software is being written by AI agents now. Not assisted by them — written by them. Claude Code, Cursor, Devin, and whatever comes next will be the primary executors of engineering work within a few years. That's already true for some teams today.

But nobody can watch what an agent is actually doing. You start a session, the agent runs for 20 minutes, you come back and either it worked or it didn't. If it failed, you have no idea when it went wrong or why. If it succeeded, you have no idea how to replicate that. Agents are black boxes.

That's the problem. S³ is building the infrastructure layer that makes agents observable, measurable, and improvable over time. Not an AI product — the layer underneath AI products that makes them trustworthy enough to run at scale.

Think about your experience building the assessment platform at Amrita. Every time an LLM-generated question was wrong, you had no way to know *why* it was wrong or *when* it started going wrong. You saw the bad output, not the drift that caused it. Sploink solves that — for any agent, any task, any domain.

---

## The phases

**Phase 1 — Measurement (building now)**

We wrap the agent CLI. Every shell command, file edit, git commit, and test run gets captured as a telemetry event. From those events, we compute three signals in real time:

- *Convergence* — is the agent actually moving toward done, or just busy?
- *Thrash* — is it stuck in a loop, editing the same file over and over, running the same failing test?
- *Momentum* — is the session accelerating or decelerating?

The output is a live session score. For the first time, you can watch an agent work and know whether it's going somewhere.

**Phase 1.5 — Scoring intelligence**

The measurement layer captures raw events. The scoring layer turns those events into behavioral intelligence.

This is where your background is directly relevant. The scoring lane is a weighted signal computation problem — given a stream of low-level behavioral events, compute a drift score that accurately predicts whether an agent is converging or diverging before the outcome is visible.

Four signals compose into one drift score:
- Error rate trajectory (0.35) — are errors increasing over time?
- Test regression depth (0.35) — did passing tests start failing?
- Novel error rate (0.20) — new error types appearing = expanding failure surface
- Retry escalation (0.10) — same failing command repeated unchanged

The weights aren't arbitrary. They reflect how strongly each signal predicts a session failure. Calibrating them against real session data is an ongoing empirical problem — exactly the kind of thing your ML background is built for.

**Phase 2 — Intervention**

Once you can see what's happening, you can act on it. When drift is detected, the system surfaces a targeted intervention — a specific prompt or agent swap — to steer the session back on track.

The intervention layer maps behavioral signals to skills. High drift + test regression → "run diagnostics before continuing." Retry escalation → "break the loop, reframe the approach." This is where RAG and semantic matching become useful — finding the right intervention for a given behavioral signature from a registry of skills.

Your experience with Pinecone and vector databases applies directly here. The skill matcher is fundamentally a retrieval problem: given a behavioral signal, find the most semantically similar intervention in the skills registry.

**Phase 3 — Routing and marketplace**

Salaxia is our routing layer. It takes a task, decomposes it, and routes to the best agent. As execution data accumulates, agents develop measurable reputations. The marketplace surfaces this — you pick agents based on a track record you can actually see.

---

## The system loop

```
User runs: sploink run -- claude "refactor auth module"
  → Sploink wraps the session
  → Captures all events in real time
  → Computes convergence, thrash, momentum
  → Emits telemetry to Salaxia at session end
  → Salaxia scores the session
  → Dashboard shows what happened
  → Interventions fire when drift is detected
```

---

## Where you fit

The scoring lane is currently unowned. It's the most important open problem on the team right now.

Five issues compose into the full behavioral scoring system. This is yours if you want it:

**SAL-31 — Drift from error signal**
Weighted scorer that takes the raw event stream and computes a `drift_score` between 0 and 1 with timestamps for each threshold crossing. The foundation everything else builds on.

**SAL-28 — Environment quality scorer**
Separates "bad agent in a good environment" from "good agent in a bad environment." Scores the scaffolding quality independently so you can isolate whether the agent or the environment caused the failure.

**SAL-29 — Behavioral signal → skill matcher**
Maps observed behavioral signatures to the right intervention. This is the retrieval problem — your RAG and vector database background applies directly here.

**SAL-26 — Intervention endpoint**
FastAPI endpoint that receives a behavioral signal and returns a targeted intervention prompt. Consumes SAL-29's skill matcher.

**SAL-32 — Unified session score**
Composites all three signal streams into one number that feeds agent ranking in the marketplace.

---

## The tech stack

| Layer | Technology |
|-------|-----------|
| Sploink CLI | Python |
| Salaxia API | Python FastAPI |
| Contracts/schemas | Pydantic |
| Database | Supabase |
| Event schema | Canonical JSON (defined in SPO-21) |

The Salaxia service lives at `services/salaxia/` in the `s-3-contract` monorepo. Four endpoints already implemented: `/telemetry-ingest`, `/intent`, `/score`, `/ranking`. The scoring lane plugs directly into `/score`.

---

## The test task

Before anything else, here's the task. 48 hours.

**Implement the drift detector (SAL-31 core).**

**Input:** A JSON array of session events. Each event has:
```json
{
  "timestamp": 1742891234,
  "event_type": "shell_command",
  "payload": {
    "exit_code": 1,
    "command": "pytest tests/",
    "tests_passed": false,
    "error_message": "ModuleNotFoundError: No module named 'auth'"
  }
}
```

**Output:**
```json
{
  "drift_score": 0.73,
  "threshold_crossings": [
    {
      "timestamp": 1742891234,
      "signal": "test_regression",
      "value": 0.41,
      "label": "A previously passing test began failing"
    }
  ]
}
```

**Requirements:**
- Pure Python
- `drift_score` always between 0 and 1
- All four signals implemented with correct weights (0.35 / 0.35 / 0.20 / 0.10)
- `threshold_crossings` array empty for clean sessions — no false positives
- Use field name `tests_passed` not `test_passed`
- Three unit tests: clean session (~0), fully drifting session (~0.9), one sub-signal in isolation

**Part 2 — API layer**
Wrap your drift detector in a FastAPI `POST /score` endpoint:
- Accepts the event array as the request body
- Returns `drift_score` + `threshold_crossings`
- Pydantic models for input validation
- Returns a clean 422 with an error message for malformed input

**Part 3 — Frontend**
Build a minimal session score card page. Use the output from Part 2 as your data source (hardcode a sample response if needed).

The page should show:
- The drift score as a large number — green if < 0.4, yellow if 0.4–0.7, red if > 0.7
- A horizontal timeline with each threshold crossing rendered as a labeled marker at its position
- Session duration and total signals fired
- Dark background, clean layout — no design system required, React or plain HTML/CSS your choice

This doesn't need to be polished. It needs to work and be readable.

---

Submit all three parts as a single GitHub repo. 48 hours.

---

## Honest context

Pre-funding, pre-revenue. Two people right now — me and Ved, our platform engineer. Three days from beta. Applying to Y Combinator in 40 days.

It's equity only at the moment. You're in it because you want to build something that matters, not because of what's certain.

The scoring lane is the gap between Sploink being a dashboard and Sploink being a system that actually catches bugs before they happen. That's the problem. It's yours if you want it.

— Timothy
