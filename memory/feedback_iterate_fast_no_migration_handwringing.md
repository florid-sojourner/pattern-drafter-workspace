---
name: Iterate fast — don't moralize about migration timing
description: When proposing schema/data-shape changes early in a phase, don't frame "do it now vs. later" as a migration-cost lecture. Just do the work.
type: feedback
originSessionId: f78d9209-6eee-4a21-9be5-e1fd5ad90bc5
---
When a phase is mid-implementation and zero production data exists yet, do not preface schema or canonical-key changes with reasoning like "the window to add it cheaply is now, before docs accumulate" or "otherwise you'll have a migration to backfill." That framing assumes a slow-moving production system. This project iterates fast — there is no production cohort whose data needs preserving across the rebuild's early phases.

**Why:** Justin pushed back on a paragraph about adding `thigh` "during 4B.2 not after," with the migration-cost rationale. His response: "That's a weird way to talk. Didn't we just finish 'day 1'? We're iterating super fast. Just do what I ask and worry less about time."

**How to apply:**
- When asked to add a canonical key, add it. Don't editorialize about timing or migration risk during early phases.
- Reserve migration-cost reasoning for genuinely production-deployed systems with real users' data at risk.
- "Just do what I ask" applies broadly — when Justin asks for a concrete change, execute it; don't relitigate sequencing.
