# Meeting Notes: Team Sync + Gaurav Onboarding

**Date:** March 20, 2026
**Attendees:** Timothy (founder), Prabh (VS Code extension), Param (world model / L4), Gaurav (new — L1-3)
**Type:** Team sync + onboarding
**Duration:** ~25 min

---

## Summary

Onboarded Gaurav into the three-pronged architecture. Assigned him Layers 1-3 (entity extraction, graph construction, marketplace router) while Param owns Layer 4 (scoring, convergence/divergence). The two need to define the L1-3 / L4 contract. Prabh is grinding through SPO-93 on Windows but hit recurring setup bugs. Launch pushed to Tuesday due to a team member's family situation. OpenCode support flagged as a gap — needs a new issue.

---

## Decisions Made

1. **Gaurav owns Layers 1-3 + marketplace router**
   - Entity extraction from natural language prompts
   - Graph construction (L1, L2, L3)
   - Router between graph context and marketplace skills
   - Directly complementary to Param's Layer 4 work

2. **Param + Gaurav must define the L1-3 / L4 contract**
   - They need to agree on the interface between the layers
   - How context flows from the upper graph into Layer 4 scoring
   - This is the foundational integration point for the world model

3. **Launch pushed to Tuesday, March 24**
   - Team member out due to family issues but working 14 hrs/day when back
   - Three-day delay from original target

4. **SPO-93 deferred**
   - Prabh to continue grinding through it but acknowledged it's long and tedious
   - Timothy: "we can talk about SPO-93 another time when we have time"

---

## Action Items

| Task | Owner | Due Date | Priority |
|------|-------|----------|----------|
| Define L1-3 / L4 contract between graph layers | Param + Gaurav | ASAP | High |
| Create Linear issue for OpenCode support | Timothy | Today | Medium |
| Delegate OpenCode issue between Gaurav and Prabh | Timothy | Today | Medium |
| Continue SPO-93 (Windows setup + divergence detection) | Prabh | Ongoing | High |
| Fix recurring header bug (thought it was fixed) | Prabh | ASAP | High |
| Send Gaurav's tasks / frontend work from Prabh while he's out | Timothy | Today | Medium |

---

## Key Insights

**Prabh's blockers (Windows setup):**
- OpenCode: "command was not found" — likely a PATH issue after install
- Header bug: thought it was fixed, now recurring — unclear root cause
- Sploink currently supports Claude Code and Cursor. Custom agent path doesn't handle OpenCode. This is a real gap.

**Gaurav's onboarding reaction:**
- Started working on first issue during the call — good signal
- Dropped connection mid-meeting, came back. Caught the context injection + divergence explanation on re-entry.
- First issue is behavioral forms — still getting oriented in the codebase/docs

**Architecture framing Timothy gave Gaurav:**
> "Whenever there is divergence... what you're gonna work on is context injection where we take stuff from the marketplace and try to push the agent back into the right course of action."

This is the clearest single-sentence description of Gaurav's role. Worth saving as canonical framing.

---

## Open Questions

- [ ] What is the exact L1-3 / L4 interface contract? — Param + Gaurav to define
- [ ] Who takes which of Prabh's frontend tasks while he's out? — Timothy to assign
- [ ] OpenCode support: is this Gaurav or Prabh's issue to own? — Timothy to decide when creating ticket
- [ ] What is Gaurav's first issue exactly? (garbled in transcript) — Check with him in WhatsApp

---

## Blockers

1. **Prabh on Windows — setup still not clean**
   - OpenCode PATH issue
   - Recurring header bug
   - Impact: slowing progress on SPO-93 and VS Code extension work
   - Resolution: may need to document a Windows-specific setup guide or just lean Claude Code / Cursor

2. **OpenCode not supported**
   - Sploink only supports Claude Code + Cursor officially
   - Custom agent path doesn't work with OpenCode
   - Needs a new issue + ownership

---

## Next Steps

**Today:**
- Timothy creates OpenCode support issue and assigns it
- Timothy sends Gaurav the frontend tasks Prabh is stepping back from
- Param + Gaurav sync on the L1-3 / L4 contract (WhatsApp or quick call)

**Before Tuesday launch:**
- Prabh resolves header bug + SPO-93 progress
- Gaurav completes first issue
- Contract between Param and Gaurav defined and documented

---

## Context

- Prabh has been grinding hard — Timothy explicitly told him to take a break. Watch for burnout.
- Gaurav dropping connection mid-call is a signal to prefer async (WhatsApp) for Gaurav given India timezone + connectivity.
- The three-pronged split (Prabh = front end/VS Code, Param = L4 world model, Gaurav = L1-3 + router) is now the canonical team structure for Phase 1.
