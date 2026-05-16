---
name: Hold scope discipline on phase branches — don't bundle drafting-table polish into phase work
description: When implementing a planned roadmap phase, only the phase's own scope ships on that branch; unrelated polish/experiments go separately
type: feedback
originSessionId: f9dd3d64-aff3-445f-8e46-62b144385589
---
When implementing a roadmap phase (Phase 3 chunks, Phase 4B, etc.),
**only the phase's own scope ships on that branch**. Drafting-table
polish, UX experiments, "while I'm in there" tweaks, restyles, and tool
additions that aren't in the phase's locked plan belong on their own
branches with their own commits — or in the roadmap's deferred-quality-
concerns list to surface later.

**Why:** the 2026-05-11 Phase 4B investigation discovered the prior
session had bundled ~1,600 lines of legitimate wearer scaffolding with
a half-dozen unrelated drafting-table experiments (per-segment select
readout, guide L/θ readout, guide stroke restyle, snap toggle, WYSIWYG
preview cleanup, fit-to-viewport zoom, a brand-new text tool). One of
the bolted-on changes broke panel selection in a way that took an
investigation to isolate. Justin pushed back sharply: "What did UX in
the drafting table have anything to do with wearer profiles and
measurements?" The bundle violated the plan locked by /office-hours +
/plan-eng-review on 2026-05-09 and made the work undebuggable until it
was split apart.

**How to apply:** when starting work on a planned phase, re-read the
phase's locked plan (roadmap section + design doc in
`~/.gstack/projects/pattern-drafter-workspace/`). Hold to it. If during
implementation an unrelated improvement is observed:
- Tiny fix (≤5 lines, obvious, no risk to phase scope): inline OK, note
  in the commit body.
- Anything bigger: stash the phase work, ship the improvement as its
  own branch/commit, then come back. OR file as a roadmap deferred-
  quality-concerns line and move on.
- New tool or new entity (e.g., text annotations): always stop, surface
  to Justin, never silently expand phase scope.
Default to "this doesn't belong here" rather than "this is easy, I'll
just include it." Bundled-scope branches are very expensive to debug
when something goes wrong.
