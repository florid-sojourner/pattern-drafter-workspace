---
name: measurements-are-optional-convenience
description: "Wearers + measurements are an optional drafter convenience, never a drafting prerequisite; competitor is the slip of paper, not other apps"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8a95c1c5-178a-4f71-9e77-e15ea3c9cb13
---

Body measurements (the Phase 4B "Wearers" feature) are an **optional convenience for the drafter**, not a prerequisite for drafting. A user can draft any pattern with measurements held in their head or on a slip of paper — wearers exist because the digital notebook is *better than paper* for someone who loses paper, not because drafting needs them.

Very-secondary future motivation: prep for parametric block generation if we ever introduce it. Today's primary motivation: Justin (organizationally challenged with paper and handwriting) wanted a digital home for his son's pants project measurements.

**Why:** Justin corrected this directly on 2026-05-13 after the design review proposed bouncing first-time users from Drafting → Measurements as if measurements were a required setup step.

**How to apply:**
- The new-pattern flow must never gate on a wearer being chosen. "No one in particular" is a first-class, default answer — not a "skip" or "later."
- Copy describing the Measurements feature should position it as a notebook ("body measurements you'd otherwise scribble on paper"), not as a drafting setup step.
- The `pattern → wearer` relationship is *metadata on the pattern*, never a parent-child URL relationship. See [[hub-is-a-fork-not-a-workspace]].
- If parametric block generation is ever revisited, treat it as a separate decision — it doesn't justify upgrading the wearer-on-pattern relationship from optional metadata today.
