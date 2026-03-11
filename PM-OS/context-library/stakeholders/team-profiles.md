# Team Profiles
*Last updated: 2026-03-11*

---

## Timothy Nguyen — Founder, System Architecture, Product Direction
- System architecture and design contracts
- Product strategy and narrative
- Evaluation metric definitions and benchmark design
- Agent scoring formulas and insight layer
- Overall product direction across Seoro, Salaxia, Sploink

---

## Imran Ullah — ML/AI Engineer (Sploink — Supply Layer)
**Core skills:**
- Agent runtime and execution environments (LangChain, LangGraph, CrewAI, custom architectures)
- Telemetry instrumentation and data pipelines
- Metrics and evaluation systems (success rate, token efficiency, retry rate, thrash detection, convergence signals)
- Benchmark engine design and execution
- ML modeling (ranking models, behavioral signals, agent momentum)
- Sandboxing and secure execution environments

**Owns:** Agent Intelligence Engine — turning black-box agents into measurable infrastructure

**Scope work for Imran as:** ML/data engineering tasks. He should not be handed frontend or DevOps work. Tasks should be framed around data schemas, model logic, signal computation, and execution pipelines.

---

## Ved Sharma — Full-Stack / Platform Engineer (Sploink — Product Layer)
**Core skills:**
- Backend infrastructure (APIs, databases, job queues, authentication)
- Frontend / dashboard UI (works with Ben on frontend)
- Containerized deployment and cloud infrastructure (Docker, autoscaling, cloud infra)
- Agent submission and versioning systems
- Template registry and platform architecture

**Owns:** Platform layer — the environment where agents are created, submitted, executed, surfaced, and compared

**Scope work for Ved as:** Full-stack platform tasks. Backend API design, database schema, job execution framework, and frontend dashboard. He collaborates with Ben (Benjamin Moon) on frontend for marketplace, agent builder, and agent visualizations.

---

## Aryan Pandit — Systems/Backend Engineer (Salaxia — Protocol Layer)
**Core skills:**
- Graph-based routing and orchestration systems
- Intent decomposition and candidate intent generation
- Agent ranking and evaluation layer (consumes telemetry → computes metrics → ranks agents)
- Multi-agent workflow orchestration
- Reputation graph systems
- Failure and bias detection

**Owns:** Routing and Protocol Layer — the brain that decides which agents handle what, and how they're orchestrated

**Scope work for Aryan as:** Systems/backend engineering with ML awareness. Graph logic, routing algorithms, ranking systems, capability mapping. Schema alignment with Imran (telemetry compatibility) is a recurring collaboration point.

---

## Benjamin Moon — Co-founder / Frontend (Seoro)
**Collaborates with:** Ved on frontend for marketplace, agent builder, and agent visualizations

---

## Collaboration Patterns
- **Imran ↔ Aryan:** Schema alignment — telemetry schema must be compatible with Salaxia's routing and reputation systems
- **Ved ↔ Ben:** Frontend work — marketplace UI, agent builder, agent visualizations
- **Timothy ↔ All:** Defines contracts on Day 1 (telemetry schema, agent card schema, benchmark tasks, scoring formula) that unblock everyone else
