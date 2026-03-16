# Meeting Notes: Positioning Agents — Divit Second Call

**Date:** March 16, 2026
**Attendees:** Timothy, Ved, Divit
**Type:** Candidate evaluation / recruiting
**Meeting Title:** Positioning Agents: Tracking Thrashing and Productivity with AI

---

## Summary

Strong second meeting. Divit verbally committed — "I would love to be a part of this as cofounder or tech support guy." Equity deferred. Wants to review codebase first and get back in a couple days with thoughts on the intervention model. Shared Gmail and GitHub. Being onboarded now.

---

## Decisions Made

1. **Divit is interested — trial period approach**
   - Why: He wants to see the code before formalizing equity
   - Equity range offered: 3-15% founding engineer, 20%+ cofounder
   - Next step: Share docs, Linear, repo. Build demo video after CLI wired end to end.

2. **Proof of concept is acceptable without replayability for pre-beta**
   - Why: No data yet anyway. POC is enough for launch.
   - If no frontend dev found in time, Ved can handle UI but it takes time from backend.

3. **Divit's role: intervention layer**
   - Confirmed he wants to work on it — "that was the right part which I was thinking"
   - Deep ML/fine-tuning background, interested in RL approach to intervention policy

---

## Key Insights from Divit

- **GTM question:** "How are we placing this in the market?" — came in prepared, thinking like a cofounder
- **Data normalization concern:** How do convergence/thrash scores generalize across ML projects vs full-stack? (Valid — different error types, different file change volumes). Timothy's answer: CLI + extension + plugin each capture different signal types, not just churn.
- **Agent modification question:** If agents come from third-party clouds, how do you actually modify them? (Right question — answered honestly: prompt injection for now, proprietary agents long-term)
- **VC feedback question:** Asked what VCs said — shows he's evaluating the bet, not just the problem
- **Self-learning signal:** Works on research paper outside internship, only free time is late at night. High intrinsic motivation.

---

## Timothy's GTM Answer (for reference)

- Short run: B2C power users (Claude Code, Cursor, Aider users) — need data first
- Hook: Repo as graph + heatmap overlay showing agent interaction over time
- Medium run: B2B — startups building better agents
- Long run: AODE (Agent Observability Developer Environment) — agents evolve through time, personal + product agents for founders
- Data model: 0-100 users = preset, 100-500 = deep learning model, 500-1000 = LLM
- Key insight shared: Bugs are created before they appear. Rewind/replay is the critical feature. Would only buy it if: (1) intervention works, or (2) UI predicts bugs before they happen.

---

## Action Items

| Task | Owner | Due |
|------|-------|-----|
| Share docs, Linear, repo with Divit | Timothy | Today |
| Wire CLI end to end | Timothy + Ved | Before Mar 18 |
| Record demo video for Divit | Timothy + Ved | Before Mar 18 |
| Onboard Divit to GitHub | Timothy | Today |
| Watch: does Divit engage with code unprompted in 48h? | Timothy | Mar 18 |

---

## Watch Signal

- Said "push equity back" — could be genuine or maintaining optionality (Aryan pattern)
- Has internship + research paper + other commitments — bandwidth real concern
- Timothy: "reminds me of Aryan — both interested in the problem, but once things get tough, are they able to put time in?"
- Key signal: unprompted codebase engagement in next 48 hours = real. Radio silence until follow-up = watch carefully.

---

## Post-Call (Timothy + Ved)

- Ved working on 3D graph visualization using Figma + open source 2D graph libs as reference
- Timothy looking for frontend dev for replayability UI — Vitali Avagyan in conversation
- If no frontend dev: proof of concept without replayability is acceptable for launch
