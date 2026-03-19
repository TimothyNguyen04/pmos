# S^3 — State of the Build
### New Team Member Onboarding Document
*March 17, 2026 — Confidential*

---

## 1. What We're Building and Why

AI coding agents (Claude Code, Cursor, Aider) have a fundamental problem nobody has solved: they drift. They edit the same file seven times. They run the same failing test over and over. They quietly spiral away from solving the problem while looking like they're making progress. And you have no way to see it happening.

**Sploink** is the observability layer that fixes this. It wraps any agent CLI session and captures a real-time event stream — file edits, shell commands, git state, test results, build outcomes. From those events, it computes in real time whether the agent is converging on a solution or thrashing.

**Salaxia** is the intelligence layer that sits on top. It receives Sploink's telemetry, scores agent behavior, and eventually routes tasks to the best-performing agent automatically. When an agent starts drifting, Salaxia fires an intervention — a targeted prompt that steers the agent back on course.

The full system loop:
```
User runs: sploink run -- claude "refactor auth module"
  → Sploink wraps the session
  → Captures all events in real time
  → Computes convergence, thrash, momentum scores
  → Emits telemetry to Salaxia at session end
  → Salaxia scores the session
  → Dashboard shows what happened
  → Interventions fire when drift is detected
```

**The long-term vision:** Sploink and Salaxia together become the orchestration infrastructure for the internet of agents. Not just coding agents — any agent taking any task. The behavioral telemetry we accumulate becomes a compounding data moat that no competitor can replicate. More sessions → better scoring → better interventions → better agents → more sessions.

---

## 2. The Two Products

### Sploink — The Capture Layer
Sploink is a Python CLI tool. Developers install it with `pip install sploink`, authenticate with their email, and run `sploink run -- claude "task"` instead of `claude "task"` directly. The wrapper intercepts the session, records everything, and emits structured telemetry.

**What Sploink captures:**
- `file_edit` — which files the agent touched, how many lines changed
- `shell_command` — every command, its exit code, duration
- `git_commit` — commit hash, files changed, diff size
- `test_run` — pass/fail, which tests, error messages
- `build` — exit code, duration, error output
- `process_spawn` — subprocesses the agent spawns

**What Sploink computes from those events:**
- **Convergence score** — is the agent making net progress toward a clean solution?
- **Thrash score** — is the agent looping on the same files/commands without resolution?
- **Momentum** — rate of change of convergence. Dropping momentum = early warning.
- **Drift from error** — are errors compounding, resolving, or staying flat?

**Where users see it:**
- Terminal dashboard — side panel running live alongside the agent session (like Granola)
- Web dashboard — full session view at `/session/<uuid>`, shareable
- Claude Code plugin — real-time metrics panel inside the IDE
- Session replay + score card — animated replay with a shareable score (like Wordle for coding)

---

### Salaxia — The Intelligence Layer
Salaxia is a Python FastAPI service hosted separately from the CLI. It receives Sploink's telemetry, runs scoring algorithms, and eventually fires interventions back to the session.

**Salaxia's current endpoints:**
- `POST /telemetry-ingest` — receives the full event stream from Sploink after a session ends
- `POST /intent` — receives the user's initial prompt, classifies intent, routes to the best agent
- `POST /score` — computes composite session score from raw telemetry
- `GET /ranking` — returns current agent rankings by capability
- `POST /intervention` *(not yet built)* — receives a behavioral signal, returns an intervention prompt

**Salaxia's scoring lane (the intelligence layer):**
Four signals compose into one session score:
1. **Environment quality score** — was the scaffolding set up for success? Separates "bad agent" from "bad environment"
2. **Drift from error score** — weighted composite: error rate trajectory (0.35) + test regression depth (0.35) + novel error rate (0.20) + retry escalation (0.10)
3. **Behavioral signal → skill matcher** — maps observed behavior to the right intervention skill
4. **Unified session score** — combines system behavior + drift + environment into one number

---

## 3. The Tech Stack

| Layer | Technology |
|-------|-----------|
| Sploink CLI | Python |
| Salaxia API | Python FastAPI |
| Contracts/schemas | Pydantic (in `s-3-contract` monorepo) |
| Database | Supabase (auth + session storage) |
| Hosting | VPS (Salaxia), PyPI (Sploink CLI) |
| Terminal dashboard | Textual/Rich (Python) |
| Web dashboard | Web (Python Flask/FastAPI serving) |
| Event schema | Canonical JSON schema defined in SPO-21 |

The codebase lives in two repos:
- `sploink` — the CLI capture layer
- `s-3-contract` — the monorepo containing Salaxia at `services/salaxia/` and shared Pydantic contracts

---

## 4. What's Been Shipped

### Foundation (Mar 12-13)
- **Dev environment** set up for the CLI wrapper
- **Canonical telemetry event schema** defined — the JSON contract every event must conform to. This is the backbone. Every emitter, every scorer, every consumer speaks this schema.
- **CLI wrapper capture layer** — intercepts `sploink run -- claude "task"`, spawns the agent as a subprocess, attaches file system watchers, shell command hooks, git state polling. Captures all 6 event types.
- **Flight recorder** — append-only raw event store keyed by `session_id`. Every session ever captured is stored immutably before any scoring touches it. This is how the data moat compounds — every session is training data for better scoring weights.
- **`POST /execute` endpoint** — programmatic session initiation, for CI pipelines and future Salaxia routing
- **Sploink → Salaxia wire** — at session end, Sploink POSTs the full event stream to Salaxia's `/telemetry-ingest`

### Measurement (Mar 13-14)
- **Convergence engine** — sigmoid scoring + momentum formula ported from SploinkRecruiter (a prior validated production system). Replaces the naive linear weighted sum with a properly normalized score that doesn't get stuck.
- **Environment + architecture telemetry** — captures scaffolding setup, tool config, system prompt structure, model parameters, context window saturation. Required to separate "bad agent" from "bad environment."
- **Task framing signal** — scores how specific vs ambiguous the initial prompt was. A narrow prompt produces different behavioral signatures than a broad one.
- **Goal attribution** — parses the initial prompt into sub-goals, tags every downstream event with which sub-goal it belongs to. This means we can score not just agents but individual skills within agents.
- **Codebase graph ingestion** — parses files, dependencies, and co-edit history into a queryable graph. Foundation for spatial visualization.

### User-Facing Surfaces (Mar 14-16)
- **Terminal dashboard v1** — side panel (Granola-style). Shows live convergence curve, codebase heatmap, token attribution, cost meter. Runs alongside the agent without tab-switching.
- **Web dashboard** — full session view at `/session/<uuid>`. Shows all telemetry, convergence curve, file heatmap. Shareable URL.
- **Landing page** — sploink.io with value prop copy and `pip install sploink` command
- **Download tracking** — PostHog analytics logs every install button click. Tells us what % of landing page visitors attempt to install.
- **Auth** — `sploink auth` prompts for email, server returns a token stored locally. Every session is now tied to a user identity.
- **Claude Code plugin** — real-time telemetry rendering inside the IDE. Was broken (worked in Cursor, not Claude Code). Fixed.
- **SEO + brand** — meta tags, page titles, logo fixed.

### Bug Fixes (Mar 15-16)
- **Heatmap field path** — heatmap was always rendering empty because `metadata.get("subtype")` should have been `payload.get("subtype")`
- **Git convergence normalization** — was dividing by last diff instead of max diff. If churn peaks mid-session then drops, the score exceeded 1.0.
- **test_passed field mismatch** — `shell_watcher.py` emitted `test_passed`, `metrics.py` expected `tests_passed`. Test signal was silently reading 0 for every session.

### Salaxia Rewrite (Mar 16-17)
- **Full TypeScript → Python FastAPI rewrite** — Aryan's original TypeScript Salaxia codebase was dropped entirely. Rebuilt in Python FastAPI inside the `s-3-contract` monorepo. All four endpoints implemented with Pydantic contracts.

### Pre-Beta Validation (Mar 15-16)
- `pip install sploink` verified end-to-end on a clean machine
- Telemetry emission to production Salaxia server confirmed working
- Pre-beta readiness checklist signed off

---

## 5. What's In Progress Right Now

| Issue | What | Owner | Status |
|-------|------|--------|--------|
| S3-19 | Reduce install to one command — `sploink init` configures Claude Code hooks once, user never thinks about Sploink again | Ved | In Progress |
| SAL-28 | Environment quality scorer — separate "bad agent" from "bad environment" by scoring scaffolding independently | **Unowned** | In Progress (stalled) |

---

## 6. What Still Needs to Be Done

### This Week — Launch Critical (Updated Mar 18)

**The core product loop is: detect bug signal → highlight location → surface intervention prompt.**
Everything below is ordered to build that loop end-to-end before anything else.

**SAL-36 — Deploy Salaxia to production** *(Overdue — do first)*
Salaxia runs locally but isn't hosted anywhere publicly accessible. The CLI calls Salaxia over the internet — it needs a production URL. Without this, nothing scores. Needs to be on a VPS with the FastAPI server running and a stable endpoint.

**SAL-17 — End-to-end integration test**
Full smoke test: user runs `sploink run`, events emit, Salaxia receives, scores compute, dashboard reflects. Nothing currently validates the full loop works as one system.

---

### The Core Product Loop — Ved's Priority Order

These five issues compose the next-generation debugger: bugs detected before they're visible, highlighted in the IDE, with a targeted intervention prompt surfaced inline.

**SAL-31 — Drift from error signal** *(blocks everything below)*
The pre-visibility bug detector. Takes the raw event stream and computes a `drift_score` plus exact timestamps for every threshold crossing. This is the signal that powers the IDE highlighting and the intervention prompt. Without it, everything downstream has nothing to work with.

Four sub-signals:
- Error rate trajectory (0.35 weight) — are errors increasing?
- Test regression depth (0.35 weight) — did passing tests start failing?
- Novel error rate (0.20 weight) — new error types appearing = expanding failure surface
- Retry escalation (0.10 weight) — same failing command retried without modification

Output: `drift_score` between 0 and 1 + `threshold_crossings` array with timestamps, signal type, and plain English label per crossing.

**SPO-85 — Live file highlighting in IDE extension** *(depends on SAL-31)*
While a session is running, highlight files in the IDE that behavioral signals indicate are at risk — before a bug is visible in output. Colored gutter indicators per signal type (red = novel error, orange = test regression, yellow = retry escalation). Hovering a highlighted file shows the signal label AND the intervention prompt from SAL-26 inline — one click to copy and paste into the agent. This is the complete three-second loop: see it, understand it, fix it.

Blocked by: SAL-31 (drift timestamps), SAL-26 (intervention prompt to surface inline).

**SAL-29 — Behavioral signal → skill matcher** *(depends on SAL-31)*
Maps observed behavioral signals to the right intervention prompt from the skills registry. High drift + test regression → "Run diagnostics before continuing." Retry escalation → "Break the loop — reframe the approach." Novel error → "New error type detected — investigate before proceeding." This is the brain behind the intervention.

**SAL-26 — `POST /intervention` endpoint** *(depends on SAL-29)*
Receives a behavioral signal (drift score, thrash flag, stall detected), consults SAL-29 (skill matcher), returns a targeted intervention prompt string. This prompt is what gets surfaced inline in SPO-85 when a developer hovers a highlighted file.

**SPO-84 — Pre-visibility bug detection markers on replay** *(depends on SAL-31 + SPO-79)*
Places colored markers on the session replay timeline at the exact timestamps where each behavioral signal crossed a detection threshold. Hovering a marker shows signal type, timestamp, and plain English label. Makes "this is where the bug became inevitable" visible.

---

### Scoring Lane — Unowned (Needs a Second Engineer)

**SAL-28 — Environment quality scorer**
Separates environment failure from agent failure. Scores scaffolding quality independently — task framing specificity, context window saturation, tool configuration.

**SAL-32 — Unified session score**
Composites system behavior (convergence/thrash/momentum) + drift from error (SAL-31) + environment quality (SAL-28) into one number that feeds agent ranking.

---

### Near-Term (Phase 2)

**SPO-79 — Session replay** *(audit trail, not the product)*
Animates the session timeline — scrub backward and forward through the session, watch convergence curve sync to file heatmap in real time. With SAL-31 markers, the replay tells a story: "this is when the bug became inevitable." Without them it's just animation. Build after the core loop is live.

**SPO-80 — Shareable session card URL**
Minimal shareable page at `/session/<uuid>/card` — drift score, convergence sparkline, top files, session duration. OG tags for Twitter sharing. Build after SPO-79.

**SPO-45 — Data fusion layer** *(needed when multi-layer scoring is live)*
Merges CLI telemetry and IDE telemetry into a unified session timeline. Required when SAL-32 composites all four graph layers — each layer produces signals on a different clock. Defer until SAL-32 is live.

**SAL-25 — Sub-goal context in Salaxia ingestion**
Process `goal_context` fields from incoming events. Enables scoring at sub-goal level, not just session level.

**SAL-24 — Skills scoring**
Score individual skills (prompts, tool chains, reasoning patterns) not just whole agents. Foundation for the skills marketplace.

**SPO-77 — Human-in-the-loop intervention UI**
Renders the intervention prompt in the dashboard with Accept/Dismiss. The human pulls the trigger — Salaxia suggests, the developer decides.

**SPO-49 — Desktop HUD**
Always-on-top floating window showing session state regardless of what app you're in. When thrash spikes, it demands attention.

**SPO-83 — Lightweight prompt suggestions**
When drift > 0.7 or thrash sustained > 5 minutes, auto-generate a recovery prompt and surface it. Simpler than the full SAL-29 routing — fires automatically.

---

### GTM — Not Started

These are the launch mechanics for getting the first 50 users:

| Issue | What |
|-------|------|
| S3-13 | Define GTM one-liner and positioning |
| S3-14 | Build reproducible demo session for launch content |
| S3-15 | Record cockpit demo video for Twitter |
| S3-16 | Write and time Show HN post |
| S3-17 | Build Twitter power user outreach list (20 targets) |
| S3-20 | Set up first 50 user tracker |
| S3-21 | Write Discord seeding script for Claude Code + Cursor communities |
| S3-23 | Post engineering recruitment on LinkedIn |

---

### Long-Term Roadmap (Phase 3+)

These exist in Linear but are not on the active path. Included for context:

- **SAL-40** — Agent sandboxing environment: any agent can join the network by running in the sandbox and getting quantified
- **SAL-41** — Agent behavior predictor: simulate the outcome of combining marketplace components
- **SAL-34/35** — Specialist agent training pipeline and marketplace submission API
- **SPO-59** — 3D codebase visualization: agents moving through a spatial graph of the repo in real time
- **SPO-64** — Critical path engine: decompose tasks into parallel agent trajectories
- **SPO-65** — Agent behavioral fingerprints: build execution profiles from accumulated session history
- **SAL-37** — Dynamic agent composition: assemble and decompose specialist agents based on live failure signatures
- **SPO-42** — iOS app: monitor and intervene on live sessions from anywhere

---

## 7. The Gaps Right Now

**Gap 1: Salaxia isn't deployed.**
SAL-36 is overdue. The scoring loop doesn't close until Salaxia is live on a production server. Every session Sploink captures right now is emitting to nowhere.

**Gap 2: The scoring lane has no owner.**
SAL-28, 31, 29, 26, 32 — the behavioral intelligence layer — was Divit's lane. He didn't engage. These five issues are the difference between Sploink being a dashboard and Sploink being an intelligent system that actually intervenes. Nobody is working on them.

**Gap 3: Session replay isn't started.**
SPO-79 is due tomorrow. This is the virality mechanic — the shareable score card that makes Sploink something developers post on Twitter. It's the most important GTM feature we haven't built.

**Gap 4: No GTM work has started.**
Every issue in the GTM section above is untouched. The product is ready to show people. Nobody is working on getting it in front of them.

---

## 8. The Team

**Timothy Nguyen** — Founder. Product, strategy, math (convergence/divergence formulas), recruiting, vision. Not a primary coder — drives architecture decisions, writes issues, ships vibe-codeable features.

**Ved (Vedaang Sharma)** — Cofounder-level engineer. Owns the entire Sploink build. Has shipped every major surface: CLI wrapper, dashboards, auth, landing page, Claude Code plugin, Salaxia rewrite, bug fixes. Fastest executor on the team. Strong technical judgment and culture filter.

**Sarvesh** — Prospective collaborator. Proposed hierarchical knowledge graphs as the context layer — 4-layer architecture (code graph → semantic/design choices → features → PRD) with dynamic stale propagation. Directly relevant to improving drift quantification. Not formally committed yet. In discussions.

**Raj (Rajatavaa)** — Prospective ML/AI hire. Submitted the marketplace prototype assessment. Under evaluation for the scoring lane.

---

## 9. How to Get Up to Speed Fast

1. Install Sploink: `pip install sploink && sploink auth`
2. Run a session: `sploink run -- claude "write a hello world script"`
3. Watch the terminal dashboard render live
4. Open the web dashboard at the URL it prints at session end
5. Read the canonical event schema (SPO-21) — this is the contract everything speaks
6. Read the Salaxia FastAPI service at `services/salaxia/` in the `s-3-contract` repo
7. Look at Linear — filter by your team (Sploink or Salaxia) to see what's assigned

The most important thing to understand architecturally: **Sploink measures, Salaxia thinks.** Every design decision flows from that separation.

---

*For questions: Timothy Nguyen — reach out directly.*
