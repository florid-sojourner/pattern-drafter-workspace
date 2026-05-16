---
name: anatomy-is-system-wide-not-per-wearer
description: "Anatomy diagrams are system-wide reference graphics, not per-wearer metadata; figureVariant removed from WearerDocument 2026-05-15"
metadata: 
  node_type: memory
  type: project
  originSessionId: 260e021d-4075-49a4-bd0f-5cf47395d5ee
---

Anatomy diagrams (if they return) are **system-wide reference graphics**, keyed by the measurement system, not per-wearer metadata.

**History:** The original Phase 4B sketch had per-wearer anatomy. Each wearer's document carried a `figureVariant: 'male' | 'female' | 'child' | 'neutral'` field, used by the `AnatomicalDiagram` component to pick which body image to render alongside the measurement fields. On 2026-05-14 morning (commit `a2f07a0`), the `AnatomicalDiagram` was dropped from the render when the bag-by-system layout replaced the bag-by-anatomy layout. The field stayed in the data model but no UI read it.

On 2026-05-15 Justin pushed back: "Why do different users have different anatomy diagrams, and why do either have any at all? I thought it was only ever a system-wide graphic." Correct. The field is being removed from `WearerDocument` in the wearer-editor redesign (see [[wearer-editor-redesign-locked]]).

**Why:** The bag-of-keys + per-system-provenance model from [[measurement-bag-model]] makes per-wearer anatomy redundant — the keys you're measuring are tied to the system you're working in, not to the wearer's biological category. If a future feature wants anatomy diagrams to teach key-taking, the diagram is `aldrich-system-body.svg`, not `marks-body.svg`.

**How to apply:**
- `WearerDocument` does not carry a `figureVariant` field. Validator silently drops it from legacy docs.
- If anatomy diagrams ever return, they're a property of a `SystemDocument` (or shipped as static asset keyed by the system's slug), never a property of a wearer.
- "Per-wearer body image" is not a feature on the roadmap. If it is ever proposed, surface this memory and the [[measurement-bag-model]] reasoning before saying yes.
