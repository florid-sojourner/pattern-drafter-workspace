---
name: Path.offset uses polygon-clipping; output topology is hostile to cubic re-fit
description: Why the SA cutting line is faceted and what the real fix looks like
type: project
originSessionId: 6246e3bc-9aa5-4af8-aa15-669267d16eaa
---
`Path.offset()` (in `pattern-drafter/src/engine/geometry/Path.ts`) routes through `polygon-clipping.union` of a flattened-cubic polygon. Output is a polyline (lineTo-only). For SA rendering this manifests as visible chord facets at high zoom and on sewn garments (the cut edge becomes the stitch line, so facets transfer to the seam).

Two cheap mitigations shipped: `smoothByCorners` Catmull-Rom re-fit removed (commit `a24d139` — was producing X-crossings/fishhooks/false-corner facets), flattening density bumped 64→256 (commit `313c2d6`). Acceptable but not fully smooth.

A third mitigation — Schneider 1990 least-squares cubic re-fit of the polyline — was attempted 2026-05-04 and abandoned. The polygon-clipping output mixes exactly-collinear runs with one or two huge straight chords inside what is nominally one smooth run between corners. Chord-length parameterization clusters all dense points near u=0, max-error always lands at the same pathological index, recursion always splits there, depth-caps to chord facets. No algorithm tweak fixes it.

**Why:** the cut edge becomes the stitch line during sewing — facets transfer to the seam, so smoothness is product quality, not vanity polish.

**How to apply:**
- The real fix is segment-aware offsetting inside `Path.offset()` itself: traverse the original path edge-by-edge and produce one output edge per input edge of the same kind (line shift perpendicular for line edges; Tiller-Hanson or de Casteljau-based cubic offset for cubic edges, splitting where curvature exceeds offset radius). Polyline middleman disappears; no re-fit needed. `polygon-clipping.union` stays as fallback for self-intersecting input (which is currently unreachable via user action).
- This is captured in `context/roadmap.md` → Deferred Quality Concerns → "Cutting-line smoothness via segment-aware offset" — revisit when facets become real friction in Phase 6 self-use, or earlier if an external user notices.
- Don't try to re-fit the polygon-clipping polyline again. Iterating on the algorithm is what wasted 2026-05-04.
