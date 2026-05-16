---
name: solo-tester-destructive-migrations-ok
description: Justin is the only user/tester; no production customers; destructive migrations and table drops are acceptable when the alternative is data-shape gymnastics
metadata: 
  node_type: memory
  type: project
  originSessionId: 260e021d-4075-49a4-bd0f-5cf47395d5ee
---

Justin is the sole tester of pattern-drafter as of 2026-05-15. The Fly.io deployment exists and runs migrations on boot, but no real users besides Justin himself touch it. There is **no production data worth preserving**.

**How to apply:**
- When a data-shape change is on the table, prefer **drop-and-recreate migrations** over careful in-place transforms whenever the in-place version adds meaningful complexity. Example: 2026-05-15 wearer-editor redesign added migration `005_wearers_reset.sql` that `DROP TABLE IF EXISTS wearers` + recreates from scratch, rather than writing an UPDATE-each-row backfill for the `draftSession` → `today's-session-record` migration. Justin's direction: "we can drop all wearer data... include a migration to drop the table and rebuild."
- Don't moralize about data preservation. Don't propose multi-step backfills as the "responsible" path when Justin has explicitly green-lit a table drop. Pairs with [[iterate-fast-no-migration-handwringing]].
- This permission extends to the wearers, patterns, systems tables in dev. Be more conservative about `users` and `sessions` (auth tables) because rebuilding requires re-registering accounts and clearing browser-stored sessions.
- This stops applying the moment a non-Justin user starts depending on the app. Resurface this memory if/when external users land (Justin's future drafter beta, etc.).