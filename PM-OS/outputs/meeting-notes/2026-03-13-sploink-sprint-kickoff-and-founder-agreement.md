# Meeting Notes: Sploink Sprint Kickoff + Founder Agreement

**Date:** March 13, 2026
**Attendees:** Timothy Nguyen, Ved Sharma, Aryan Pandit
**Type:** Team kickoff + legal walkthrough
**Duration:** ~20 minutes

---

## Summary

Timothy walked the team through the Phase 1 sprint plan (4-day, 2-person, telemetry focus), the full system architecture (B2B-first pivot, single node: low-level telemetry), and the founder agreement terms. Aryan and Ved were largely passive. Aryan hasn't read the Linear docs yet and couldn't engage on sprint risks or agreement terms. He will review and flag exclusions async.

---

## Decisions Made

1. **Sprint structure confirmed: 2-person for now**
   - Ved owns infrastructure (day 1)
   - Aryan owns telemetry model (day 1 — convergence/divergence/momentum math)
   - Third engineer (Dhrit or other) activates the three-signal split when they join
   - Target: 4 days

2. **B2B-first via power users**
   - Validate CLI with power users first, then expand to enterprise
   - No Salaxia integration yet — this sprint is measurement only
   - Phase 1 focus is a single node: low-level telemetry

3. **Founder agreement terms walked through**
   - NDA + IP assignment included in agreement
   - Aryan equity: 15% (verbally confirmed again, still open to renegotiate)
   - Vesting: 1-year cliff, 4-year standard schedule
   - Removal: majority vote required (2 of 3 founders)
   - Cause: crime, IP breach
   - Compensation: up to 20% of revenue once $50k/month for 2 consecutive months
   - Board: Timothy only for now. Expands to 3 (Timothy + one founder + one investor) post-funding
   - 83(b) election included (tax treatment on restricted stock at grant)

4. **Aryan exclusions: pending**
   - Timothy flagged that Aryan's grad work on agent error analysis may need carving out from IP assignment
   - Aryan will review the agreement and respond async

---

## Action Items

| Task | Owner | Due Date | Priority |
|------|-------|----------|----------|
| Read phase 1.1 doc on Linear + Sploink repo (extension) to understand telemetry math | Aryan | Mar 13 EOD | High |
| Start telemetry model: define convergence, thrash, momentum formulas | Aryan | Mar 14 | High |
| Write explanation doc of telemetry math (for external readers) | Aryan | Mar 14-15 | Medium |
| Set up infrastructure (phase 1 sprint day 1) | Ved | Mar 13-14 | High |
| Review founder agreement; flag any exclusions Timothy should carve out | Aryan | Mar 14 | High |
| Sign founder agreement with Aryan | Timothy | Before Mar 15 | Critical |
| Continue recruiting (Dhrit onboard timing) | Timothy | Ongoing | High |

---

## Key Insights

**On architecture:**
- The vision Timothy presented: CLI running agents in a terminal + convergence/divergence graph side by side. Read-only cockpit first. Human-on-the-loop, not relaunching.
- Agent runtimes are getting longer (10-20 min, sometimes days) — the mid-session intervention angle is the real product thesis.
- Phase 1 is static parameters. Phase 2 adds dynamic parameters via heuristics from the flight recorder pairing user-observed data with system-observed data.

**On the founder agreement:**
- Agreement currently says "Georgia corporation" — but per the decision to incorporate as Delaware C-Corp (Stripe Atlas or Clerky), this needs to be updated before signing. Do NOT sign the Georgia version.

---

## Open Questions

- [ ] What are Aryan's exclusions from IP assignment? — Aryan to flag — **By:** Mar 14
- [ ] Will Dhrit onboard before Sunday? What's the latest on that? — Timothy to follow up
- [ ] Is the agreement template updated to reflect Delaware C-Corp, not Georgia? — Timothy to verify before signing

---

## Engagement Observation (DRI note)

**Aryan was significantly less engaged than expected.** He hadn't read the Linear docs or phase 1.1 before the meeting, couldn't speak to sprint risks, and gave minimal responses throughout. When asked what worried him: "I don't know yet. I need to check what we're doing first."

This is a signal worth watching. Possible explanations:
- **Still onboarding mentally** — hasn't had time to read the docs, so has nothing to respond to yet. Possible given midterm conflict Thursday.
- **Passive communication style** — not a red flag if he executes well; just low verbal output in group settings.
- **Uncommitted** — hasn't fully decided he's in, especially before signing.

**Recommended action:** Before Thursday, send Aryan a short async check-in. Ask specifically: (1) did you read the phase 1.1 doc, (2) do you have what you need to start the telemetry model, (3) any blockers or questions on the agreement terms? Low-friction touch — keep it short, make it easy for him to respond.

---

## Next Steps

**Before Sunday Mar 15:**
- Aryan and Ved both executing their sprint tracks
- Founder agreement signed with both (Timothy committed to this)
- Confirm Delaware C-Corp entity is reflected in the agreement before signing

**If Dhrit joins:**
- Split into three-signal model: Dhrit (system behavior), Aryan (drift from error), Ved (environment/agent composition)
- S-32 and S-34 activated for Dhrit

