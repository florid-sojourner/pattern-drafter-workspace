---
name: Port UX from Justin's sketches faithfully (not verbatim, not reinvented)
description: When porting UI/interactions from Justin's sketches during the rebuild, read the sketch's interaction code first and match the refined behavior. Suggestions welcome; silent reinvention is not.
type: feedback
originSessionId: bddb0d38-cbf2-484a-a96f-f8c72abf9e70
---
When porting UX from Justin's existing sketches during the pattern-drafter rebuild, do not apply generic conventions. Read the sketch's drawing-tool handlers, stores, and interaction logic first, then match the behavior. Justin allows observations and improvements on any ported item ("verbatim" is too strong) but do not reinvent an interaction he already refined.

**Why:** On 2026-04-24 during Phase 2C I wrote new drawing tools from generic Pen-tool conventions without reading the sketch's UI code. Justin flagged regressions: close-by-snap-to-first-node missing (couldn't produce a Panel), curve-tool mechanics diverge from his refined design (curves in the sketch are accessed via drag-handles on existing segments, not a separate click-drag curve tool). R14 already says "preserve the sketch as read-only reference" — the intent was porting, not reinvention.

**How to apply:** Before writing UX code for the rebuild, enumerate sketch behaviors, read their implementations in `pattern-drafter-sketch/src/`, and produce a behavior-by-behavior plan before coding. Flag any intentional divergence from the sketch explicitly before writing code rather than after the user spots it. When unclear which sketch features were "refined" vs "draft," ask Justin before assuming.
