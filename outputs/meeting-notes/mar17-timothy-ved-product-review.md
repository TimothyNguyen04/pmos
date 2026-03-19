# Meeting Notes — Timothy + Ved Product Review
**Date:** March 17, 2026
**Participants:** Timothy (founder), Ved (engineer)
**Type:** Product review / build sync

---

## Key Decisions

**Extension + CLI architecture locked**
CLI is primary interface. Extension runs hidden in background. Both read different data types and congregate into a unified signal. No extension UI exposed to user — just the CLI. Eliminates interface confusion and user choice paralysis.

**Desktop app required for launch**
Moving between screens is bad UX. Desktop app gives all-in-one view. Launch needs to include the desktop app, not just CLI + web dashboard.

**Graph deferred — 2-layer reduction worth exploring**
Full 4-layer Sarvesh graph not needed before launch. Timothy floated a simplified 2-layer version: PRD layer (high-level prompts) + code layer (drift/thrash signals). This may capture 80% of the value at 20% of the complexity. Revisit before committing to full 4-layer implementation.

**Launch criteria = interventions, not just telemetry**
Timothy's view: the product is launchable when it can suggest the right prompts/interventions. This is a higher bar than "telemetry works" — aligns with Phase 2, not Phase 1. Tension with shipping fast — needs resolution.

**Metric calibration is the pre-launch blocker**
No formula derivable from first principles. Only path: run real sessions, eye test output, tune parameters, repeat. Timothy offered to supply API credits for this process. (SPO-93 created and assigned to Ved.)

---

## Product State (as observed in session)

- UI needs design upgrade — flagged as known gap
- Convergence showing ~3% — clearly miscalibrated, needs tuning
- Graph visualization looking good ("softy") — positive signal
- API server and main functionality working
- Extension still loading from local files — needs publishing before launch
- CLI install (`sploink init`) shipped today (S3-19 ✓)

---

## On Sarvesh

Both Timothy and Ved taking "wait and see" approach. Ved's read: BIT grad, solid full stack project, thought deeply about the architecture, articulate on details. Verdict depends on how he plans to implement. Timothy's filter: does he point himself at the right problem without being directed?

**Risk:** Aryan and Divit both showed passive engagement before being cut. Need a specific 48-hour signal from Sarvesh to differentiate.

---

## Hiring

Timothy continuing to talk to candidates. Current filter: backend/full stack skills, not AI. The scoring lane needs someone who can translate behavioral specs into weighted formulas — not an ML researcher.

---

## Action Items

| What | Owner | Issue |
|------|-------|-------|
| Empirically tune convergence/momentum parameters | Ved | SPO-93 |
| Session replay + shareable score card | Ved | SPO-79 |
| Desktop HUD | Ved | SPO-49 |
| Publish extension (move off local files) | Ved | — |
| Evaluate 2-layer graph reduction vs full 4-layer | Timothy | — |
| Define 48-hour Sarvesh signal | Timothy | — |
| Continue backend/full stack candidate pipeline | Timothy | — |
