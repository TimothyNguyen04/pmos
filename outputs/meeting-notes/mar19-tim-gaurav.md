# Meeting Notes: Tim and Gaurav

**Date:** March 19, 2026
**Attendees:** Timothy (founder), Gaurav (Gauravchau2412)
**Type:** Candidate intro call
**Duration:** ~20 min

---

## Summary

Strong intro call. Gaurav is a 2nd year CS student at SCPU (major IT hub in India), focused on AI agent memory management and context engineering. He came in curious about knowledge graphs, independently raised temporal knowledge graphs as relevant, and drew a page indexing analogy that showed real structural understanding. Committed to 3 hrs/day, founding engineer track. Test task pending.

---

## Candidate Assessment

**Strengths:**
- Has worked with knowledge graph representations before -- not just theory. Knows entity-relation extraction, node/edge structure, traversal complexity in multi-dimensional graphs.
- Unprompted: mentioned temporal knowledge graphs as important for what we're building. He's already thinking in the right direction.
- Page indexing analogy was sharp. He mapped the hierarchical KG to search engine page graphs on his own -- shows he understands the structural logic, not just the label.
- Pragmatic instinct: suggested .md files might be more efficient than a full graph for some knowledge representation use cases depending on latency constraints. That's the right kind of pushback -- not resistant, just calibrated.
- Context engineering focus maps directly to the graph layer work. He's not a generalist who stumbled in -- this is his area.
- Energy was genuine. "I would love to work on this." No hesitation when asked.

**Concerns:**
- 3 hrs/day ceiling. Said "max I can go is three hours per day" -- founding engineer, not cofounder bandwidth.
- India time zone -- async coordination will matter.
- Came in thinking we were building self-evolving agents (AutoGPT-style). Had to reframe him. Not a bad sign -- it means he was thinking bigger -- but he'll need the spec to calibrate correctly.
- Depth on hierarchical KGs specifically is untested. He's worked with flat/temporal graphs but said "now that's the problem" when the traversal complexity came up. Worth probing in the task.

**Role fit:** Knowledge graph layer -- extracting entities from natural language prompts, building the upper layers (Layer 1-3) of the 4-layer graph, memory management for agent context. Directly complementary to Param's work on Layers 4/scoring.

---

## Key Technical Exchange

**Gaurav on graph structure:**
> "For knowledge graphs, we have to prepare an extractor. The user gives natural language prompt, we extract main entity-relation pairs. In hierarchical, you don't do it in one dimension. You do it in two dimensions. The reasoning gets complex. The traversal gets complex."

This is accurate and he said it without prompting. He knows what he's getting into.

**Gaurav on page indexing:** Drew analogy between hierarchical KG and page indexing (search engine graph of linked pages). Correct structural intuition -- nodes/edges with hierarchical relationship traversal maps cleanly to how page indexes work.

---

## Next Steps

| Task | Owner | Due Date | Priority |
|------|-------|----------|----------|
| Send test task to Gaurav | Timothy | Today | High |
| Add Gaurav to WhatsApp | Timothy | Done ✅ | -- |
| Onboard if task clears | Timothy | Post-task | -- |

---

## Comp / Commitment

- **Track:** Founding engineer (3 hrs/day = ~21 hrs/week)
- **Comp:** Equity only, pre-funding
- **Gaurav's take:** "It's completely fine." No hesitation on equity-only.
- **His framing:** Okay with non-equal equity split given the hours constraint. Mature read on the situation.

---

## Open Questions

- [ ] What test task to send? Should be KG-focused -- entity extraction, graph construction, or layer population. Probe the depth he claimed.
- [ ] How does Gaurav's work integrate with Param's? Param owns graph scoring/risk (SPO-85/86/98). Gaurav would likely own upper layer population (Layers 1-3, entity extraction from PRD/README/prompts).

---

## Context

Referred via YC Cofounder Matching or similar (interests: knowledge graphs, RAG, multi-agent systems). SCPU is a major tech university in India. Second year, so likely 2027-2028 graduation. Engaged and available now.

**Timothy's read:** "DAMN! i got another good person."
