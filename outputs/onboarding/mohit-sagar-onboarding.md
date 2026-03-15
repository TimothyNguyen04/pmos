# Welcome to S³ — Mohit Sagar Onboarding
*March 2026 | Prepared by Timothy Nguyen*

---

## 1. The Vision

S³ is building the infrastructure layer for the agentic internet.

Think about what AWS did for cloud — it didn't build apps, it built the foundation that every app runs on. We're doing the same thing for agents. Right now, agents are disconnected, unranked, and unaccountable. Nobody knows which agents are actually good, nobody measures whether they're making real progress, and there's no intelligent layer that connects what a user needs to the agents that can deliver it.

We're building that layer. Three products, one system:

**Seoro** — the perception layer. Captures ambient context from a user's life: meetings, emails, messages, calendar. Converts that raw data into structured intent. Think of it as the eyes and ears.

**Salaxia** — the routing brain. Takes the intent Seoro generates and decides which agent should handle it, based on capability matching and reputation scores built from real performance data. Gets smarter with every execution.

**Sploink** — the execution and measurement layer. Runs the agents, captures low-level telemetry on what they actually do (file edits, commands, build results, retries), and feeds that data back to Salaxia so routing improves over time.

The linking key across all three: `task_id`. Every user action creates a task_id that flows from Seoro → Salaxia → Sploink → back to Salaxia as telemetry. That loop is the product. The more it runs, the smarter it gets.

**Pre-beta launch: Sunday March 15.** You're joining at the most critical moment.

---

## 2. The Team

| Person | Role | What they own |
|---|---|---|
| **Timothy Nguyen** | Founder, DRI | Architecture, product direction, PM. Come to him with blockers and questions. |
| **Benjamin Moon (Ben)** | Co-founder | Seoro Mac app and frontend. He owns all Swift/iOS and Seoro product work. |
| **Aryan Pandit** | Systems/Backend | Salaxia — routing engine, intent scoring, reputation graph. |
| **Imran Ullah** | ML/AI Engineer | Sploink — runtime telemetry, CLI wrapper, agent evaluation. |
| **Ved Sharma** | Full-Stack Platform | APIs, database, infrastructure. Backend is complete. Currently overloaded — your role directly relieves him. |

**One rule:** Don't create Seoro issues or assign tasks to Ben. He manages his own board.

---

## 3. The Sprint

**Goal:** End-to-end working loop by Sunday March 15.

That means: a user gives Seoro a task → Salaxia routes it to an agent → Sploink executes it and captures telemetry → telemetry feeds back to Salaxia → ranking improves.

Every issue on Linear is wired with blocking dependencies so the team knows what order to build in. Your work is on the critical path — database schema and auth are the foundation everything else depends on.

**Key dates:**
- Mar 12 (Thursday): Auth, DB migrations, user isolation done
- Mar 13 (Friday): API contracts finalized, services able to talk
- Mar 14 (Saturday): Wiring complete, integration test running
- Mar 15 (Sunday): Pre-beta launch

---

## 4. Your Objectives

You're taking four issues off Ved so he can focus on the cross-service API contracts that block the entire team. These are your priorities in order:

### Priority 1 — SEO-41: User Authentication and Multi-Tenancy
**Due: March 12**

No real users without this. The auth module exists (`auth/` directory with JWT validation, signup, login) but needs end-to-end verification and `user_id` enforcement on every endpoint.

What you're building:
- Verify Supabase Auth flow end-to-end from the Mac app through the backend
- Ensure every intelligence endpoint requires a valid JWT
- Add `user_id` to every table that's missing it
- Row-Level Security (RLS) on all tables so users only see their own data
- Session management: token refresh, logout, session expiry

Done when: a user can sign up, log in, and every API call is scoped to their data only.

### Priority 2 — SEO-45: Database Migrations
**Due: March 12**

Every agent and artifact feature is blocked until these tables exist. Foundation work.

Tables to create:
- `agents` — id, user_id, name, description, type, status, config (JSONB), capabilities (array), created_at
- `artifacts` — id, user_id, agent_id, title, content, artifact_type, metadata (JSONB), version, created_at
- `adaptation_events` — id, user_id, agent_id, event_type, before_state, after_state, trigger, created_at
- `agent_insights` — id, agent_id, user_id, insight_type, content, confidence, created_at

All tables need RLS enabled with `user_id` policies. Write the migrations as repeatable SQL so they can be re-run cleanly.

Done when: all tables exist, RLS is on, and a test insert + select confirms isolation works.

### Priority 3 — SEO-57: Fix user_id Isolation on Integration Tables
**Due: March 12**

Four existing tables are broken for multi-tenancy:
- `email_messages` — uses `person_id` instead of `user_id`
- `calendar_events` — no `user_id` column
- `notion_pages` — no `user_id` column
- `slack_messages` — `user_id` stores Slack's platform ID, not Seoro's user ID

For each: add a proper `user_id` column (UUID, references auth.users), backfill where possible, enable RLS policy.

Done when: all four tables have proper user_id isolation and RLS enforced.

### Priority 4 — SPL-25: Append-Only Event Storage
**Due: March 14**

Coordinate with Imran on this one. He's emitting workspace telemetry events from the CLI wrapper. You're building the storage layer.

Requirements:
- Append-only event stream — never mutate past events
- Queryable by: `session_id`, `task_id`, `agent_id`
- Sort by `sequence_index` (primary) + `timestamp` (secondary)
- Must support reconstructing full event timeline for a session

Coordinate with Imran on:
- Ingestion endpoint format (what does the POST body look like?)
- Event volume expectations
- Confirm end-to-end: Imran emits → your endpoint receives → stored and queryable

Done when: events emitted by Imran's CLI wrapper are stored, queryable in order, and filterable by session/task/agent.

---

## 5. Where to Find Everything

- **Linear** — source of truth for all tasks. Your issues: SEO-41, SEO-45, SEO-57, SPL-25
- **GitHub** — `github.com/S-3-ai/Seoro` — the main repo
- **This doc** — architecture reference for the overall system
- **Questions** — message Timothy directly. He has a 20-page architecture doc and answers questions fast.

---

## 6. How We Work

- Linear is the source of truth. If it's not in Linear, it doesn't exist.
- Every issue has a description with ## Context / ## What to build / ## Success criteria. Read those before starting.
- Post progress updates as comments on your issues. Keeps the team unblocked.
- If you're stuck for more than an hour, message Timothy or Ved. Don't spin.
- We ship fast. Perfect is the enemy of Sunday.

---

## 7. The Bigger Picture

You're joining as a founding engineer. That means once we get funding and start hiring, you'll lead the engineers below you. The work you do this week ships to real users Sunday and directly feeds into the telemetry loop that makes the whole system smarter.

The problem you're solving — invisible agent behavior, agents that thrash instead of converge, routing that improves from real performance data — nobody has solved this cleanly yet. The infrastructure you're building is the foundation of that solution.

Welcome to the team. Let's ship.

---

*Questions? Message Timothy on WhatsApp.*
