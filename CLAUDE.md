# Claude orientation — pattern-drafter rebuild

This directory (`/home/justin/Documents/Personal/pattern-drafter-workspace/`) is where Justin is rebuilding **pattern-drafter**, a domain-specific CAD tool for garment pattern drafting. An earlier experimental sketch lives at `pattern-drafter/` (kept as read-only reference). The rebuild will land in a new directory (see `context/roadmap.md` → Open Decisions).

## Read these first, in order

1. **`context/rules.md`** — invariants we follow while building. Load-bearing. Violations need an explicit conversation.
2. **`context/wiki.md`** — shared understanding. Domain glossary, premises, sketch lessons, stack decisions.
3. **`context/roadmap.md`** — what's next, current phase, open decisions.
4. **`~/.gstack/projects/pattern-drafter-workspace/justin-rebuild-design-20260423-144423.md`** — canonical office-hours design doc with full problem statement and rationale. Downstream gstack skills auto-discover this.

## Who the user is

Justin — hobby sewist rebuilding a tool he previously sketched. Works across multiple machines. Cares about taste, craft, and the quality of the drafting experience. See `context/wiki.md` → Primary User.

## How Justin collaborates

- Direct feedback, including corrections — take them seriously and update.
- Values specificity over generic advice.
- Will challenge defaults (e.g., TypeScript — see `rules.md` → R11) and expects real reasons, not deference.
- Open to being pushed on product decisions; appreciates calibrated acknowledgment of strong answers (not sycophancy).
- Signals impatience when over-explained; honor "let's not stay too long."

## Current state (2026-04-23)

- Office-hours interview complete. Premises locked. Approach B selected. Design doc **APPROVED**.
- Logistics resolved: sketch renamed (at Phase 1 kickoff) to `pattern-drafter-sketch/`; rebuild claims `pattern-drafter/`; same GitHub repo (private); same Fly.io app; clean-slate first commit with `sketch-final` tag on prior HEAD.
- No code written yet. Phase 1 (rename + scaffold) is the next step.
- Assignment pending: find one non-Justin hobby sewist to talk to this week. See design doc → The Assignment.

## Sketch location

Currently at `./pattern-drafter/`; will be renamed to `./pattern-drafter-sketch/` at Phase 1 kickoff. Don't modify. Reference freely. Has proven-good geometry primitives (`src/engine/geometry/`), expression engine (`src/engine/interpreter/expressionEngine.ts`), and session handling (`server/src/session.ts`) we're porting. Structural debt (dual stores, wizard-first UX, hardcoded garment logic) is what the rebuild is fixing.

## Memory and context files

- **Persistent memories:** `~/.claude/projects/-home-justin-Documents-Personal-pattern-drafter-workspace/memory/` (indexed by `MEMORY.md` there). Contains user profile and feedback from past sessions.
- **Project context:** `./context/` (roadmap, wiki, rules). Update these as decisions land.
- **Gstack canonical doc:** `~/.gstack/projects/pattern-drafter-workspace/` (design docs, auto-discovered by /plan-* skills).

## Working style

- Prefer editing existing files over creating new ones.
- Consult `context/rules.md` before proposing architectural changes. Rule violations warrant discussion, not silent drift.
- Keep `context/roadmap.md` decisions log current when decisions land.
- Never drop the sketch directory without explicit instruction (see rule R14).

## Skill routing

When the user's request matches an available gstack skill, invoke it via the Skill tool — the skill has multi-step workflows, checklists, and quality gates that produce better results than ad-hoc answers.

Key routing rules:
- Product ideas, "is this worth building", brainstorming → invoke /office-hours
- Strategy, scope, "think bigger" → invoke /plan-ceo-review
- Architecture review → invoke /plan-eng-review
- Design system → invoke /design-consultation
- Design review of a plan → invoke /plan-design-review
- Developer experience of a plan → invoke /plan-devex-review
- "Review everything" → invoke /autoplan
- Bugs, "why is this broken" → invoke /investigate
- Test a site, find bugs → invoke /qa (or /qa-only)
- Code review → invoke /review
- Visual polish → invoke /design-review
- Ship / deploy / PR → invoke /ship
- Merge + verify → invoke /land-and-deploy
- Security audit → invoke /cso
- Save / restore progress → invoke /context-save and /context-restore
- Code quality dashboard → invoke /health
