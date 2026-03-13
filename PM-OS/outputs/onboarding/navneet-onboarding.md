# Navneet — Seoro Backend Onboarding
*S³ | March 2026*

---

## The Big Picture

S³ is a three-layer agentic stack. Think of it as three companies working together:

```
SEORO (you work here)
"What does the user need right now?"
Captures context — emails, meetings, screen, messages.
Understands the user deeply. Sends intent to Salaxia.
        ↓
SALAXIA
"Which agent handles this?"
Routes intent to the best AI agent based on capability + past performance.
        ↓
SPLOINK
"Execute and measure."
Runs the agent. Captures exactly what it did. Sends telemetry back to Salaxia
so routing gets smarter over time.
```

**Your job is entirely inside Seoro.** Seoro is the perception layer — it knows the user, their relationships, their calendar, their emails, their patterns. When a user asks a question, Seoro figures out what context is relevant and surfaces it intelligently. That's what you're building.

You don't work on the iOS/Mac app (that's Ben, requires a Mac). You work on the Python backend that powers it.

---

## Your Repo

```
git clone git@github.com:S-3-ai/seoro.git
cd seoro
```

Ask Timothy for SSH access if you don't have it yet.

### Codebase Map (what matters for you)

```
seoro/
├── intelligence/           ← YOUR HOME. Everything you build lives here.
│   ├── api/                ← FastAPI endpoints (HTTP layer)
│   │   ├── conversations.py       ← Chat interface endpoints
│   │   ├── notifications.py       ← Nudge/notification endpoints
│   │   └── ...
│   ├── core/               ← Shared utilities and config
│   │   ├── intent_classifier.py   ← Classifies query types
│   │   ├── claude_client.py       ← Claude API wrapper (use this for LLM calls)
│   │   └── config.py              ← All env vars and settings
│   ├── search/             ← Search and retrieval
│   │   ├── retrieval.py           ← Main retrieval logic
│   │   └── query_classifier.py    ← Routes queries to retrieval strategies
│   ├── processing/         ← Background processing pipelines
│   │   ├── pattern_detector.py    ← Detects behavioral patterns
│   │   └── memory_tiers.py        ← Memory tier logic (reference this)
│   ├── scheduling/         ← Background jobs
│   │   └── sync_scheduler.py      ← APScheduler jobs (add nudge job here)
│   └── ingestion/          ← Data ingestion pipeline
│       └── pipeline.py            ← Post-ingestion hooks (add memory gen here)
├── database/               ← Data layer
│   └── operations.py              ← All DB reads/writes (use these functions)
└── auth/                   ← Authentication (don't touch, just use middleware)
```

### Local Setup

```bash
# Python 3.11+
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Copy env file (Timothy will share the values)
cp .env.example .env

# Run the backend locally
uvicorn intelligence.server:app --reload

# You don't need the Mac app running. Test your endpoints directly via curl or Postman.
```

---

## Your Tasks — In Order

Do them in this exact sequence. SEO-15 is due first.

---

### Task 1: SEO-15 — Context Routing
**Due: Mar 12 | Priority: Urgent**
Linear: https://linear.app/seoro/issue/SEO-15

**What it is in plain English:**
Right now when a user sends a chat message, the backend doesn't know what kind of context to pull. "What did Sarah say last week?" needs to search emails and meetings. "What should I focus on today?" needs to pull calendar and promises. "What's our Q2 strategy?" needs to search documents. You're building the router that figures out which retrieval strategy to use based on what the user is asking.

**Where to start:**
Read `intelligence/search/query_classifier.py` first. It already classifies queries as NAVIGATIONAL, EXPLORATORY, or DEFAULT to adjust search weights. You're extending this concept — instead of just adjusting weights, you're routing to different retrieval sources entirely.

Then read `intelligence/api/conversations.py` to understand how chat messages flow. The message comes in via `POST /api/conversations/{uuid}/messages`. That's where you wire in the context router.

**What to build:**

1. Extend `intelligence/search/query_classifier.py` to add a new classifier that identifies query type:
   - **People query** — "What does Sarah want from me?", "What did John say?" → pull PersonContext + meetings + emails for that person
   - **Topic query** — "What's our Q2 strategy?", "What did we decide about pricing?" → topic memories + related documents + meetings
   - **Action query** — "What should I do today?", "What am I supposed to send?" → calendar + promises + nudges
   - **General** — everything else → semantic search across all sources

2. In `intelligence/search/retrieval.py`, add a `route_context(query, user_id)` function that takes a classified query type and assembles the right context. Use the existing retrieval functions already in that file — you're just adding routing logic on top.

3. Wire it into `intelligence/api/conversations.py` — when a message comes in, classify it, route it, and include the assembled context in the Claude prompt.

**Success check:**
- Ask "What did Sarah say?" → response references real email/meeting data about Sarah
- Ask "What should I focus on today?" → response references calendar and pending promises
- Generic question → falls back to semantic search

---

### Task 2: SEO-13 — Automatic Memory Generation Pipeline
**Due: Mar 13 | Priority: High**
Linear: https://linear.app/seoro/issue/SEO-13

**Dependency:** Check with Ben that SEO-12 (Memory View data model) is done before you start building. If it's not, work on SEO-5 in parallel.

**What it is in plain English:**
When new data comes in (an email thread, a meeting transcript), Seoro shouldn't just dump it in the database raw. It should synthesize it into a memory — a human-readable summary tied to an entity (a person, a topic). If a memory about Sarah already exists, update it with what's new. If it doesn't, create one. Over time, Sarah's memory gets richer without the user doing anything.

**Where to start:**
Read `intelligence/processing/memory_tiers.py` to understand how memories are currently structured. Read `intelligence/ingestion/pipeline.py` to see how data flows after ingestion — that's where you'll hook in.

**What to build:**

1. Create `intelligence/processing/memory_generator.py` — new file:
```python
class MemoryGenerator:
    def generate_or_update(self, entity_id, entity_type, new_data, user_id):
        # 1. Check if memory exists for this entity in DB
        # 2. If yes: fetch existing memory + new data, call Claude to synthesize updated summary
        # 3. If no: call Claude to create new memory from new data
        # 4. Write back to DB with updated timestamp and strength score
```

2. In `intelligence/ingestion/pipeline.py`, after ingestion completes, call `MemoryGenerator.generate_or_update()` for each entity identified in the new data.

3. In `database/operations.py`, add memory CRUD functions if they don't exist: `create_memory()`, `get_memory_by_entity()`, `update_memory()`.

4. Use `intelligence/core/claude_client.py` for all summarization calls — it's already set up, don't create a new Claude client.

**Success check:**
- New email from Sarah comes in → Sarah's memory auto-updates without any user action
- Ingest 5 emails from Sarah → one memory, not five separate entries
- Memory gets richer with each new email, not just longer

---

### Task 3: SEO-5 — Proactive Nudge Engine
**Due: Mar 13 | Priority: High**
Linear: https://linear.app/seoro/issue/SEO-5

**What it is in plain English:**
Instead of waiting for the user to ask something, Seoro proactively surfaces suggestions. "You have a meeting with Sarah in 30 minutes — here's what you discussed last time." "You promised to send John that doc three days ago and haven't." You're building the background job that checks conditions and generates these nudges, and the endpoint that serves them to the app.

**Where to start:**
Read `intelligence/scheduling/sync_scheduler.py`. It uses APScheduler to run background jobs (Gmail sync, Calendar sync, etc.) on intervals. You're adding a nudge evaluation job to this same scheduler. The pattern is already there — follow it.

Read `intelligence/processing/pattern_detector.py`. It already detects recurring behavioral patterns per person. The nudge engine will use these patterns as one of its trigger inputs.

Read `intelligence/api/notifications.py` to see what's already there for the notifications endpoint.

**What to build:**

1. In `intelligence/scheduling/sync_scheduler.py`, add a new scheduled job `evaluate_nudges()` that runs every 15-30 minutes per user:
   - Check for upcoming calendar events in the next 60 minutes → if attendee context exists, generate meeting prep nudge
   - Check `database/operations.py` for unresolved promises → if past due, generate reminder nudge
   - Check `pattern_detector.py` for relationship decay alerts → generate relationship ping nudge

2. In `database/operations.py`, add:
   - `create_nudge(user_id, nudge_type, content, context)`
   - `get_active_nudges(user_id)` — returns nudges not yet dismissed
   - `dismiss_nudge(nudge_id, user_id)`

3. In `intelligence/api/notifications.py`, add `GET /v1/nudges` endpoint that returns active nudges for the current user, and `POST /v1/nudges/{id}/dismiss` to dismiss one.

**Success check:**
- Nudges appear before meetings, not after
- Dismissed nudges don't come back
- Max 3-5 nudges per day (don't spam)
- Each nudge references real data ("your meeting with Sarah" not "an upcoming meeting")

---

## General Rules

**Making Claude API calls:** Always use `intelligence/core/claude_client.py`. Don't import the Anthropic library directly.

**Database reads/writes:** Always use `database/operations.py`. Don't write raw SQL.

**Auth:** Your endpoints need `require_auth` middleware. Look at how other endpoints in `intelligence/api/` do it — copy that pattern.

**Testing:** Test your endpoints locally with `uvicorn intelligence.server:app --reload` and hit them via curl or Postman. You don't need the iOS app.

**No Mac required:** Your work is 100% Python backend. The SwiftUI iOS app is Ben's domain. You'll never need to touch it.

**Asking for help:** Message Timothy on WhatsApp anytime. If a file you need to touch doesn't exist yet or a dependency (like SEO-12) isn't done, flag it immediately — don't wait.

---

## Where to Start Right Now

1. Get repo access from Timothy
2. Clone the repo, set up your Python environment
3. Read `intelligence/api/conversations.py` and `intelligence/search/query_classifier.py`
4. Start building SEO-15 — it's due tomorrow

Good luck.
