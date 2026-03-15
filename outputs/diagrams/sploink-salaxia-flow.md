# Sploink + Salaxia — Full System Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         INPUTS                                  │
│                                                                 │
│   Semantic Data                    Telemetry Data               │
│   ─────────────                    ──────────────               │
│   • Written prompt                 • File edits                 │
│   • Task intent                    • Shell commands             │
│   • Goal framing                   • Test runs                  │
│   • Sub-goals (S-29)               • Git operations             │
│                                    • Build outputs              │
└────────────────┬────────────────────────┬───────────────────────┘
                 │                        │
                 ▼                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                      QUANTIFICATION                             │
│                                                                 │
│   System (what the agent IS doing)                              │
│   ─────────────────────────────────                             │
│   • Behavior signals (file churn, command patterns)             │
│   • Goal attribution — events mapped to sub-goals (S-29)        │
│   • Session progress over time                                  │
│                                                                 │
│   Environment (what SURROUNDS the agent)                        │
│   ──────────────────────────────────────                        │
│   • Model (Claude, GPT-4, Gemini)                               │
│   • Tools available (bash, browser, MCP)                        │
│   • Skills loaded                                               │
│   • Scaffolding and task framing (S-34)                         │
│   • Context window saturation                                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   GROUND TRUTH SIGNALS                          │
│                                                                 │
│   Convergence ──────────────────── Is the agent moving toward   │
│   (0.0 → 1.0)                      done? Git progress, test     │
│                                    pass rate, file churn        │
│                                                                 │
│   Divergence / Thrash ─────────── Is the agent stuck or going   │
│   (boolean + confidence)           backwards? Repeated edits,   │
│                                    same command failing, zero    │
│                                    git delta                     │
│                                                                 │
│   Error Drift ─────────────────── Is the agent moving away      │
│   (0.0 → 1.0)  ← Aryan (SEO-31)   from a working state? Error  │
│                                    rate trajectory, test        │
│                                    regression, novel errors     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                     Threshold crossed?
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  INTERVENTION ROUTING                           │
│                  (Salaxia — Aryan)                              │
│                                                                 │
│   S-33: Sploink emits signal → POST /intervention               │
│         ↓                                                       │
│   SEO-26: Receive behavioral signal                             │
│         ↓                                                       │
│   SEO-29: Signal → skill matcher                                │
│           "thrash on auth.py → narrow scope reprompt"           │
│         ↓                                                       │
│   SEO-30: Delivery layer → dashboard + outcome tracking         │
└──────────────────┬──────────────────────┬───────────────────────┘
                   │                      │
                   ▼                      ▼
┌──────────────────────────┐  ┌───────────────────────────────────┐
│  CHANGE THE SYSTEM       │  │  ADD TO THE SYSTEM                │
│  (modify current agent)  │  │  (inject new agent)               │
│                          │  │                                   │
│  • Narrow scope reprompt │  │  • Specialist agent mid-session   │
│  • Context injection     │  │    (S-58) — handles specific      │
│    (S-51) — nudge agent  │  │    blocker, merges back           │
│  • Swap skill/tool       │  │  • Parallel agents (S-54) —       │
│  • Adjust task framing   │  │    multiple trajectories on       │
│                          │  │    same step simultaneously       │
└──────────────────────────┘  └───────────────────────────────────┘
                   │                      │
                   └──────────┬───────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FEEDBACK LOOP                                │
│                                                                 │
│   Session ends → did convergence improve after intervention?    │
│         ↓                                                       │
│   Outcome recorded → skill effectiveness score updated          │
│         ↓                                                       │
│   Next intervention routes better (compounding data flywheel)   │
│         ↓                                                       │
│   Reputation graph (Salaxia) — agents ranked by real outcomes   │
└─────────────────────────────────────────────────────────────────┘
```

## Aryan's scope

| Issue | Layer | What it does |
|-------|-------|-------------|
| SEO-31 | Quantification | Compute error drift signal (ground truth #3) |
| SEO-26 | Intervention routing | POST /intervention endpoint — receive signal from Sploink |
| SEO-29 | Intervention routing | Match signal → right intervention skill |
| SEO-30 | Intervention routing | Deliver recommendation to dashboard, track outcome |

## What this unlocks

Once these four issues are done, the full loop closes:
measure → quantify → signal → route → intervene → measure again.
That's the first closed-loop agentic feedback system.
