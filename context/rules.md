# Pattern Drafter Rebuild — Rules

> **Purpose:** invariants we follow while building. Short, punchy, ready-to-reference during coding. Violations should trigger an explicit conversation, not silent drift.
> **Pair with:** `wiki.md` (context) and `roadmap.md` (forward plan).

## Product rules

**R1. Design target is Justin, not a hypothetical market.**
Every design decision should be testable against: "does this make the tool better for Justin?" If the answer is "it prepares us for hypothetical users X, Y, Z" — defer it.
*Why:* Premise 1. Commercial viability is upside. Over-designing for a market we haven't validated dilutes the one user we know.

**R2. Joyful iteration is the product.**
If a feature makes drafting faster but iteration more tedious, reject it. If a feature adds a tiny bit of drafting friction but makes iteration sing, ship it.
*Why:* Premise 2. The physical medium (paper) is the enemy because iteration on paper is miserable. Digital wins by making iteration mentally exciting.

**R3. Output quality is non-negotiable.**
Generated pattern pieces must look like pieces a sewist would be proud to cut. No "parametric-generator ugly" aesthetic. Curves smooth, proportions correct, pieces visually clean.
*Why:* Premise 3. Justin explicitly rejected online parametric generators on taste. The bar is hard.

**R4. Tool learning curve is minutes, not hours.**
Seconds to start drafting. 15–30 minutes to learn the drafting table core. 30–60 seconds per fringe feature. If onboarding requires a tutorial video longer than 2 minutes, something's wrong.
*Why:* Premise 4. This is close to the heart of the product, not a nice-to-have.

## Architecture rules

**R5. One document model. One source of truth.**
The pattern being edited is one object. Not split across stores. Not duplicated between wizard and editor. One.
*Why:* The sketch's dual-store design (`patternStore` + `draftingStore`) caused sync fragility. Fixing this is a core rebuild goal.

**R6. Object-model DSL is v1. Generative procedure DSL is deferred.**
The drafting table's internal language (panels, darts, seams, notches, grain, ops) is core. A DSL for auto-drafting from measurements+method is NOT v1 scope.
*Why:* Premise 6. Parametric is deferred because (a) it's moot without great drafting, (b) only one public procedure available. See `wiki.md` → The Two DSLs.

**R7. Everything runs client-side unless there's a reason it can't.**
No IP protection means no "run the DSL on the server to hide it." The drafting table interprets its own operations locally. Server is for persistence, auth, and anything that genuinely requires coordination.
*Why:* Premise 5. The sketch split architecture for IP protection we've dropped. Collapse the split.

**R8. User-level operations are first-class history commands.**
A knife cut + rotate is one undo entry, not 47 point-moves. A panel mirror is one entry, not "create N nodes + create path + reverse order." History is expressed at the level the user thinks.
*Why:* Premise 2 (joyful iteration). If undo operates at the SVG-node level, iteration feels broken; users can't mentally track what undo will do.

**R9. Seam-pair constraints are enforced by the document model, not checked after the fact.**
When you define "seam A on panel X pairs with seam B on panel Y," the model *knows*. Operations on X's seam A can react. Validation isn't a post-hoc linter.
*Why:* Makes "walk-the-seams" a live property rather than a report card. Matches how a sewist actually thinks about seams.

**R10. Auth + DB + cloud save from day 1.**
Justin needs multi-machine access. Local-first was rejected. Don't defer auth/DB to a later phase.
*Why:* Premise 7 (real user need stated 2026-04-23).

**R11. TypeScript with pragmatic strict. `any` at boundaries is OK.**
Strict mode by default. `any` is acceptable at I/O boundaries (DB JSONB, file load, untyped third-party libs). No generic gymnastics. No branded types unless there's a concrete bug they prevent.
*Why:* Domain is type-shaped; rebuild benefits from type safety; strict-mode fights don't pay.

## Process rules

**R12. Every rebuild decision is tested against the locked premises.**
When you're tempted to add or defer something, write the sentence "this serves premise N because..." If you can't, reconsider.
*Why:* Premises are the product's spine. Drift happens silently.

**R13. Don't write plan artifacts before skills run.**
If we're doing a structured planning exercise, run the relevant gstack skill first and let its output fill the docs. Don't pre-populate `roadmap.md` or similar with my own guesses.
*Why:* Mistake made on 2026-04-23 during this project kickoff. User flagged it. Saved as feedback memory.

**R14. Preserve the sketch as read-only reference during the rebuild.**
The sketch is being renamed to `pattern-drafter-sketch/` and preserved locally for reference. In the shared GitHub repo, the sketch's final commit will be tagged `sketch-final` before the clean-slate rebuild commit lands. Don't `rm -rf` or `git push --force` away the sketch during rebuild phases 1–6. Archive or remove only after the rebuild is self-sufficient and Justin confirms.
*Why:* The sketch has good parts (geometry primitives, expression engine, session handling, Dockerfile shape) that we'll lift verbatim. Also, it encodes the only drafting-system-as-data we've written to date.

**R15. Server-side tests exist from day 1.**
The sketch had none — this is explicitly called out as a rebuild gap. Don't ship auth or pattern CRUD without integration tests.
*Why:* Sketch's gap; fixing it is a rebuild goal.

**R16. Panel vertices have at most 2 incident segments within a single panel path. No forking.**
A panel models a fabric piece — its outline is a simple curve. Within one panel path, a node may appear at most once in `nodeIds` (so endpoints sit at 1 segment, interior nodes at 2, and closed-panel nodes at 2). Tools that could create a fork must instead JOIN, REJECT, or ROUTE AROUND. Across panels, sharing a vertex is fine (each panel keeps its own ≤2 count locally).
*Why:* Fabric edges don't branch. The Pen's mid-draw "click another panel's endpoint = join + finalize" mechanic exists to honor this rule. Self-intersection prevention (planned Phase 2 follow-up) is the same invariant viewed from a different angle.

**R17. Panel exists ↔ a closed `role: 'panel'` path exists. Operations that would open a panel's path are rejected at dispatch.**
A `Panel` entity (Phase 3) is created the moment a panel-role path becomes closed (close-by-snap, cross-panel join) and destroyed when its underlying path is deleted (cascade). Any operation that would open a closed panel's path — interior-node deletion that breaks the polygon, splitting a closed path into two open paths, etc. — is rejected by the same dispatch-gate pattern that gates self-intersection (R16's enforcement mechanism). Annotations referencing nodes that get removed in a *legal* edit cascade-clean silently, surfaced in undo history with a descriptive label so Cmd+Z recovers them.
*Why:* Phase 3 promotes panels to first-class entities with annotations attached. Without this invariant, a panel's `pathId` could point at a now-open path and the renderer would crash. Same PREVENT-during-action discipline Phase 2 closed on (R16, self-intersection prevention) — extended to Panel↔Path coherence.

## Anti-patterns (do NOT do)

**A1. Don't add a 4-step wizard before the drafting table.**
Sketch did this (person-type → measurements → style → drafting). It violates R4 (learning curve) and gates the product's value behind ceremony.

**A2. Don't hardcode garment-specific logic in the main flow.**
`if (basePattern?.garment_key === 'trouser') { makeWaistband(); makeFly(); }` is how the sketch does it. Don't. Use a registry or domain-op extension pattern.

**A3. Don't build parametric v1.**
Tempting because the sketch already has infrastructure for it. Don't. Premise 6. Focus budget on the drafting table.

**A4. Don't add multi-system drafting methods for the same garment.**
"Aldrich vs. Armstrong vs. TextileBlog men's trouser" is not a Justin-problem. One good method per garment is sufficient (once we even reach generative). No method-selection UI.

**A5. Don't add commercial machinery ahead of user value.**
No billing, subscriptions, team accounts, or pattern marketplace until there's a second real user. Auth exists because of R10 (multi-machine), not because of payments.

**A6. Don't build 3D preview or drape simulation.**
Huge scope, not on any user's critical path. 2D pattern pieces are the deliverable.

**A7. Don't silently introduce abstractions for hypothetical future needs.**
Three similar lines is better than a premature abstraction. The sketch's DSL was well-designed but premature for one garment — don't repeat with framework/plugin/registry-of-registries patterns.

**A8. Don't protect the drafting rules as IP.**
Premise 5. The sketch's server-side interpreter existed for this reason. We're not doing that anymore.

**A9. Don't duplicate pattern state in multiple columns or stores.**
Sketch's `saved_patterns.style_opts / measurements / draft_data` triple-store is a sync bug waiting to happen. One document, one representation.

**A10. Don't rename things to feel productive.**
If there's a real clarity win, rename. If you're renaming because the rebuild "feels" stuck, that's not a rebuild, it's anxiety.

## Decision log (where rules come from)

- R1–R10 locked 2026-04-23 via /office-hours interview (premises 1–7 + rebuild approach B).
- R11 adjusted 2026-04-23 after Justin challenged TypeScript default; landed on pragmatic strict.
- R13 from feedback memory (see `~/.claude/projects/-home-justin-Documents-Personal-pattern-drafter-workspace/memory/feedback_run_skills_before_writing.md`).
- Anti-patterns A1–A4, A8, A9 derived from sketch lessons (`wiki.md` → Sketch Lessons → Dropped).
- A5, A6, A7, A10 are general engineering hygiene aligned with the premises.
- R8 example refreshed 2026-04-27 (slash-and-pivot moved into Instruments as the knife; R8 itself unchanged in spirit).
- R17 added 2026-04-27 from /plan-eng-review on Phase 3 architecture (Panel↔Path lifecycle).
