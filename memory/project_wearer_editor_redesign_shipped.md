---
name: wearer-editor-redesign-shipped
description: "Wearer editor redesign SHIPPED 2026-05-15 on phase-4b-wearers (chunk 4B.9) — Variant A single column edit-in-place, soft session model, figureVariant gone, verb-form door labels. Spec at context/wearer-editor-redesign.md. 448 → 547 tests."
metadata: 
  node_type: memory
  type: project
  originSessionId: bd932061-2c85-4bb4-9489-35f4a7e95bcb
---

The wearer editor (`/wearers/:id`) redesign **shipped on 2026-05-15** as chunk 4B.9 on the `phase-4b-wearers` branch. Spec at `context/wearer-editor-redesign.md`. 22 decisions D4–D25 via `/plan-design-review` (CLEAR, 3/10 → 9/10) + `/plan-eng-review` (CLEAR). 448 → 547 tests green.

**Load-bearing decisions (now live in code):**

1. **Variant A IA — single column edit-in-place.** No banner gating editing. Reads walk `sessions[]` newest-first with sparse fallback (`valueForKey` in `wearerIO.ts`). Typing writes through to today's session record. Past sessions in a collapsed native `<details>` at the bottom.

2. **Soft session model (D5 + D11-bis).** `draftSession` removed entirely from type, validator, store, UI. One session record per calendar day. `setMeasurementValue(key, value)` creates today's record on first keystroke and amends it same-day. **Sparse — D16:** only keys typed today land in today's record; older values fall through. `setMeasurementValue(key, undefined)` (D23 Backspace) deletes today's key and drops today's record if it becomes empty.

3. **Past-session correction (D10).** `editFinalizedSession(sessionId, key, value)` is the one explicit destructive history-mutation path. Each expanded past-session row has a `correct a value` link that flips values to editable inputs. `takenOn` preserved.

4. **Per-section "Save as system" removed (D7).** Extraction is exclusively the Systems → `+ New system` form's job. Open question: verify the Systems form covers the use case; if not, surface a "Start from a wearer's bag" affordance as a fast-follow.

5. **`figureVariant` gone (D15).** Field, type, FIGURE_VARIANTS set, validator branch, `emptyWearerDocument` second param, `NewWearerForm` figure-radio fieldset — all deleted. Validator silently drops legacy `figureVariant` and `draftSession` on read. Migration 005 resets the wearers table on next deploy.

6. **Door labels are verbs (D14 + D24)** — `Draft Patterns` / `Measure Wearers` on hub door titles AND inside-door breadcrumb roots. Symmetric. See [[door-labels-are-verbs]].

7. **Long-form dates (D13).** `formatLongDate('2026-05-15')` → `'15 May 2026'`. Helper in `wearerIO.ts`.

8. **Throttled save indicator.** `aria-live="polite"` region announces transitions (idle→saving→saved→error) once, not per keystroke.

9. **CSS scope** (wearer editor only): two body greys (`#1a1a1a`, `#666`), two border tones (`#ece8df`, `#ddd`), cream `#fbf7ef` tint dropped. `<480px` breakpoint bumps touch targets to 44px+.

**Tests added:**
- `src/state/__tests__/wearerStore.test.ts` (new — closes D19 pre-existing gap). Cases: load / setName / setMeasurementValue (sparse + delete + fallback) / writeTodaysValue (pure) / editFinalizedSession / applySystem (dedup) / removeSystemSection / save-status.
- `src/io/__tests__/wearerIO.test.ts` (rewritten). Legacy-strip regressions for both figureVariant and draftSession; `valueForKey`, `formatLongDate`, `todayLocal` coverage. Clock seam via `vi.setSystemTime` per D17.

**Next:** dogfood on a clean dev DB, then merge `phase-4b-wearers` → `main`. The per-section `Save as system` removal is the one decision worth verifying against real use.

See also [[measurement-bag-model]] and [[hub-is-a-fork]] — both still load-bearing. Memory [[wearer-editor-design-review-pending]] is stale; this memory supersedes it.
