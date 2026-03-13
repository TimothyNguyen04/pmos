# Meeting Notes: Team Strategy & Recruiting Review
**Date:** March 11, 2026
**Attendees:** Timothy Nguyen, Ben Moon
**Type:** Co-founder sync — strategy, team, process
**Duration:** ~45 minutes

---

## Summary

Wide-ranging co-founder sync covering team changes (Imran removal, Navneet onboarding, Mohit hold), equity structure for engineers, Linear hygiene, Claude workflow tips from Ben, competitive reconnaissance assignments, and honest feedback on vision stability and decision-making process. Sunday launch confirmed.

---

## Decisions Made

1. **Imran removed from team**
   - Rationale: Slow rolling work, using logistics as excuses (cursor credits, etc.)
   - All tasks reassigned to Ved
   - Imran removed from Linear

2. **Mohit — hold for now**
   - Timothy reviewed transcript and answers, didn't like them
   - Hold until better candidate available

3. **No third co-founder right now**
   - Agreement: dilution risk outweighs benefit at this stage
   - Future co-founder (if any): needs to be US-based, technically elite

4. **Ved equity structure (proposed)**
   - 5-10% on main S³ entity
   - Higher upside potential on Sploink spinout later
   - Ben aligned on keeping main split clean between Timothy and Ben

5. **PM-OS removed from shared repos**
   - Contains internal PM docs, interview notes, etc. — not for engineer visibility

6. **Big decisions require looping in Ben first**
   - Hiring, equity, banking, major architecture changes
   - Timothy committed to texting Ben before any medium/big call

7. **Sunday launch still on**
   - If they don't fully ship, the deadline will have accelerated progress regardless

---

## Action Items

| Task | Owner | Due Date | Priority |
|------|-------|----------|----------|
| Fix Linear issues — add descriptions, proper markdown, no escaped \n | Timothy | Mar 11 EOD | High |
| Assign issues to milestones in Linear | Timothy | Mar 11 EOD | High |
| Competitive recon: sign up for betas/demos for competitor list (trace.so, getden.io, emdash.sh, etc.) | Timothy | Mar 11 EOD | High |
| Screen record competitor UX flows | Timothy | Mar 11 EOD | Medium |
| Research Mercury vs Ramp, share pros/cons with Ben | Timothy | Mar 12 | Medium |
| Download recommended MCPs: context7, paper search, A0-based MCP | Timothy | Mar 11 | Medium |
| Download superpower skills from Discord | Timothy | Mar 11 | Medium |
| Go through full PM substack Ben shared | Timothy | This week | Medium |
| Remove PM-OS from shared S³ repos | Timothy | Mar 11 | High |
| Remove Imran from all Linear projects | Timothy | Done | Complete |
| Hold on Mohit — no onboarding | Timothy | — | Done |
| Loop Ben in on Navneet task assignments | Timothy | Mar 11 | Medium |

---

## Key Feedback from Ben (Process)

### 1. Linear Issues Need More Detail
Ben's standard: each issue has Context, What to Build, and Success Criteria — specific enough that an engineer never needs to ask a follow-up question. Current issues are too sparse. Fix: treat each issue like you're babysitting the engineer in the doc, not in WhatsApp.

### 2. Vision Changes Are Slowing Things Down
Biggest concern: scope or direction changes after work has started. Example: engineer is almost done with a view, Timothy introduces a new direction, everything has to restart. Creates confusion for engineers and delays launch.

**Request:** Come to consensus before work starts. Written, locked, shared. Constant changes — even valid ones — compound into real delays.

### 3. Document Management
Ben flagged a pattern of repeating discussions that were already settled. Root cause: decisions aren't being written down and referenced. Fix: one shared, organized doc layer where decisions live so neither has to re-litigate.

### 4. Loop Ben in on Big Decisions
This includes: hiring/onboarding engineers, equity discussions, banking setup, and major architecture decisions. These affect workflow and equity for both founders. Ben wants to be involved, not informed after the fact.

---

## Ben's Claude Workflow (Notes)

- **Teammate mode:** Splits terminal into multiple parallel agents with shared context — runs all of them simultaneously and they communicate with each other
- **Context7 MCP:** Pulls latest library documentation at inference time — prevents Claude from hallucinating outdated API patterns
- **Paper search MCP:** Searches actual research papers — used for data fusion, memory systems, proactive intelligence research for Seoro
- **Skill routing:** Superpower skills from Discord (research-codebase, create-plan, validate-plan, implement-plan) — go through each manually so you know when to invoke them
- **Linear workflow via Claude:** Start with blocking issues in project → work through dependency chain → if blocked by someone else, message them → move to unblocked work in parallel
- **Issue creation:** Give Claude the full codebase context + tell it to find gaps → create issues from that analysis with production-grade detail

---

## Competitive Recon Assignment (from Ben)

Ben assigned Timothy to research the following competitors, focused on Sploink/execution layer:
- trace.so
- getden.io
- emdash.sh
- (others TBD via WhatsApp from Ben)

**Method:** Sign up for betas, schedule demo calls if available, screen record UX flows, note architecture patterns even if no beta access. Goal: understand what they're building, how they're doing it, and what we can learn or avoid.

---

## Team Status

| Person | Status | Notes |
|--------|--------|-------|
| Ben | Active, on track | Front-end heavy week, doing "middle end" SwiftUI + OCR wiring |
| Ved | Active, CTO track | Completed 8 issues, stayed up until 2am. Equity docs before Sunday. |
| Aryan | Active | Meeting tonight 8pm PT on router decision (add vs defer) |
| Navneet | Starting | 4 tasks assigned (SEO-15, SEO-13, SEO-5, SPL-24) |
| Mohit | On hold | Answers/transcript didn't meet bar. Revisit later. |
| Imran | Removed | All tasks reassigned. Out of Linear. |

---

## Open Questions

- [ ] Will Aryan's router work get deferred or included for Sunday? — Timothy/Aryan — tonight
- [ ] What tasks does Ben want to assign Navneet beyond what's already in Linear? — Ben — Mar 12
- [ ] Mercury vs Ramp — which banking setup is right? — Timothy to research — Mar 12
- [ ] What does the shared decision/vision doc look like? — Timothy + Ben — this week

---

## Political Notes (Internal — Do Not Share)

Ben is asserting co-founder parity. Multiple signals in this call:
- Had to explicitly tell Timothy to loop him in on big decisions
- Flagged Linear quality, PM-OS visibility, vision changes, document management
- Wants to be involved in banking setup
- Asked about previous exit (transparency test)

None of this is hostile — he ended warm and is locked in for Sunday. But he's watching how Timothy operates, especially after seeing two people removed in quick succession. The pattern he's flagging (solo decision-making, constant scope changes, poor documentation) needs to be addressed with concrete process, not just acknowledgment.

**Highest priority follow-up:** Propose a shared decision doc / scope lock process before work starts on any new feature. Converts Ben's feedback into a system he can see working.
