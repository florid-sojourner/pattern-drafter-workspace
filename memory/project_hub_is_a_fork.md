---
name: hub-is-a-fork-not-a-workspace
description: "Hub is a two-door chooser (Drafting / Measurements), mutually exclusive activities — newcomers pick one and the other disappears"
metadata: 
  node_type: memory
  type: project
  originSessionId: 8a95c1c5-178a-4f71-9e77-e15ea3c9cb13
---

The hub at `/hub` is a **two-door chooser**, not a workspace that shows both activities at once. Two cards: Drafting (`/patterns`) and Measurements (`/wearers`). Mutually exclusive. After the user picks, the other activity is not visible until they navigate back to `/hub`.

Cross-pollination only happens at the moments where it's contextually necessary (e.g., new-pattern modal offering wearer quick-picks), and only inside one door — never on the chooser itself.

**Why:** Justin's framing on 2026-05-13 — "a user who shows up to get into the one or the other they're there for in a mutually exclusive way to totally avoid confusion. Wearer Measurements vs Drafting." Lower cognitive load; fewer choices at once; the main barrier in other drafting platforms (Seamly2D, Valentina, etc.) is "no idea where to start," which a populated workspace makes worse.

**How to apply:**
- The chooser page shows only the two doors. No recent-work strip, no counts, no "+ new" buttons, no email/sign-out chrome competing for attention beyond the necessary minimum.
- Inside Drafting: pattern list, resume affordance, "+ New pattern" with optional wearer pick. See [[measurements-are-optional-convenience]] for why wearer is optional.
- Inside Measurements: wearer list, "+ New wearer," measurement editor. Never shows pattern chrome.
- The wearer-this-pattern-is-for relationship is metadata-shaped on the drafting page (Inspector accordion, breadcrumb segment that links to her measurement editor) — never a hierarchy in URL terms. Pattern lives under Drafting; wearer lives under Measurements; the link between them is a `wearerId` field, not a URL nesting.
- Breadcrumb hierarchy honors the two-door tree: `⌂ Hub › Patterns › {pattern}` and `⌂ Hub › Measurements › {wearer}`.
