---
name: door-labels-are-verbs
description: Both hub doors AND inside-door breadcrumb roots use verb phrases (Draft Patterns / Measure Wearers); D24 override of an earlier verb-door/noun-breadcrumb split
metadata: 
  node_type: memory
  type: project
  originSessionId: 260e021d-4075-49a4-bd0f-5cf47395d5ee
---

As of 2026-05-15, both the hub doors and the inside-door breadcrumb roots use **verb phrases**:

- Hub door: `Draft Patterns` (→ `/patterns`)
- Hub door: `Measure Wearers` (→ `/wearers`)
- Breadcrumb root inside `/patterns`: `Draft Patterns › <pattern-name>`
- Breadcrumb root inside `/wearers`: `Measure Wearers › <wearer-name>`

**Why both verb-form (D24 — eng review revision):** An earlier draft had the door verb-form (`Measure Wearers`) but the breadcrumb root noun-form (`Wearers`). Outside-voice review flagged the split as cognitive friction — door label and breadcrumb show different words for the same destination. Justin chose symmetric verb-form across both surfaces. Reads slightly weirder inside a breadcrumb chain but is unambiguous about what activity you're in.

**Why verb phrases at all (D14, original):** Justin pointed out 2026-05-15 that the prior noun-only labels (`Drafting`, `Measurements`) describe the *area* but not the user's *action*. The two-door hub is fundamentally a "pick what you're doing" chooser per [[hub-is-a-fork]], so the door labels should be actions.

**How to apply:**
- Hub door labels: verb phrase. Inside-door breadcrumb root: SAME verb phrase. Don't split.
- If new top-level activities are ever added to the hub, follow the pattern: verb-phrase door + verb-phrase breadcrumb root.
- Routes (`/patterns`, `/wearers`) stay nouns — those are URLs, not user-facing labels.
- Sites updated in 2026-05-15 spec: `src/ui/HubPage.tsx:23,29`, `src/ui/WearersList.tsx:39`, `src/ui/wearers/WearerEditor.tsx:123,140,212`, `src/ui/PatternsList.tsx:132`.
- See [[wearer-editor-redesign-locked]] for the broader decision context.
