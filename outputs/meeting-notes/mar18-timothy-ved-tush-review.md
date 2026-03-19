# Meeting Notes: Tush Candidate Review + Product Debrief
**Date:** March 18, 2026
**Attendees:** Timothy (founder), Ved (cofounder), Tush (candidate)
**Type:** Candidate evaluation + product roadmap sync
**Duration:** ~45 min

---

## Summary

Two-part session. First, Tush demoed his marketplace prototype work — company API key system, agent version tracking, partial marketplace browse. Timothy and Ved passed on him post-call: can write and understand code but weak on execution and explanation. Second, Timothy and Ved aligned on the core product loop, launch decision gates, and a major architecture shift: replace the desktop HUD with a VS Code-style IDE extension as the primary interface.

---

## Part 1: Tush Candidate Review

### What He Demoed
- **Company API key system** — pivoted from per-model credentials to one API key per company giving access to all agents in their portfolio. Key delivered via email for security. Active status shown in dashboard.
- **Agent version tracking ("evolution")** — compares older vs. newer versions of the same company's agent, outputs the newer version as the canonical one. Scoped to intra-company comparison only (not cross-company).
- **Browse marketplace** — basic listing of available agents.
- **Invite link system** — company creator can send invite links to team members.
- **Testing** — ran GitHub CI and Postman checks. Tests not currently passing. Admitted he doesn't fully understand how testing works yet.

### Agent Delegation Architecture (as explained by Tush)
User input → intent parsing → router → primary agent selected → sub-agents allocated (parent/child model) → outputs merged → done.

### What's Not Done
- User-side of the marketplace (conscious tradeoff — prioritized evolution first)
- Tests are failing, unclear root cause

### Assessment (Timothy + Ved, post-call)

| Signal | Verdict |
|--------|---------|
| Code quality | Good |
| Understanding existing code | Good |
| Execution speed | Weak |
| Explanation quality | Too wordy, not crisp |

**Decision: Pass on Tush.** Explanations weren't bare bones. Execution signals not strong enough. Two more candidates in the pipeline to evaluate.

---

## Part 2: Product Debrief (Timothy + Ved)

### Decisions Made

**1. Deploy first, then drift detection**
Order is locked: SAL-36 (deploy Salaxia) → SAL-31 (drift from error) → SPO-85 (IDE highlighting). No shortcuts.

**2. Pre-visibility detection is the product niche**
Cursor and Codex auto-fix after an error appears. Sploink's differentiated position: catch the small behavioral signals *before* the error surfaces. Drift detection = pre-visibility bug detection. That's the pitch.

**3. Launch decision gate at SPO-85**
The first three issues (deploy, drift signal, IDE highlighting) are mandatory. Once SPO-85 ships, Timothy and Ved will evaluate difficulty of SAL-29 and make a go/no-go call: ship at that point or continue.

**4. SAL-29 is a prompt generator, not an agent router**
Clarified today. SAL-29 generates a targeted recovery prompt for the user (e.g., "break the loop — reframe the approach"). The user copies and pastes it into their agent. It's not autonomous delegation. Simpler than previously framed.

**5. IDE extension replaces desktop HUD as primary interface**
Major architecture shift. Instead of building SPO-49 (desktop HUD) as a separate app, the terminal dashboard UI gets embedded inside a VS Code-style extension. Advantages:
- Users don't tab-switch
- Extension can capture data CLI can't (user prompt text, which agent is selected, file edits, real-time agent responses)
- Installable on any IDE and OS
- CLI still runs underneath — extension is the surface, CLI is the engine

**6. Both CLI and extension are used together**
Not either/or. CLI captures system-level events (shell commands, git, test output). Extension captures IDE-level events (user prompt, agent identity, file interactions). They feed a unified signal.

**7. Prompt selection: choose first, generate later**
For intervention prompts: Phase 1 = surface the best prompt from an existing skill registry. Phase 2 = generate new prompts based on accumulated behavioral data. Don't try to generate before there's enough signal.

**8. Own IDE as long-term vision**
Extension is the near-term move. Owning a full IDE (where CLI is embedded inside) is the long-term horizon. Not on the immediate path.

### Core User Flow (locked)
```
User is coding in their IDE
→ Drift detected (thrash, novel error, retry escalation)
→ Extension highlights the risky file and lines
→ Tooltip shows: what signal fired + intervention prompt
→ User fixes it
→ Continues coding
→ Loop repeats
```
That's the product. Everything else is supporting infrastructure.

---

## Action Items

| Task | Owner | Due | Notes |
|------|-------|-----|-------|
| Let Tush know the decision | Timothy | Today | Pass — keep it brief |
| Create spec/issues for IDE extension architecture | Timothy | This week | Research needed first; run through Ved before starting |
| Meeting with Git Nexus guy (frontend candidate) | Timothy | Mar 19 | Let Ved know how it goes |
| Continue backend/full stack candidate pipeline | Timothy | Ongoing | 2 more in pipeline |
| Build IDE extension (VS Code-style, replaces HUD) | Ved | TBD pending spec | Blocked on Timothy's spec |
| Deploy Salaxia (SAL-36) | Ved | Overdue | Do first |
| Drift from error signal (SAL-31) | Ved | After SAL-36 | Blocks SPO-85 |
| Live IDE file highlighting (SPO-85) | Ved | After SAL-31 | Launch gate |

---

## Open Questions

- [ ] API cost model for extension — users call models through their own installed tools (Claude, Codex), so API costs stay on the user. Confirm this is the right framing before publishing the spec. — **Timothy**
- [ ] Is SPO-49 (desktop HUD) now fully replaced by the extension, or do both exist? — **Timothy + Ved** — resolve in spec
- [ ] What specific events can the extension capture that the CLI can't? Need a complete list before building. — **Ved** to enumerate

---

## Hiring Context

- **Tush:** Pass. Out.
- **Next two candidates:** Backend-oriented. That's the right profile — the scoring lane needs someone who can translate behavioral specs into weighted formulas, not an ML researcher.
- **Git Nexus guy (tomorrow):** Frontend. Could be relevant for extension UI.
- **Priority hire remains:** Backend/full stack for the scoring lane (SAL-28, SAL-31, SAL-32).

---

## Context for Future Reference

The IDE extension idea came from Ved during the debrief — Timothy hadn't thought of it. Insight: because users type their prompt *into* the extension, and the extension calls their already-installed agents (Claude, Codex), there are no additional API costs. Sploink just observes. This also means the extension can capture the user prompt, which the CLI wrapper can't currently access. This is architecturally cleaner than the HUD approach and removes the "install another app" friction at onboarding.

The framing shift that crystallized today: Cursor and Codex fix bugs after they appear. Sploink prevents bugs from becoming visible in the first place. That's the niche.
