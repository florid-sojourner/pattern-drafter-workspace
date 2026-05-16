---
name: Run discovery skills before writing plan artifacts
description: When a user asks for a structured rebuild/planning exercise with gstack skills, run the skills first — don't pre-populate plan/roadmap files with assumptions
type: feedback
originSessionId: c1db4391-0f33-439e-8d91-bba0ebf5c54f
---
When the user asks me to kick off a planning / rebuild exercise using gstack skills (office-hours, plan-ceo-review, plan-eng-review, etc.) and ALSO asks me to organize output in specific markdown files, **run the skills first** — don't write the plan artifacts ahead of them.

**Why:** On 2026-04-23 during the pattern-drafter rebuild kickoff, I started writing `roadmap.md` with placeholder sections *before* launching the office-hours interview. Justin interrupted: "why didn't you start any g skills agents to start an interview like office-hours before proposing a roadmap md?" The files should be filled in *from* what the skills surface, not as scaffolding I guess at up front.

**How to apply:**
- If the user asks for skills-driven planning, launch the skills FIRST (office-hours, /plan-ceo-review, codebase Explore agent, etc.).
- Only write organizing files (roadmap, wiki, rules, design docs) after the skills have produced real content to pour in.
- It's OK to create empty directories or touch empty placeholder files, but don't fill sections with my own guesses about "phases" or "open questions" before the interview runs.
- This applies even when the user has explicitly asked for those files — they meant "organize the output there," not "pre-fill with your assumptions."
