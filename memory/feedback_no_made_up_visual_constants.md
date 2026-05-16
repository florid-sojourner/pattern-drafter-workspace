---
name: Don't invent visual constants — ground in reference or flag uncertainty
description: When picking dimensions for visual elements (notch widths, glyph proportions, label offsets, etc.), don't silently invent numbers and present them as decisions. Either ground in a real reference (Big-Four envelope, textbook, sketch port) or surface the uncertainty so Justin can supply the reference.
type: feedback
originSessionId: c3455ebd-271f-486e-88f1-78f366096e6b
---
When a task involves picking sizes, ratios, or proportions for visual elements (notch wedge half-width, dart leg thickness, label offsets, glyph aspect ratios), don't just pick numbers that "feel about right" and ship them as if they were design decisions. Either:

1. **Ground in a real source.** Reference a sketch port, a textbook proportion, a Big-Four envelope photo, or a stated convention.
2. **Surface the uncertainty up front.** "I'm picking 1.5mm half-width because I don't have a reference — want to confirm against your pattern books or change it?" — let Justin redirect before committing to bad numbers.

**Why:** chunk 3.10 first ship had wedge half-width = 1.5 mm and double-spacing = 4.5 mm as fixed mm constants. They were made up — no source, no Big-Four reference, no sketch port. The result was a 6:1 spike when actual envelope notches are clearly triangular (~1.5:1). Justin caught it: "Where'd you get those dimensions?" Recalibration cost was small but the right move was not making them up in the first place.

**How to apply:** before shipping visual constants, check if there's a reference in the sketch, the design doc, or one of Justin's pattern books — and if not, say so. Bonus: prefer SA-fraction or relative-scale constants over fixed mm when the thing should preserve shape across sizes (Justin called this out explicitly: "I want to retain the SHAPE"). Pairs with the existing "Ask for artifacts before diagnosing visual bugs" memory — same spirit applied to *creating* visuals, not just debugging them.
