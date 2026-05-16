# Wearer editor redesign

**Date locked:** 2026-05-15
**Source:** `/plan-design-review` run against `pattern-drafter/src/ui/wearers/WearerEditor.tsx`. Driven by Justin's 2026-05-14 evening complaints captured in memory `[[wearer-editor-design-review-pending]]`.
**Branch:** lands on a follow-on chunk of `phase-4b-wearers` (or a fresh `wearer-editor-redesign` branch). Must merge before `phase-4b-wearers` → `main`.

## What this is

A spec for redesigning `/wearers/:id` to fix five complaints: disorganized IA, overall complexity, unasked-for notes input, wordy copy, and the read-only "viewing past session" banner.

## Decisions locked

| ID | Question | Decision |
|---|---|---|
| D4 | IA direction | Variant A — single column, edit-in-place, sessions demoted |
| D5 | Session lifecycle UI | Soft — sessions invisible to user, no draft/save/discard chrome |
| D6 | Past-sessions visibility | Collapsed accordion at bottom; expand → list of `<date> · <N values>` rows |
| D7 | Per-section "Save as system" | **Removed** from wearer editor entirely; lives only in Measurements → Systems |
| D8 | Past-session view mechanism | Inline expansion (read-only mini-table within the accordion row) |
| D9 | Empty-bag CTA hierarchy | Both visible, equal weight: `[+ add measurement]` and `[ Apply a system ▾ ]` |
| D10 | Edit a past session's values | Gated affordance: expanded past session has a `correct a value` link → values become editable inline → save updates that session in place, keeps original `takenOn` |
| D11-bis | When a measuring session becomes a record | **As soon as you write anything.** One record per calendar day. First keystroke creates today's session record; subsequent edits on the same day amend it. No `draftSession` concept in user OR data model. |
| D12 | Title (wearer name) editing | Out of scope; current autosave-on-keystroke stays |
| D13 | Date format | Long form: `15 May 2026` |
| D14 | Hub door labels | Verb phrases: `Draft Patterns` / `Measure Wearers`. Bundled into this PR. |
| D15 | `figureVariant` cleanup | Bundled into this PR. Remove from `WearerDocument`, validator, types, NewWearerForm radio buttons, vestigial component references. |
| D16 | Today's session record contents | **Sparse.** Only the keys actually typed that day land in `sessions[0].values`. Display falls back through prior sessions for unedited keys. History row count reads "today: N values" where N = literal keys-typed-today. |
| D17 | Clock seam for tests | `vi.setSystemTime` in tests. Production code reads `new Date()` directly. |
| D18 | `notes` field | `WearerDocument.notes` field stays (no UI; preserves any dev-DB data). Session-level notes (already on `WearerSession.notes`) deferred as a fast-follow TODO: surface at the bottom of a session's measurement list. |
| D19 | wearerStore baseline tests | Bundle a full `src/state/__tests__/wearerStore.test.ts` covering all actions (existing + new). Closes a pre-existing zero-coverage gap. |
| D20 | Test scope | Full: store + io + component branches. Estimate revised from review: closer to 600-900 lines (subagent challenge). |
| D22 | Multi-tab write conflicts | Accept LWW (status quo for both patterns and wearers routes). Flag as a known constraint in the plan; TODO for an etag/If-Match audit when multi-device editing becomes a real-world problem. |
| D23 | Delete semantics under sparse model | Backspace = revert to prior session value. Clearing today's input deletes the key from today's record; fallback resurrects the prior value. No tombstone in v1. |
| D24 | Door/breadcrumb consistency | **Both verb-form.** Inside-door breadcrumb root becomes `Measure Wearers` / `Draft Patterns` too. Overrides earlier "noun root" intent. |
| D25 | Existing `draftSession` data | Drop the wearers table outright via new migration `005_wearers_reset.sql`. No production data exists (Justin is the only tester); the validator's silent-drop is a defense-in-depth fallback, not the primary path. Fly + dev DB both re-run migrations on boot, so the table is recreated empty on next deploy. |

## Page structure (final)

```
+----------------------------------------------------------+
|  [breadcrumb: Hub > Measure Wearers > Mark Jaynes]       |
+----------------------------------------------------------+
|  Mark Jaynes                              saved          |   <- title row
|  -------------------------------                         |
|                                              unit cm     |   <- meta strip (unit only)
|                                                          |
|  Aldrich                                                 |   <- system section
|   Bust                              [  102 ] cm          |
|   Waist                             [   86 ] cm          |
|   ...                                                    |
|                                                          |
|  Other                                                   |   <- only if has manual entries
|   Inseam variant                    [   83 ] cm          |
|                                                          |
|  + add measurement   |  [ Apply a system v ]             |   <- secondary affordances
|                                                          |
|  v  Past sessions (2)                                    |   <- collapsed accordion
+----------------------------------------------------------+
```

Empty bag (no systems applied yet, no manual entries):

```
+----------------------------------------------------------+
|  Untitled                                  saved         |
|  -------------------------------                         |
|                                              unit cm     |
|                                                          |
|  [ + add measurement ]    [ Apply a system v ]           |   <- two CTAs side-by-side
|                                                          |
+----------------------------------------------------------+
```

## What gets removed

From `WearerEditor.tsx`:

- `notes` input + label + `setNotes` wiring
- `wearer-editor__viewing-past` banner + render branch
- `wearer-editor__finalize` footer (Discard draft + End session and save buttons)
- `SaveSectionForm` component + all `wearer-editor__save-template*` classes
- `wearer-editor__section-action` (Save-as-system button in legend)
- `savingSectionId`, `saveSystemName`, `saveSystemError` state
- `onSaveSectionAsSystem` callback
- `showFinalizeFlash` flash flag (no end-session ceremony)
- `wearer-editor__empty` empty-state prose

From the store (`wearerStore`):

- `startDraftSession`, `finalizeSession`, `discardDraftSession` actions
- `draftSession` field on the doc (becomes implementation detail of "today's session record" if needed; goal is to remove entirely)
- `setDraftValue` becomes `setMeasurementValue(key, value)` — writes through to today's session, auto-creating today's session record on first call of the calendar day

From `wearerIO`:

- `DraftSession` type if no longer referenced
- `FigureVariant` type + `FIGURE_VARIANTS` set + `figureVariant` field on `WearerDocument`
- The `figureVariant` validation branch (`wearerIO.ts:146-147`)
- The default-value reference at `wearerIO.ts:106,112` (drop the param)
- Validator drops `figureVariant` silently when reading legacy docs (back-compat for existing DB rows)

From CSS:

- `.wearer-editor__viewing-past`
- `.wearer-editor__notes`
- `.wearer-editor__start`, `__start h2`, `__start-note`, `__start-btn` (already orphaned by 2026-05-14 evening work; confirm + delete)
- `.wearer-editor__finalize`, `__discard`, `__end-session`
- `.wearer-editor__save-template*` (5 classes)
- `.wearer-editor__section-action`
- `.wearer-editor__empty`, `__empty-hint`
- `.wearer-editor__other-empty` (already orphaned per the file)
- `wearer-editor__apply` cream tint background → drop tint, keep 1px border

## What gets added

- `setMeasurementValue(key, value)` store action with calendar-day session auto-create logic; **sparse semantics** per D16 (only the keys actually typed land in `sessions[0].values`)
- `editFinalizedSession(sessionId, key, value)` store action (D10 correct-a-value flow); preserves original `takenOn`
- `valueForKey` rewritten to walk `sessions[]` backward until a value is found (sparse-fallback per D16); not just `sessions[0]`
- Past-sessions outer accordion using native `<details>`/`<summary>`; per-row inner expansion uses a `<button aria-expanded>` + adjacent panel (native `<details>` doesn't natively give nested per-row expansion)
- Inline `correct a value` link on expanded past sessions
- `formatLongDate(iso: string): string` utility (`"2026-05-15"` → `"15 May 2026"`)
- `SaveIndicator` becomes an `aria-live="polite"` region — **throttled** so it only announces status transitions (`idle → saving`, `saving → saved`, `saving → error`), not every keystroke. Avoids screen-reader spam.
- New migration `server/src/migrations/005_wearers_reset.sql` — drops + recreates the `wearers` table. Runs on next `npm run migrate` (dev) and on next deploy (Fly).

## State matrix

| State | UI |
|---|---|
| Loading | top-of-page skeleton, no spinner |
| Load 404 / error | inline error + `Back to wearers` link (unchanged from current) |
| Empty bag, 0 systems available | two CTAs side-by-side; `Apply a system ▾` disabled, with `Build a system in Systems →` link below it |
| Empty bag, has systems | two CTAs side-by-side; both enabled |
| Populated bag, no manual entries | system fieldsets only; below them: `+ add measurement` inline link + `[ Apply a system ▾ ]` |
| Populated bag, has manual entries | system fieldsets + `Other` fieldset with entries + inline `+ add measurement` row inside `Other` |
| Saving | `saving…` in `aria-live` region near title |
| Saved | `saved` in same region |
| Save failed | red `save failed: <reason>` in same region; retry on next edit |
| Past sessions accordion (collapsed) | `▽ Past sessions (2)` (uses native `<details>`) |
| Past sessions accordion (expanded) | list of `<15 May 2026> · <8 values>` rows; clicking a row expands it inline |
| Past session expanded | read-only mini-table of that session's values + small `correct a value` link |
| Past session in correction mode | values become editable inputs; `Save` / `Cancel` buttons; on save → `editFinalizedSession` writes through |
| Out-of-range field | per-field warning, `aria-describedby` on input (was free-floating) |
| Apply-system: pick succeeded | section auto-appears; first row gets focus |
| Apply-system: error | red text below picker (unchanged) |

## Copy table

| Element | Old | New |
|---|---|---|
| Notes input | `optional notes` | removed |
| Banner | `Showing values from 2026-05-13. Start a new session?` | removed |
| Empty bag prose | `This wearer has no measurements yet. Apply a system to seed a set of measurement fields, or add a one-off measurement below.` | removed; replaced by 2 controls |
| Add-row placeholder | `add measurement (e.g., Chest pocket drop)` / `add measurement (e.g., Bust)` | `Measurement name` |
| Apply-system label | `Apply a system` | unchanged |
| Apply-system empty placeholder | `(no systems yet — create one in Measurements → Systems)` | picker disabled; small text below: `Build a system in Systems →` |
| Apply-system populated placeholder | `Choose one…` | unchanged |
| Past-sessions accordion title | `Sessions (N)` | `Past sessions (N)` |
| Past-session row | `<date> · <N> values · <notes>` | `<15 May 2026> · <8 values>` (notes shown only if present) |
| Past-session edit affordance | n/a | `correct a value` (inline link, expanded state only) |
| Footer (discard / end session) | `Discard draft` / `End session and save` | removed |

## Companion changes (D14 + D24 — hub door labels + breadcrumb roots)

Sites to touch:

- `src/ui/HubPage.tsx:23` — `Drafting` → `Draft Patterns`
- `src/ui/HubPage.tsx:29` — `Measurements` → `Measure Wearers`
- `src/ui/WearersList.tsx:39` — breadcrumb root `Measurements` → `Measure Wearers`
- `src/ui/wearers/WearerEditor.tsx:123,140,212` — breadcrumb root `Measurements` → `Measure Wearers`
- `src/ui/PatternsList.tsx:132` — breadcrumb root `Patterns` → `Draft Patterns`
- Routes (`/wearers`, `/patterns`) unchanged — only the visible labels move

Pattern: **both door label AND inside-door breadcrumb root use the verb phrase, identically**. Reads as `⌂ Hub › Measure Wearers › Mark Jaynes` and `⌂ Hub › Draft Patterns › Trouser block`. Symmetric across both doors (D24 override of an earlier verb/noun split).

## Companion changes (D15 — figureVariant cleanup, expanded)

Beyond the data-model removal, the cleanup also touches `WearersList.tsx`:

- `WearersList.tsx:283-360` — `NewWearerForm` component drops the entire `<fieldset>` of figure radio buttons. Form collapses to a single name input + Cancel/Create.
- `WearersList.tsx:92, 299, 342` — `onCreate` signature simplifies to `(name: string) => Promise<void>`; `figure` state + `figureVariant` references removed.
- `wearerCreators.ts:5` — `wearerNew(name)` drops the `figureVariant` param.
- `wearerIO.ts:104-115` — `emptyWearerDocument(name)` drops the `figureVariant` param.
- All existing `wearerIO.test.ts` cases that pass `figureVariant` need updating; the `figureVariant must be one of` validation test gets replaced with a "legacy field silently dropped" regression test.

No pattern code reads `wearer.figureVariant` (audited 2026-05-15 — 23 references all inside the four expected files + tests). Safe to delete.

## Mobile / responsive

- **Viewport < 480px:** stack `.wearer-field` to `label / [input chip · unit]` — label on its own line at 1rem; input + unit on a second 0.9rem line, right-aligned. Use `@container` query if `.wearer-editor__work` is the container; else `@media (max-width: 480px)`.
- **Touch targets:** all `<input>`, `<button>`, `<select>` minimum 44×44 CSS px. Buttons go from ~32px tall → 40-44px.
- **Empty-bag CTA pair:** stacked vertically below 480px (not side-by-side).

## A11y

- `SaveIndicator` → `aria-live="polite"` so screen readers announce save state without taking focus.
- Past-sessions accordion → native `<details>`/`<summary>` (keyboard expand/collapse without custom JS).
- Out-of-range warning → `aria-describedby` on the input, not free-floating text below.
- Tab order: title → unit picker → first measurement row → next row → ... → `+ add measurement` link → `Apply a system ▾` → `<details>` summary for past sessions.

## Token cleanup (folded into this redesign)

Within the wearer-editor's CSS scope only — not a system-wide token consolidation. Collapse:

- Body text greys → 2 tones: `#1a1a1a` (strong), `#666` (muted). Drops `#888`, `#555`, `#5a4a36`.
- Border tones → 2 tones: `#ece8df` (rule), `#ddd` (input border). Drops `#ccc`, `#d8d2c2`, `#e6e1d6` (consolidate to `#ece8df`).
- Drop the `#fbf7ef` cream tint on the apply-system box; keep 1px border only.

## Engineering implications

- **Data model:** `WearerDocument.draftSession` is **removed outright** from the type. Validator strips it from legacy docs (defense-in-depth) but the wearers table is reset on deploy anyway, so the validator strip is for development docs and the rare in-flight edit straddling a deploy.
- **Session auto-create (`setMeasurementValue`):** Only `setMeasurementValue` creates today's session record. `applySystem` and `addManualMeasurement` are bag-structure operations — they mutate `doc.measurements` only, never create a session record. A user can apply a system on day 1 and walk away with no values typed; `sessions[]` stays empty.
- **Sparse session semantics (D16):** `setMeasurementValue(key, value)` walks `doc.sessions`. If `sessions[0]?.takenOn === today`, mutate `sessions[0].values[key]`. Otherwise unshift `{ id, takenOn: today, values: { [key]: value } }`. If `value === undefined`, delete the key from today's record; if today's record then has empty `values`, drop the record itself (avoids stranded empty session-rows after Backspace).
- **`valueForKey` fallback (D16 correctness consequence):** must walk `sessions[]` backward from `sessions[0]` until a value is found for the key, not just check `sessions[0]`. With sparse storage, the latest session may not have the key at all. O(N·R) per render is fine at expected scale (N≤50, R≤30).
- **Delete semantics (D23):** Backspace clears today's input → `setMeasurementValue(key, undefined)` → key dropped from today's record → fallback resurrects prior value. No tombstone in v1. Reads as "I didn't measure this today."
- **`takenOn` timezone:** Use local-time YYYY-MM-DD, not UTC. `new Date().toLocaleDateString('en-CA')` returns `2026-05-15` in local time (`en-CA` is the only locale whose `toLocaleDateString` defaults to ISO format). Avoids the "user types at 8pm Pacific, gets tomorrow's UTC date" surprise.
- **Same-day rollover:** at midnight the next `setMeasurementValue` call rolls into a new session. Acceptable; users rarely measure at midnight.
- **Clock seam (D17):** Tests use `vi.setSystemTime(new Date('2026-05-15T10:00:00'))` to control which date the production code observes. No production-code change.
- **Validator change:** `validate(doc)` strips legacy `draftSession` and `figureVariant` fields silently. The wearers table reset (migration 005) means we won't actually encounter legacy fields in normal operation; this is belt-and-braces for any edge case.
- **`editFinalizedSession` (D10 correct-a-value):** new store action; finds session by id; mutates `values[key]`; keeps `takenOn`; goes through the same autosave path. **Tradeoff acknowledged:** violates the prior "history is never destructively edited" invariant in `wearerStore.ts:13`. Update the store-comment to read "history is read-only by default; explicit `correct a value` affordance is the only mutation path." No audit-log v1 (deferred TODO).
- **Multi-tab/multi-device write conflicts (D22):** No etag, no version check, no merge. The wearer's full document is PUT on each autosave; whoever saves last wins. Same as the patterns route. **Known constraint, accepted scope.** TODO: cross-route etag audit if multi-device becomes a real-world problem.
- **Pattern→wearer dangling references after migration 005:** Patterns with `wearerId` referencing dropped wearer rows render as "no wearer" gracefully per the existing weak-reference convention. Listed: 3 patterns at the dev DB at truncate time (`New garment`, `Maya skirt`, `A-line shirt`). No fix needed; the references just become orphans the client tolerates.

## Not in scope

- **DESIGN.md consolidation.** Token system is workspace-wide; lives in a separate `/design-consultation` task already flagged in the roadmap (line 375).
- **Mobile / tablet pass for the rest of Phase 4B.** Wearer-editor mobile pass is in scope here; other Phase 4B pages (`WearersList`, Systems tab, etc.) remain on the deferred list.
- **Built-in measurement systems** (Aldrich, Quiring, etc.) — already deferred separately (line 378).
- **Color contrast audit** — separate task (line 376).
- **Wearer-name title edit refinement** (D12) — deferred to future polish pass.
- **Growth-tracking / diff visualization across sessions** — would be a delightful add but explicitly not asked for. Note as TODO if Justin sees the editor and wants it.
- **Wearer-level notes field** — if Justin ever wants notes on a wearer, that's a new feature, separately spec'd.

## TODOs to consider after implementation

- **Session-level notes textarea at the bottom of a session's measurement list** (D18). `WearerSession.notes` field already exists; just needs a write affordance + a small text input at the bottom of the active session's measurement list (and a read-only display under each past-session expansion). Justin's framing: "if our design changes land well, it might be nice to actually still have notes — at the bottom not the top."
- **DESIGN.md run** — would resolve the cross-page token-system gap that surfaced in Pass 5.
- **Typography accent on wearer-name title** — small serif accent on `h1` would lean into the "digital notebook" framing; needs a system-wide font decision first.
- **Per-row "last set: 13 Apr" affordance** — would surface drift when re-measuring a growing kid. Deferred at design review (D11-bis simplification removed the need in v1).
- **Growth tracking view across sessions** — useful for parents; not motivated by Justin's slip-of-paper use case directly.
- **Cross-route etag/If-Match audit** (D22) — when multi-tab editing becomes a real-world problem on either patterns or wearers, address concurrency control across both flows. Currently neither has it.
- **`editFinalizedSession` audit log** — the "correct a value" affordance destructively rewrites history. v1 has no audit trail. If wearer data ever has external consumers (cloud sync, prints, etc.), reintroduce the invariant via append-only edit log.
- **"Start from a wearer's bag" affordance on the Systems → New system form** — verifies-or-fills D7's open question. If extracting reusable key-sets from a typed bag is genuinely common, the Systems form needs to support importing from a wearer's current bag (preserving labels, ranges, descriptions, grouping). Currently from-scratch only.
- **Recommended next: `/qa` after implementation** to dogfood the redesign end-to-end on a clean DB.

## Open question (one)

The redesign deletes the per-section "Save as system" button (D7). Verify that the Measurements → Systems → `+ New system` form actually covers the use case it was substituting for. If the per-section flow was the fastest way to extract "the keys I just typed in" as a reusable system, the Systems form needs to support importing/seeding from an existing wearer. Currently it does not — it's a from-scratch form. Flag for verification at implementation time; may require a small "Start from a wearer's bag" affordance on the Systems form.

---

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|---|---|---|---|---|---|
| CEO Review | `/plan-ceo-review` | Scope & strategy | 0 | — | Not run; scope is single-page redesign, not a strategy question. |
| Codex Review | `/codex review` | Independent 2nd opinion | 0 | — | Not available this session (no OpenAI key). Outside voice ran via Claude subagent instead. |
| Eng Review | `/plan-eng-review` | Architecture, code quality, tests, performance | 1 | CLEAR (PLAN) | 0 critical gaps; 4 architecture issues raised + resolved (D16-D19); 0 code-quality blockers; full test coverage scoped (D20, ~600-900 lines); 0 performance issues. Outside voice raised 17 findings; 4 cross-model tensions (D22-D25) resolved with user. |
| Design Review | `/plan-design-review` | UI/UX gaps | 1 | CLEAR (PLAN) | Score 3/10 → 9/10. 12 decisions made (D4-D15). |
| DX Review | `/plan-devex-review` | Developer experience gaps | 0 | — | Not applicable (not a developer-facing surface). |

**UNRESOLVED:** 1 (per-section `Save as system` removal — verify the Systems form covers the extracted use case before merge; otherwise add a "Start from a wearer's bag" affordance).

**Decisions made:** 22 (D4-D25).

**CROSS-MODEL:** Outside voice (Claude subagent, fresh context) surfaced 17 findings. 4 were elevated to user decisions (D22 multi-tab LWW, D23 delete semantics, D24 verb/noun split, D25 draftSession data). The rest folded into the plan write-up. Net: subagent caught real gaps (timezone, aria-live spam, nested expansion mechanism, valueForKey fallback algorithm); rejected one "simpler alternative" because it contradicted accepted UX decisions D5+D11-bis.

**VERDICT:** CEO not run; Design + Eng CLEARED. Plan is implementation-ready. Next step: `/ship` after implementation, or `/qa` if you want to dogfood the redesign end-to-end on a clean DB first.
