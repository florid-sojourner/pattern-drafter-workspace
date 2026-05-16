---
name: Sketch drawing features that reached maturity (rebuild port list)
description: The specific drawing behaviors in pattern-drafter-sketch that Justin confirmed were refined and worth porting to the rebuild. Other sketch UI (toolbar, etc.) was not yet refined.
type: project
originSessionId: bddb0d38-cbf2-484a-a96f-f8c72abf9e70
---
Refined drawing behaviors in `pattern-drafter-sketch/src/` that Justin confirmed were reaching maturity (2026-04-24) and should inform the rebuild:

- **Points / pen tool** — for drawing shapes with straight sides. Includes close-by-snap-to-an-existing-point which produces a Panel (closed path). This is how you finish a shape.
- **Curves via drag-handles** — curves are accessed by dragging little tags / handles on existing segments (NOT a separate curve tool). Justin called this "quite excellent."
- **Darts** — fairly refined as a domain primitive.
- **Cut line vs sew line display** — toggleable (setting / context menu / similar). Seam-allowance visualization.

Not yet refined in the sketch — rebuild is free to design these fresh:

- Toolbar / tool-switching UX.
- Any drawing paradigm not in the list above.

**Why:** Phase 2C of the rebuild diverged from the refined behaviors because I wrote new tools without reading the sketch's interaction code. Item-by-item audit is underway (see feedback_port_ux_faithfully.md).

**How to apply:** When implementing drafting-table drawing features, match the four behaviors above; observations/improvements are welcome but no silent reinvention. For un-refined areas (toolbar, etc.) freer design is OK. Darts implementation lives in Phase 3 per the roadmap — do not pre-empt.
