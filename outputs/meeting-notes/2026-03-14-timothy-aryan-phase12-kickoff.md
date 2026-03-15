# Meeting Notes: Phase 1.2 Kickoff — Timothy + Aryan

**Date:** March 14, 2026
**Attendees:** Timothy Nguyen, Aryan Pandit
**Type:** Engineering sync / task assignment

---

## Summary

Timothy walked Aryan through Phase 1.2 scope: error drift as the third quantification signal, low-level Salaxia routing, and the human-on-the-loop paradigm. Aryan aligned on the vision, proposed building the error monitoring layer in Python as a standalone API (not converting all of Salaxia), and pushed back on the terminal UI in favor of a plugin + website approach.

---

## Decisions Made

1. **Error drift is Aryan's primary task**
   - Build in Python, expose as API endpoint
   - Log errors when they occur, attach to existing system
   - Language doesn't matter — "it's an API call"

2. **Don't convert all of Salaxia to Python**
   - Aryan's framing: "These things are very loosely coupled. Just build the parts in Python that need to be Python, and expose an API."
   - Validates the contract approach over full rebuild (S-63 deferred)

3. **Plugin over custom terminal**
   - Aryan: "If you provide your own terminal, no one gonna use it. Provide a plugin."
   - Claude Code, Gemini CLI, VS Code all have extension/plugin marketplaces
   - Plugin attaches, reads logs with user permission — no custom terminal needed

4. **Website for logs and dashboard**
   - User installs plugin → coding happens in their terminal → they go to website to see logs, errors, reports
   - Long run: agents callable from the website
   - Timothy owns: landing page + dashboard

---

## Action Items

| Task | Owner | Due | Notes |
|------|-------|-----|-------|
| Build error drift module in Python, expose as API | Aryan | This week | SEO-31 — log errors, track drift, expose endpoint |
| Low-level routing system (if bandwidth allows) | Aryan | This week | Split with Dhrit if possible |
| Set up Linear MCP in Claude Code | Aryan | Today | Timothy recommended it for context from Linear |
| Landing page | Timothy | Mar 17 | S-40 |
| Dashboard (web UI) | Timothy | Mar 19 | S-41 |

---

## Key Insights

**Aryan on the plugin approach:**
> "We don't need to build a plugin terminal from scratch. If we provide a plugin, everyone gonna use it. It will be much easier and convenient for the user."

This aligns with S-50 (MCP server already done) — Sploink as a plugin/MCP that attaches to any coding environment, not a custom terminal.

**Aryan on Python conversion:**
> "We don't need to do from the ground up. These things are very loosely coupled. In the end, it's an API call — it doesn't matter."

Validates contract-first approach. Build error drift in Python, expose endpoint, wire to existing TypeScript Salaxia via API.

**Aryan on error logging:**
> "Once the error comes, IDEs already try to resolve it automatically. What we can do is log those errors so the user knows, and in the long run use those to provide a different path."

Good framing — Phase 1 is read-only, intervention is automatic later.

**Timothy's marketplace insight (end of meeting):**
> "As more people use our plugin, we get more data, and we use that data to ultimately create agents that go in the marketplace."
Flywheel: plugin usage → flight recorder data → agent training → marketplace agents → better interventions.

---

## Open Questions

- [ ] Can Aryan split error drift + low-level routing with Dhrit? — Aryan to confirm
- [ ] What specific error types to log in Phase 1? (compile errors, test failures, lint?) — Aryan to define
- [ ] Which plugin marketplace to target first? (VS Code, Claude Code MCP, Gemini CLI?) — Timothy to decide

---

## Architecture Clarification (from this meeting)

Aryan confirmed the loosely-coupled API approach. The system is:

```
Terminal (any) → Sploink plugin captures logs/errors
    ↓
Python error drift module (Aryan) → exposes API
    ↓
Salaxia receives signal via API (language-agnostic)
    ↓
Website dashboard shows user what happened
```

Not a custom terminal. Not a full Salaxia rewrite. Plugin + API + website.
