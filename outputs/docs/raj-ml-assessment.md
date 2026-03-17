# Raj — ML Engineer Skill Assessment

## Candidate
**Name:** Raj
**Role:** AI/ML Engineer (Divit replacement — behavioral scoring lane)
**Date sent:** 2026-03-17
**Deadline:** 2026-03-19 (48 hours)
**Status:** Pending submission

---

## Candidate Background
- **Vibe Engine** (9 months, full-time) — AI Software Developer. GEO (Generative Engine Optimisation), backend systems, GenAI applications including chatbots and voice agents.
- **AI VC** (1 year 2 months, part-time) — AI Software Engineer. Custom AI agents for landing page generation, web scraping, lead generation.
- **Neugence Technology** (4 months, internship) — ML Engineer. ML pipelines, fine-tuning open source LLMs (T5) on custom data.

**Assessment:** Applied ML background. Practical over research-heavy. Python-native. Has built real agents and fine-tuned models. Not a pure theorist — which is fine. The scoring lane requires mathematical thinking + clean implementation, not academic research. The question is whether he can translate a behavioral spec into original math.

---

## What Lane He's Filling

Divit was supposed to own:
- SAL-28 — Environment quality scorer
- SAL-31 — Drift from error signal
- SAL-29 — Behavioral signal → skill matcher
- SAL-26 — POST /intervention endpoint

Divit didn't engage. Raj is being evaluated to own this lane.

**The scoring lane is the intelligence layer of Sahlia.** It takes raw telemetry from Sploink and turns it into behavioral signals that drive interventions, marketplace routing, and eventually the agent improvement predictor. If Raj owns this well, he's a critical long-term hire.

---

## The Task

### Implement a Drift from Error Scorer

Given a sequence of agent session events, implement a Python function that computes a **drift from error score between 0 and 1**.

A score of **0** means the agent is clean — errors are resolving, tests are passing, no regression.
A score of **1** means maximum drift — errors are compounding, tests are failing, the agent is spiraling.

### The score must capture four sub-signals:

**1. Error rate trajectory (weight: 0.35)**
Are errors increasing or decreasing over time? Compare error count in the first half of the session vs the second half. Increasing = drift. Decreasing = recovery.

**2. Test regression depth (weight: 0.35)**
Did passing tests start failing? Measure the drop from peak passing rate to current passing rate. A session that starts with passing tests and ends with failing tests is drifting.

**3. Novel error rate (weight: 0.20)**
Are new error types appearing that weren't present earlier? Each novel error type that appears is a signal of expanding failure surface.

**4. Retry escalation (weight: 0.10)**
Is the agent retrying the same failing command without modification? Consecutive identical failing shell commands = escalating retry behavior.

### Final formula:
```
drift_score = (
    0.35 * error_rate_trajectory +
    0.35 * test_regression_depth +
    0.20 * novel_error_rate +
    0.10 * retry_escalation
)
```
Score is clamped to [0, 1].

---

### Sample Event Data

```json
[
  {"timestamp": 0, "type": "test_run", "passed": true, "errors": []},
  {"timestamp": 60, "type": "test_run", "passed": false, "errors": ["TypeError: undefined"]},
  {"timestamp": 120, "type": "shell_command", "command": "python test.py", "exit_code": 1},
  {"timestamp": 180, "type": "shell_command", "command": "python test.py", "exit_code": 1},
  {"timestamp": 240, "type": "test_run", "passed": false, "errors": ["TypeError: undefined", "KeyError: missing key"]},
  {"timestamp": 300, "type": "shell_command", "command": "python test.py", "exit_code": 1},
  {"timestamp": 360, "type": "test_run", "passed": false, "errors": ["TypeError: undefined", "KeyError: missing key", "AttributeError: NoneType"]},
  {"timestamp": 420, "type": "shell_command", "command": "python test.py", "exit_code": 1},
  {"timestamp": 480, "type": "file_edit", "file": "auth.py", "lines_changed": 12},
  {"timestamp": 540, "type": "test_run", "passed": false, "errors": ["TypeError: undefined", "KeyError: missing key"]}
]
```

---

### Deliverables

1. **`drift_scorer.py`** — the scoring function with clear docstrings. Must handle edge cases: empty session, session with no errors, session with only shell commands and no test runs.

2. **`test_drift_scorer.py`** — exactly 3 test cases with meaningful assertions:
   - **Clean session** — agent starts and ends with passing tests, no retries. Expected score: < 0.2
   - **Drifting session** — errors compound, novel errors appear, retries escalate. Expected score: > 0.7
   - **Recovering session** — starts bad, errors resolve by end, tests start passing again. Expected score: < 0.4

3. **A written note (5-10 lines)** covering:
   - Why you chose the weights you chose (or why you kept/changed the suggested ones)
   - How you handled the edge case of a session with no test runs
   - One thing you'd improve if you had more time

**Time:** 48 hours
**Language:** Python only. No external ML frameworks needed — pure math and data structures.

---

## What We're Actually Testing

| Dimension | What good looks like |
|-----------|---------------------|
| Mathematical thinking | Translates behavioral description into weighted formula. Doesn't hardcode thresholds arbitrarily. |
| Edge case handling | Empty session, no test runs, single event — handles all without crashing. |
| Test quality | Tests are meaningful, not just "assert score > 0". Clean, drifting, recovering sessions each tell a story. |
| Code clarity | Docstrings, variable names that explain intent, readable without explanation. |
| Written reasoning | Can articulate tradeoffs. Knows what they don't know. |
| Independence | Submits without follow-up. May ask one clarifying question but doesn't need hand-holding. |
| Speed | Submits before deadline. Early submission = high bandwidth signal. |

---

## Evaluation Rubric

**Hire (strong yes):**
- Formula is original and defensible
- All 3 test cases are meaningful with correct expected ranges
- Written note shows genuine reasoning
- Submits in < 36 hours
- Goes beyond the spec (e.g. adds a 4th test, surfaces an edge case we didn't think of)

**Conditional (verify further):**
- Formula works but weights are arbitrary with no reasoning
- Test cases pass but are thin
- Written note is vague
- Submits right at deadline
- Needs one follow-up nudge

**Pass:**
- Formula is hardcoded or doesn't implement all 4 sub-signals
- Test cases only test happy path
- Written note missing or one sentence
- Ghosts or misses deadline
- Copies existing scoring code without understanding it

---

## Context: Why This Task Specifically

This is SAL-31 — drift from error signal — which was in Divit's lane. It's the behavioral scoring primitive that:
- Feeds the intervention trigger (SPO-33) with a real signal
- Powers the lightweight prompt suggestions (SPO-83)
- Eventually feeds Sahlia's skill matcher and agent improvement marketplace

If Raj can implement this cleanly, he's not just a hired hand — he understands the core measurement primitive that makes the whole system defensible. That's the lane.

---

## Follow-up Signal to Watch

- **Asks a clarifying question** — good. Shows he read the spec carefully.
- **Submits early** — great. Real bandwidth and drive.
- **Goes beyond the spec** — cofounder signal. Watch carefully.
- **Vague questions ("what should the output be?")** — warning. Didn't read the spec.
- **Silence until deadline** — watch. May still deliver. Reserve judgment.
- **Ghosts** — Aryan/Divit pattern. Move on immediately.

---

## Decision Timeline

| Date | Action |
|------|--------|
| 2026-03-17 | Task sent |
| 2026-03-19 | Deadline — review submission |
| 2026-03-19 | Decision: hire, conditional, or pass |
| 2026-03-20 | If hire: onboard to GitHub + Linear, assign SAL-31 officially |
