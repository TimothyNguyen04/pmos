# GTM Strategy: Sploink — Power User Launch
**DRI:** Timothy Nguyen
**Status:** Active — March 2026
**Target:** First 50 active users, first 200 installs

---

## The Opportunity

Agent session lengths are getting longer. Claude Code, Cursor, and Devin are all pushing toward longer autonomous runs. A 2-minute session saves 5 minutes. A 2-hour session saves a day. The market is actively trying to extend session length — but the infrastructure to make long sessions manageable doesn't exist.

Sploink is the first cockpit for long-running agent sessions. Not observability. Active session management. Human-on-the-loop.

---

## ICP — Exactly

**Who:** Power users running Claude Code, Cursor, or custom agent frameworks for 20+ minute sessions daily.

**Pain:** Has had a session "go in circles." Either killed it or waited it out. Burned $5-10 on tokens that produced nothing. No visibility into whether the session was making progress or wasting time.

**Context:** Pays for Claude Pro or Cursor Pro — already converted on AI coding. Lives in the terminal. Cares about token cost. Technically sophisticated enough to understand convergence as a concept.

**Language they use:** "spinning," "going in circles," "getting stuck," "burning tokens." Not "thrashing."

---

## The One-Liner

> "See what your agent is actually doing. Intervene before it wastes an hour."

---

## The Aha Moment

User installs sploink. Runs their first session. Sees the convergence curve catch a thrash they would have missed. Saves time and money in the first 15 minutes.

Everything before the aha moment is acquisition. Everything after is retention.

---

## Entry Point

```bash
pip install sploink
sploink run -- claude "build the auth module"
```

Zero config. Zero behavior change. Wraps any agent command. Side panel opens automatically.

---

## Acquisition Channels

| Channel | Approach | Target |
|---|---|---|
| **Twitter/X demo video** | Show a real 14-minute session. Curve climbs, flatlines at 7min, intervention fires, recovery. Visceral and shareable. | 30 installs |
| **Show HN** | "Show HN: I built a real-time cockpit for long-running agent sessions" | 20 installs |
| **Discord seeding** | Claude Code Discord + Cursor Discord. Find threads where people complain about agents spinning. Drop naturally. | 15 installs |
| **Direct DM outreach** | 20 power users on Twitter publicly frustrated with agent sessions. Personal DM, offer early access. | 10 installs |
| **GitHub demo repo** | Sample session recording showing thrash detection. Gets discovered organically. | 5 installs |

**Total target:** 200 installs, 50 active sessions (ran at least one real session through sploink)

---

## Trust Mechanism

Terminal users are skeptical of new tools. Trust is earned by:

1. **Open source capture layer** — they can read exactly what's being collected
2. **Local-first** — telemetry stored on device by default, opt-in to cloud sync
3. **No account required for v1** — zero friction, zero commitment
4. **README transparency** — explicit list of what sploink captures and what it doesn't

---

## 30-Day Plan

| Week | Focus | Target |
|---|---|---|
| **Week 1** | Get demo working on a real codebase. Record the cockpit catching a real thrash. | Demo video ready |
| **Week 2** | Twitter post + Show HN | 200 installs |
| **Week 3** | Discord seeding + direct DM outreach | 50 active sessions |
| **Week 4** | Feedback loop. Where is the aha moment? Where is drop-off? What intervention is used most? | Qualitative insight from 10 users |

---

## Positioning Against Competitors

| Competitor | What they do | What we do differently |
|---|---|---|
| LangSmith | Traces LLM calls (API-level) | System-level behavioral telemetry — execution behavior, not call logs |
| AgentOps | Similar to LangSmith | Same gap — call tracing, not convergence/thrash detection |
| Cursor/Claude Code | Agent IDEs | Passive — show you output. We're active — cockpit + intervention palette |

**Key message:** You wouldn't run a database without monitoring. You're running agents blind.

---

## Success Metrics

- **Week 2:** 200 installs
- **Week 3:** 50 active sessions (at least one real sploink-wrapped session)
- **Week 4:** 10 users give qualitative feedback on the aha moment
- **Month 2:** 5 users run sploink on every session (habit formed)

---

## Open Questions

- [ ] What is the exact install command? pip vs npm vs brew?
- [ ] Local-first storage format — where does telemetry live on device?
- [ ] Demo session: which codebase, which agent, which task? Needs to be reproducible.
- [ ] Twitter outreach list: who are the 20 target power users?
- [ ] Show HN timing: what day/time maximizes visibility?
