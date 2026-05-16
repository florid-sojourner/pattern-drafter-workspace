---
name: When an algorithm fails on hostile inputs, change the input shape, not the algorithm
description: Recognise faster when iterative tweaks aren't going to land; the right fix is upstream
type: feedback
originSessionId: 6246e3bc-9aa5-4af8-aa15-669267d16eaa
---
When trying to make an algorithm work on data whose *topology* is structurally wrong for it (not just noisy or imperfect), repeatedly tuning the algorithm is the wrong move. Identify the bad-shape data, fix the source that produced it, and the algorithm becomes a non-issue.

**Why:** 2026-05-04 spent a session iterating on Schneider 1990 least-squares cubic fitting (iterated Newton-Raphson, dense-sample error, collinear dedupe, long-chord split, several other tweaks) trying to smooth the SA cutting line, all on the polyline output of `polygon-clipping`. Each fix changed the output but the same ~1.7 mm max fit error persisted, because the input had pathological topology — collinear runs and huge straight chords mixed with dense smooth-curve samples inside one "smooth run." Chord-length parameterization can't handle that distribution; no algorithm tweak fixes it. Real fix is to replace polygon-clipping with segment-aware offset (line→line, cubic→cubic via Tiller-Hanson) so the output is well-shaped from the start.

**How to apply:**
- After 2 fix attempts that change behaviour without improving the metric, stop and ask: is the input *shape* compatible with this algorithm's assumptions, or am I trying to recover well-shaped output from ill-shaped input?
- If the input came from another step in the pipeline, look at that step's contract. Often the right fix is "make that step produce a different shape" rather than "make this step tolerate the wrong shape."
- Tell-tale sign you're in this situation: the same metric value (or near-same) recurs across structurally different fixes.
