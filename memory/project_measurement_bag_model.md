---
name: measurement-bag-model
description: "Wearer measurements are an open-keyed bag rendered as per-system sections; v1 ships empty bag + user-CRUD on measurement systems; built-in (Aldrich/Quiring/Big-Four) deferred; the noun is 'system', not 'template'"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8a95c1c5-178a-4f71-9e77-e15ea3c9cb13
---

Measurements on a wearer are stored as an **open-keyed bag** of `{ systemId, key, label, value }` entries — no fixed schema, no canonical 27-key set. The user types what's on their slip of paper. The app organizes by **provenance** (which system the entry came from), never by name.

**Vocabulary: "system", not "template".** Renamed 2026-05-14 to match patternmaking-industry usage ("Aldrich system", "Müller & Sohn system", "Quiring system"). Code: `systems` table, `SystemDocument`, `applySystem`, `systemId`. UI labels: "Systems" tab, "Save as system", "Apply a system". The user's own saved key-sets read coherently as "Justin's system" too.

**Provenance is per-entry (this is load-bearing).** Two systems may name a measurement the same word and measure it differently — e.g., Aldrich-Bust ("across the fullest part, parallel to the floor") vs. Armstrong-Bust ("level at the apex"). Blending them silently produces a wrong value the drafter will trust. So entries are NEVER merged by name. A wearer with Aldrich+Armstrong both applied has *two* Bust entries, distinguished by `systemId`. Manual one-offs use `systemId: null` and live in an "Other" section.

**Systems as data:** A system defines a *set of keys* (name + optional group + optional range). Applying a system to a wearer adds its keys to the wearer's bag, additively (never destructive). Two scopes:

- **User-defined systems (v1, shipped 2026-05-14):** A drafter saves a key set under their account, applies it to any wearer they own. Two creation paths: (1) "+ New system" form on the Measurements → Systems tab (name + per-key min/max/description); (2) "Save as system" per-section button on the wearer page (captures section's bag entries). Applies via "Apply a system ▾" picker at the bottom of the editor.
- **Built-in systems (deferred):** Aldrich, Armstrong, Big-Four, Quiring, etc. The `systems` table already has the `is_built_in` column and the route protections (403 on PUT/DELETE) for built-in rows. v1 ships zero seeded built-ins per Justin's 2026-05-14 directive. When they land, it's content-only — no schema change.

**Applying a system is additive, idempotent within scope, never destructive.** Pull in Aldrich → bag gains an Aldrich-scoped entry for every Aldrich key. Pull in Armstrong on top → bag gains a separately-scoped Armstrong entry for every Armstrong key (Armstrong-Bust coexists with Aldrich-Bust, distinct systemIds). Repeated application of the same system is allowed: each application creates a fresh section; store-side dedup suffixes any colliding bag-keys (`waist`, `waist_2`, `waist_3`).

**Wearer page renders sections by systemId**, not by name. Each section is a `<fieldset>` with the system's name as legend. Order of sections is encounter order. Manual one-offs (`systemId: null`) live in a section labeled "Other." Per-section "Save as system" was removed 2026-05-15 (see [[wearer-editor-redesign-locked]]); system creation lives only in Measurements → Systems.

**Empty bag is the default.** New wearers start with `measurements: []` — no hardcoded starter, no auto-applied set. The editor's empty state shows two equal-weight CTAs: `[+ add measurement]` and `[ Apply a system ▾ ]` side-by-side. As of the 2026-05-15 redesign, the `draftSession` concept is removed — typing writes through directly to today's session record (one record per calendar day, auto-created on first keystroke). See [[wearer-editor-redesign-locked]] for full details.

**Why (rename + drop starter):** Justin's 2026-05-14 feedback: (1) "template" is generic UX language; "system" matches what hobby + pro drafters actually call these key-sets. (2) The system-wide starter template was over-reach for v1 individual-self scope — empty bag is the honest start; the user picks what to populate. The schema kept the `is_built_in` column for the future tranche so no migration is needed when Aldrich/Quiring built-ins land.

**Why (per-system namespaces, original 2026-05-13 reframe):** The "union of all measurements across drafting systems" approach the original Phase 4B plan called for produced an intimidating 27-input wall of fields, most of which any individual project doesn't need. Different drafting systems use overlapping-but-not-identical key sets; forcing a single union obscures system-specific vocabulary (Aldrich "Front Length" vs Armstrong "Center Front Length"). The bag-of-keys + per-system-provenance model lets the drafter focus on the system they're working in. Also: "the competitor is the slip of paper" — the slip has whatever the drafter wrote down, not a fixed schema. See [[measurements-are-optional-convenience]].

**How to apply:**
- `wearer.measurements` is an open bag `[{ systemId, key, name, value, ... }]`. Validator accepts legacy `templateId` field name on entries for dev DB back-compat with pre-rename docs (one-line fallback in `validateMeasurementEntry`).
- `(systemId, key)` is the entry's identity, not just `key`. Applying Aldrich + Quiring + Armstrong creates three separate sections; if all three name a measurement "Waist," the user writes the value three times. No de-dup, no shared-value-multi-section, no "first applied wins."
- New wearers ship with empty `measurements: []`. Don't reintroduce hardcoded measurement field grids anywhere — if a key-set is needed, it becomes a system.
- New-system form lives at Measurements → Systems → "+ New system". Captures name, optional description, and one-or-more key rows (name + optional slug + optional min/max cm + optional one-line description). Min/max drive the wearer field's out-of-range warning.
- The `is_built_in` column + protections stay in the schema for the future tranche; no built-ins are seeded today.
