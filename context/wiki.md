# Pattern Drafter Rebuild — Wiki

> **Purpose:** shared understanding. What we know about the domain, the sketch's lessons, the premises, and the stack. Grows as we learn. Distinct from `roadmap.md` (forward plan) and `rules.md` (invariants).

## What the product is

A **domain-specific CAD tool for garment pattern drafting**. Browser-based, account-backed (so patterns sync across machines), cloud-deployed (Fly.io).

v1 = a great manual drafting table with rich domain semantics (darts, seam pairs, slash-and-pivot). Later = parametric auto-draft layered on top.

Target user: Justin and hobby sewists like him — motivated people who want custom-fit garments but have been burned off by existing tools (professional pattern software: too complex; online parametric generators: ugly output).

## Why it's being rebuilt

An experimental sketch exists at `/home/justin/Documents/Personal/pattern-drafter-workspace/pattern-drafter/`. It was a learning exercise (React + Fastify + Postgres, JSON DSL for drafting systems, one pattern implemented: men's trouser TextileBlog method). Lessons from the sketch are in this file (see **Sketch Lessons**), and the rebuild keeps what worked while restructuring around better product premises.

**Rebuild logistics (resolved 2026-04-23):**
- Local: sketch will be renamed to `pattern-drafter-sketch/` at Phase 1 kickoff; rebuild claims `pattern-drafter/`.
- GitHub: same repo (private). Current sketch HEAD tagged `sketch-final` before clean-slate first commit.
- Deployment: same Fly.io app.
- Product name: kept as "pattern-drafter."

## Primary User (locked)

**Justin.** A hobby sewist who:
- Followed Cornelius Quiring's paid trouser-drafting tutorial
- Drafted pants on paper with an oversized woodworker's construction square, pencil, tape, scissors
- Got pants that didn't fit well
- Found existing software unintuitive, complex, often worse than paper
- Found online parametric generators produced ugly output
- Stopped sewing for a long time
- Works across multiple machines — needs cloud sync, not local-only

Secondary target (aspirational, not validated): hobby sewists making clothes for themselves or family. See `roadmap.md` → Phase 7 for the assignment to validate this.

## Premises (locked 2026-04-23)

1. **Primary user is Justin** + hobby sewists like him. Commercial is upside, hobby-tool is fallback.
2. **The physical medium is the enemy.** Paper + pencil makes both drafting *and* iteration painful. Digital should make both mentally exciting — iteration especially. Joyful iteration > fast iteration.
3. **Output quality is a hard bar.** Garments must look attractive and wearable. "Parametric-generator ugly" is disqualifying.
4. **Learning curve is near the heart of the product.** Seconds to start. 15–30 min to learn the drafting table core. 30–60s per fringe feature. Drafting a new garment legitimately takes longer — that's work, not friction.
5. **IP protection of drafting rules is not a goal.** The server-side / client-side DSL split from the sketch collapses. Drafting methods are published in books anyway.
6. **v1 is a CAD tool for drafting, not a parametric generator.** Parametric auto-draft is deferred because: (a) parametric output is just a starting point — moot without great drafting; (b) insufficient publicly documented procedures for a multi-method system; (c) the drafting table is useful on its own with user-supplied book methods.
7. **Backend + auth + cloud DB from day 1.** Multi-machine access is a real user need.

## Domain Glossary

| Term | Definition |
|------|------------|
| **Pattern** | The full set of pieces required to sew a garment (e.g., front trouser panel + back trouser panel + waistband + fly). One saved document in the tool. |
| **Panel** | A single closed shape that gets cut from fabric. Has a grain line, a seam perimeter (edges), darts, notches. |
| **Seam edge** | A typed edge of a panel, labeled with its role (inseam, outseam, waist, crotch). Paired to an associated seam edge on another panel — the lengths should match. |
| **Seam pair** | Two seam edges (one per panel) that will be sewn together. Constraint: lengths must match (within tolerance). Mismatches are surfaced by `walk-the-seams`. |
| **Dart** | A wedge folded out of a panel to shape fabric over 3D curves of the body (e.g., waist dart on trouser back). Has two legs meeting at a point. Consumes seam length at the seam it originates from. |
| **Notch** | A positional mark on a seam, used at sewing time to align panels correctly. |
| **Grain line** | Direction of the fabric's warp threads relative to the panel. Critical for drape. |
| **Construction line** | A guide line drawn during drafting that isn't part of the cut panel (e.g., hip line, crotch depth line). |
| **Slash-and-pivot** | A classic pattern-manipulation operation: cut the panel along a line, pivot one side by an angle to add or remove shape (e.g., add room in the seat). Must preserve panel topology. |
| **Walk-the-seams** | A validation operation: traverse each seam pair and confirm the two edges have matching lengths. Surface mismatches. |
| **Ease** | Extra room added beyond body measurements for comfort, movement, or style. |
| **Drafting method / system** | A published procedure for turning body measurements into pattern pieces for a specific garment. Examples: TextileBlog, Aldrich, Armstrong, Cornelius Quiring. The sketch had one (TextileBlog trousers). |

## The Two DSLs (important distinction)

The sketch conflated these in `textileblog_mens_trouser.json`. In the rebuild, we separate them:

### Object-model DSL (v1, required)
The language the drafting table speaks natively. Describes *what exists in a pattern*:
- Panel (closed path + grain + metadata)
- Seam edge (typed edge + seam-pair link)
- Dart (legs + point + seam it consumes from)
- Notch (position on a seam edge)
- Construction line
- Domain operations as typed commands (slash-and-pivot, add-ease, mirror, etc.)

### Generative procedure DSL (deferred past v1)
The language for encoding a drafting method (e.g., TextileBlog's trouser procedure). Describes *how to produce an initial pattern from measurements*:
- Variables / expressions referencing measurements and style params
- Ordered point construction
- Panel assembly from points

In the sketch, the generative DSL was executed server-side (for IP protection — a goal we've dropped). In the rebuild, if/when we build this, it runs client-side and produces an initial object-model document the user then edits.

## Stack Decisions

| Layer | Choice | Notes |
|-------|--------|-------|
| Frontend framework | React 19 | Inherit from sketch. Plenty good for this. |
| Language | TypeScript with pragmatic strict | `any` at boundaries OK; don't go strict-mode gymnastics |
| Build tool | Vite | Sketch uses it; no reason to change |
| State | Zustand, single store with slices | Rebuild fixes sketch's dual-store sync issues |
| Canvas | SVG (revisit if perf wall) | Simpler, DOM-inspectable, matches sketch |
| Backend | Fastify 5 | Inherit from sketch |
| DB | Postgres 16 | Inherit from sketch; JSONB for pattern docs |
| Auth | Hand-rolled session tokens (SHA256-hashed) | Sketch's `server/src/session.ts` is production-grade |
| Deploy | Fly.io | Inherit |
| Testing | Vitest (frontend), add server tests too | Sketch had no server tests — fix that |

## Sketch Lessons

### Kept (proven correct, worth preserving)
- **Geometry primitives** — `Point`, `Path`, `Panel`, `BoundingBox`. Pure, serializable, well-tested. File: `src/engine/geometry/`.
- **Expression engine** — `src/engine/interpreter/expressionEngine.ts`. Sandboxed mathjs with custom geometry functions. Keep for later parametric phase.
- **Session token hashing** — `server/src/session.ts`. SHA256 hashes stored in DB, raw token in cookie. Production-grade. Clears cookies with matching attributes (Chrome bug workaround).
- **Rate-limited auth routes** — 10 req/min on login/register, 300/min global.
- **Multi-stage Dockerfile** shape — well-commented, separates frontend/backend build concerns.
- **Undo/redo structure** — correct (cloned panels, cap depth). Will revisit granularity.
- **Seam length computation** — `src/engine/seamMatching.ts`. 20-point numerical integration for cubic Bezier arc length. Overkill for most cases but correct.

### Dropped (structural debt)
- **Dual Zustand stores** (`patternStore` + `draftingStore`) — duplicated panel state, no sync. Wizard→editor transition throws away state. Rebuild uses one document model.
- **Server-side DSL interpreter** — only reason was IP protection, which we've dropped as a goal. Everything runs client-side.
- **Wizard-first UX** — 4-step flow (person-type → measurements → style → drafting) gates the drafting experience behind ceremony. Rebuild goes drafting-first.
- **Hardcoded garment logic** — `if (basePattern?.garment_key === 'trouser')` in `patternStore.ts:188`. Doesn't scale. Rebuild uses a domain-operation registry.
- **Style pieces as imperative client code** — `makeModernWaistband`, `makeZipperFly`, etc. Live outside the DSL, not reusable. Rebuild expresses these as domain operations in the object-model DSL.
- **`saved_patterns.style_opts / measurements` JSONB columns** duplicating `draft_data` — sync risk, no enforced consistency.

### Friction points in the sketch (from Explore agent analysis)
- Two stores, one pattern: sync fragility when transitioning wizard → editor
- `draftingStore.ts` at 965 lines is a monolith; should split
- String keys for adjustment tracking (`${panelIndex}-${segmentIndex}-${field}`) — brittle
- README claims "needs signed tokens"; code already implements them — stale docs
- No server-side tests
- No E2E tests
- Only DSL ever tested with a real garment is `SIMPLE_RECT`; the actual trouser is untested

Full Explore report is in conversation history from 2026-04-23 (not archived to a file yet; if lost, re-run Explore agent with the same prompt).

## User Research

_Empty. First assignment (see `roadmap.md` → Phase 7 / design doc → The Assignment): find one non-Justin hobby sewist, talk to them, capture verbatim quotes here._

### Template for entries

```markdown
### {Name or handle}, {date}, {context of conversation}

**Background:** what they sew, what methods they use, what they've tried

**Pain points (verbatim):**
> "direct quote"
> "another direct quote"

**Reaction to the tool (if shown):**

**Surprises (things they did or said I didn't expect):**
```

## User Guide Notes (collecting for eventual user docs)

Justin's own running list of tips, quirks, and gotchas that should land in a written user guide later. Add entries as they surface during real drafting; keep them short and behavior-focused.

- **Moving points after curves exist:** once the panel has curved segments, moving a point may be restricted in some directions to keep the outline from self-intersecting. If a direction is blocked, try a different direction, or straighten the affected lines first and then move the point.

## Related external resources

- **Cornelius Quiring's tutorial series** — Justin's source for the men's trouser method. Paid private tutorial.
- **TextileBlog** — the drafting system Justin followed. Published online, not proprietary.
- **Aldrich, Armstrong, Bernard** — standard pattern-drafting textbooks. Reference material for domain vocabulary and alternative methods.

## Canonical design doc

`~/.gstack/projects/pattern-drafter-workspace/justin-rebuild-design-20260423-144423.md` — the full office-hours design doc. Read it for the complete problem statement, demand analysis, approach rationale, and assignment. Downstream gstack skills (`/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`) will auto-discover this file.
