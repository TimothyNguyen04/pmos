# S³ — User Flow (Pre-Beta)

## How a user interacts with the system

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER                                     │
│                                                                 │
│   Opens Seoro (Mac app or web) and gives context/task          │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SEORO (Mac app / web)                        │
│                                                                 │
│  1. Captures context: screen, calendar, email, messages         │
│  2. Click triggers context capture                              │
│  3. Auto-generates a query from what the user is doing          │
│     e.g. "Draft follow-up to yesterday's investor call"         │
│                                                                 │
│  → Sends query to Salaxia via POST /intent                      │
└────────────────────────────┬────────────────────────────────────┘
                             │  POST /intent
                             │  { task_id, query, context }
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   SALAXIA (routing brain)                       │
│                                                                 │
│  1. Generates candidate intents with confidence scores          │
│     e.g. draft_email (0.82), schedule_meeting (0.63)            │
│  2. Applies threshold (0.7) → selects winning intent            │
│  3. Picks best agent: capability match + reputation score       │
│                                                                 │
│  → Sends task to Sploink via POST /execute                      │
└────────────────────────────┬────────────────────────────────────┘
                             │  POST /execute
                             │  { task_id, agent_id, intent, context }
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   SPLOINK (execution engine)                    │
│                                                                 │
│  1. Runs the selected agent                                     │
│  2. Captures workspace telemetry: file edits, commands,         │
│     build/test results, diffs, retries                          │
│  3. Detects convergence vs thrashing                            │
│                                                                 │
│  → Returns result + artifact to Seoro                           │
│  → Sends telemetry back to Salaxia via POST /telemetry          │
└──────────┬──────────────────────────────────────┬───────────────┘
           │ result / artifact                    │ POST /telemetry
           ▼                                      ▼
┌──────────────────────┐              ┌───────────────────────────┐
│  SEORO               │              │  SALAXIA (ranking update) │
│                      │              │                           │
│  Shows user:         │              │  Updates agent reputation │
│  - Email draft       │              │  High success → rank up   │
│  - Meeting invite    │              │  Errors/thrash → rank down│
│  - Summary           │              │  Better routing next time │
│  - Action items      │              └───────────────────────────┘
└──────────────────────┘
```

---

## Environment (Pre-Beta)

| Service  | Where it runs         | Notes                                      |
|----------|-----------------------|--------------------------------------------|
| Seoro    | Mac app (local)       | Ambient capture runs on user's Mac         |
| Seoro backend | GCP Cloud Run    | Intelligence, integrations, storage        |
| Salaxia  | Cloud (TBD)           | Routing + ranking engine                   |
| Sploink  | Cloud (TBD)           | Agent execution + telemetry                |

**For local testing:** all services can run locally on Mac before cloud deployment.

---

## The linking key

`task_id` flows through every service:
```
Seoro creates task_id
  → Salaxia receives task_id with /intent
    → Sploink receives task_id with /execute
      → Telemetry tagged with task_id sent back to Salaxia
```

This is how the system knows which task generated which telemetry and improves routing for that task type.
```
