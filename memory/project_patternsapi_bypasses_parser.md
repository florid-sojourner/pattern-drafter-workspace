---
name: patternsApi.get bypasses documentIO.parse
description: New top-level Document fields must be normalized at the store boundary, not just in the parser, because cloud-loaded patterns skip the parser
type: project
originSessionId: 18976409-9f02-46c5-b0fb-2cdee5a28161
---
`src/io/patternsApi.ts` returns the raw server JSON cast to `Document` — it does NOT call `documentIO.parse`. AppShell + FileActions feed the result straight into `setDocument` / `loadSyncedDocument`.

**Why:** Saves a serialize-deserialize round-trip on every cloud load; server is treated as authoritative. The parser exists for *imported file* validation, not for cloud round-trips.

**How to apply:** When adding a new top-level field to `Document`:
- Don't rely on `documentIO.parse`'s forward-compat defaults to fill it for cloud-loaded patterns — they won't run.
- Normalize at the store boundary instead. There's a `normalizeLoadedDocument()` helper in `state/store.ts` (added 2026-05-09 after this exact bug broke loading of "Mark Pant Block" when `textAnnotations` was added). Extend that helper with the new field's default.
- Renderers that read the new field should also tolerate `undefined` defensively, in case any code path bypasses both the parser and the normalizer.

**Past incident (2026-05-09):** Added `doc.textAnnotations` + new `TextLayer` doing `Object.values(doc.textAnnotations)`. Old saved patterns crashed-to-blank on load because `textAnnotations` was undefined. Data was intact, render path threw.
